# Paper Metadata

This page provides details regarding paper metadata. Of the 380 papers, 132 (34.7%) were empirical and 248 (65.3%) were non-empirical. Please note that literature for 2025 is not fully represented as searches ended in January. More details for paper metadata can be found in our manuscript and its supplementary information.

The most frequent publication type was journal article (n=201, 52.9%), followed by book chapter (n=69, 18.2%), conference proceeding (n=41, 10.8%), and dissertation (n=38, 10.0%). Two books particularly contributed to the sharp rise in literature for 2024, together containing 20 relevant chapters: _Video Games and Environmental Humanities: Playing to Save The World_ (Aliano & Crowley, 2024) and _Ecogames: Playful Perspectives on the Climate Crisis_ (op de Beke et al., 2024). For journal articles, several texts were from the non-peer-reviewed outlet _Journal of Geek Studies_ (n=22, 10.9%). The most popular peer-reviewed journals represented were _Games and Culture_ (n=9, 4.5%), _Ecozon@_ (n=6, 3.0%), _People and Nature_ (n=5, 2.5%), and _American Entomologist_ (n=5, 2.5%).

Use the tables to search for papers according to their year and language of publication, and first authors' country of affiliation.

---

## Paper types over time

Number of papers published each year.

```sql paper_types_query
select 
    publication_year,
    count(publication_year) as paper_count,
    replace(upper(substring(papers.type, 1, 1)) || lower(substring(papers.type, 2, strlen(papers.type))), '_', '-') as paper_type,
from literature_db.papers
group by all
```

<BarChart
    data={paper_types_query}
    x=publication_year
    y=paper_count
    series=paper_type
    xFmt=id
/>

```sql papers_by_year
select  
    first_author, 
    publication_year, 
    title 
from literature_db.papers
```

<Slider title="Publication Year" name=yearSlider min=2001 max=2025 fmt=id />

```sql filtered_papers
select 
    '/papers/' || id as link,
    publication_year, 
    first_author, 
    title from literature_db.papers 
where publication_year = '${inputs.yearSlider}'
order by publication_year, first_author, title
```

<DataTable data={filtered_papers} rows=25 link=link>
    <Column id=publication_year fmt=id />
    <Column id=first_author />
    <Column id=title />
</DataTable>

---

## Literature by country

Country of the first author's affiliation for each paper. Where first authors specified an affiliated institution, their geographic distribution was strongly biased towards North American (35.4%) and European (44.7%) countries. Click on a country to filter the table below.

```sql countries_query
select
    '/papers/' || papers.id as link,
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
order by first_author, publication_year, title asc
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
    geoId=name_long
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
    <DataTable data={countries_query} link=link rows=25 sort="country asc">
        <Column id=country />
        <Column id=first_author />
        <Column id=publication_year fmt=id />
        <Column id=title />
    </DataTable>
{:else}
    <DataTable data={filtered_countries_query} rows=25 link=link>
        <Column id=country />
        <Column id=first_author />
        <Column id=publication_year fmt=id />
        <Column id=title />
    </DataTable>
{/if}

```sql language_query
select 
    '/papers/' || papers.id as link,
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

---

## Language of papers
Representation of languages for the 380 papers. English constitutes __<Value data={language_percentage_query} column=percent fmt=pct0 />__ of the data set.

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
order by first_author, publication_year, title asc
```

<DataTable data={filtered_language_query} rows=25 link=link>
    <Column id=language />
    <Column id=first_author />
    <Column id=publication_year fmt=id />
    <Column id=title />
</DataTable>
