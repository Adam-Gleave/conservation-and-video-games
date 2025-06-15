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
