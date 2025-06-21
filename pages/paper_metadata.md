# Paper Metadata

Some introduction here

## Paper Types over Time

Summary of paper types (empirical vs. non-empirical) by publication date.

```sql paper_types_query
select 
    publication_year,
    count(publication_year) as paper_count,
    type
from literature_db.papers
group by all
```

<BarChart
    data={paper_types_query}
    x=publication_year
    y=paper_count
    series=type
    xFmt=id
/>

```sql papers_by_year
select  
    first_author, 
    publication_year, 
    title 
from literature_db.papers
```

<Slider title="Publication Year" name=yearSlider min=2002 max=2025 fmt=id />

```sql filtered_papers
select 
    '/papers/' || id as link,
    publication_year, 
    first_author, 
    title from literature_db.papers 
where publication_year = '${inputs.yearSlider}'
```

<DataTable data={filtered_papers} rows=25 link=link>
    <Column id=publication_year fmt=id />
    <Column id=first_author />
    <Column id=title />
</DataTable>

## Literature by Country

Country of the primary author of each piece of literature. Click on a country to filter the table below.

```sql countries_query
select
    countries.name as country,
    first_author,
    publication_year,
    title,
    publications.name as publication,
from literature_db.papers
join literature_db.countries
on literature_db.papers.first_author_country = literature_db.countries.id
join literature_db.publications
on literature_db.papers.publication = literature_db.publications.id
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
    <DataTable data={countries_query} rows=25>
        <Column id=country />
        <Column id=first_author />
        <Column id=publication_year fmt=id />
        <Column id=title />
    </DataTable>
{:else}
    <DataTable data={filtered_countries_query} rows=25>
        <Column id=country />
        <Column id=first_author />
        <Column id=publication_year fmt=id />
        <Column id=title />
    </DataTable>
{/if}

```sql language_query
select 
    language,
    first_author, 
    publication_year, 
    title, 
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

<DataTable data={filtered_language_query} rows=25>
    <Column id=language />
    <Column id=first_author />
    <Column id=publication_year fmt=id />
    <Column id=title />
</DataTable>
