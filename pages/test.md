## Literature by Country

Country of the primary author of each piece of literature. Click on a country to filter the table below.

```sql countries_query
select
    first_author,
    countries.name as country,
    publication_year,
    title,
    publications.name as publication,
from literature_db.papers
join literature_db.countries
on literature_db.papers.first_author_country = literature_db.countries.id
join literature_db.publications
on literature_db.papers.publication = literature_db.publications.id
group by all
order by first_author asc
```

```sql countries_count
select
    country, count(country) as country_count
from ${countries_query}
group by country
```

<AreaMap
    data={countries_count}
    areaCol=country
    geoJsonUrl='https://d2ad6b4ur7yvpq.cloudfront.net/naturalearth-3.3.0/ne_110m_admin_0_countries.geojson'
    geoId=name
    value=country_count
    startingZoom=4
    height=420
    name=publication_map
/>

```sql filtered_countries_query
select * from ${countries_query}
where country = '${inputs.publication_map.country}'
```

{#if inputs.publication_map.country == true}
    <DataTable data={countries_query}>
        <Column id=first_author />
        <Column id=country />
        <Column id=publication_year fmt=id />
        <Column id=title />
        <Column id=publication />
        <Column id=language />
    </DataTable>
{:else}
    <DataTable data={filtered_countries_query}>
        <Column id=first_author />
        <Column id=country />
        <Column id=publication_year fmt=id />
        <Column id=title />
        <Column id=publication />
        <Column id=language />
    </DataTable>
{/if}

```sql language_query
select 
    first_author, 
    publication_year, 
    title, 
    publications.name as publication,
    language,
from literature_db.papers
join literature_db.publications
on literature_db.papers.publication = literature_db.publications.id
group by all
```

```sql language_percentage_query
select percent from
(
    select 
        language, 
        count(language) / sum(count(language)) over() as percent
    from ${language_query}
    group by language
)
where lower(language) = 'english'
```

## Language Breakdown
Written language of each piece of literature. English constitutes __<Value data={language_percentage_query} column=percent fmt=pct0 />__ of the data set.

```sql language_donut_query
select
    language as name,
    count(language) as value
from ${language_query}
group by language
```

<ECharts config={
    {
        tooltip: {
            formatter: '{b}: {c} ({d}%)'
        },
        series: [
            {
                type: 'pie',
                radius: ['40%', '70%'],
                data: [...language_donut_query],
            }
        ]
    }
}/>

Filter all literature by language below:

<Dropdown
    data={language_query}
    name=language_dropdown
    value=language
    title="Language"
/>

```sql filtered_language_query
select *
from ${language_query}
where language like '${inputs.language_dropdown.value}'
order by first_author asc
```

<DataTable data={filtered_language_query}>
    <Column id=first_author />
    <Column id=publication_year fmt=id />
    <Column id=title />
    <Column id=publication />
    <Column id=language />
</DataTable>

## Paper Types over Time

Summary of paper types (empirical vs. non-empirical) by publication date.

```sql paper_types_query
select 
    publication_year,
    type,
    count(publication_year) as paper_count
from literature_db.papers
group by publication_year, type
```

<BarChart
    data={paper_types_query}
    x=publication_year
    y=paper_count
    series=type
    xFmt=id
/>

## Games by Platform

All games included in the map, broken down by platform.

```sql platforms_query
select
    platform, count(platform) as platform_count
from literature_db.game_platforms
group by platform
order by platform asc
```

<BarChart
    data={platforms_query}
    x=platform
    y=platform_count
    sort=false
/>

<Dropdown
    data={platforms_query}
    name=platform_dropdown
    value=platform
    title="Platform"
    order="platform asc"
/>

```sql games_by_platform_query
select games.name as game, array_agg(platform) as platforms
from literature_db.games
join literature_db.game_platforms
on literature_db.games.id = literature_db.game_platforms.game_id
where type = 'game'
group by games.name
having contains(platforms, '${inputs.platform_dropdown.value}')
order by game asc
```

<DataTable data={games_by_platform_query}>
    <Column id=game />
    <Column id=platforms />
</DataTable>

## Games by Genre

All games included in the map, broken down by genre.

```sql genres_query
select
    genres.name as genre, count(games_to_genres.genre_id) as genre_count
from literature_db.games_to_genres
join literature_db.genres
on literature_db.games_to_genres.genre_id = literature_db.genres.id
group by name 
order by name asc
```

<BarChart
    data={genres_query}
    x=genre
    y=genre_count
    sort=false
/>

## Studies by Research Type

```sql research_types
select
    studies.research_type as research_type, count(research_type) as research_type_count
from literature_db.studies
group by research_type
order by research_type asc
```

```sql research_types_donut_query
select
    research_type as name,
    research_type_count as value
from ${research_types}
```

<ECharts config={
    {
        tooltip: {
            formatter: '{b}: {c} ({d}%)'
        },
        series: [
            {
                type: 'pie',
                radius: ['40%', '70%'],
                data: [...research_types_donut_query],
            }
        ]
    }
}/>

## Studies by Data Type

```sql data_types
select
    studies.data_type as data_type, count(data_type) as data_type_count
from literature_db.studies
group by data_type
order by data_type asc
```


```sql data_types_donut_query
select
    data_type as name,
    data_type_count as value
from ${data_types}
```

<ECharts config={
    {
        tooltip: {
            formatter: '{b}: {c} ({d}%)'
        },
        series: [
            {
                type: 'pie',
                radius: ['40%', '70%'],
                data: [...data_types_donut_query],
            }
        ]
    }
}/>

```sql studies
select * from literature_db.studies
```

```sql study_count_query
select 
    count(case when comparator is not null then 1 else null end) as comparator_count,
    count(case when control_group = true then 1 else null end) as control_group_count,
    count(case when pre_post_measures = true then 1 else null end) as pre_post_measures_count,
    count(*) as all_count
from literature_db.studies
```

```sql comparator_percent_query
select 
    (comparator_count / all_count) * 100 as percent
from ${study_count_query}
```

```sql control_group_percent_query
select
    (control_group_count / all_count) * 100 as percent
from ${study_count_query}
```

```sql pre_post_measures_percent_query
select
    (pre_post_measures_count / all_count) * 100 as percent
from ${study_count_query}
```

__<Value data={comparator_percent_query} column=percent fmt=pct0 />__ of all studies use comparators. __<Value data={control_group_percent_query} column=percent fmt=pct0 />__ of all studies use control groups. __<Value data={pre_post_measures_percent_query} column=percent fmt=pct0 />__ of all studies use pre/post measures.

<!-- <Dropdown
    data={genres_query}
    name=genre_dropdown
    value=genre
    title="Genre"
    order="genre asc"
/>

-- ```sql games_by_genre_query
-- select games.name as game, array_agg(genres.name) as genres
-- from literature_db.games
-- join literature_db.games_to_genres
-- on literature_db.games_to_genres.game_id = literature_db.games.id
-- join literature_db.genres
-- on literature_db.genres.id = literature_db.games_to_genres.genre_id
-- group by games.name
-- having contains(genres, '${inputs.genre_dropdown.value}')
-- order by game asc
-- ```

<DataTable data={games_by_genre_query}>
    <Column id=game />
    <Column id=genres />
</DataTable> -->
