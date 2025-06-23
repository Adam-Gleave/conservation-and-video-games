# Studies

## List of Empirical Papers

```sql empirical_papers
select 
    first_author,
    publication_year,
    title,
   '/papers/' || papers.id as link
from literature_db.papers 
where type = 'empirical'
order by first_author, publication_year, title asc
```

<DataTable data={empirical_papers} rows=25 link=link>
    <Column id=first_author />
    <Column id=publication_year fmt=id />
    <Column id=title />
</DataTable>

<p/>
<br/>

---

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
    (comparator_count / all_count) as percent
from ${study_count_query}
```

```sql control_group_percent_query
select
    (control_group_count / all_count) as percent
from ${study_count_query}
```

```sql pre_post_measures_percent_query
select
    (pre_post_measures_count / all_count) as percent
from ${study_count_query}
```

<p/>
<br/>

__<Value data={comparator_percent_query} column=percent fmt=pct2 />__ of all studies used comparators. 

__<Value data={control_group_percent_query} column=percent fmt=pct2 />__ of all studies used control groups. 

__<Value data={pre_post_measures_percent_query} column=percent fmt=pct2 />__ of all studies used pre/post measures.

<br/>

---

<br/>

<!-- ## Studies by Research Type -->

```sql research_types
select
    replace(upper(substring(studies.research_type, 1, 1)) || lower(substring(studies.research_type, 2, strlen(studies.research_type))), '_', '-') as research_type,
    count(lower(research_type)) as research_type_count
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

## Studies by Research Type

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

```sql data_types
select
    upper(substring(studies.data_type, 1, 1)) || lower(substring(studies.data_type, 2, strlen(studies.data_type))) as data_type,
    count(data_type) as data_type_count
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

## Studies by Data Type

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

<p/>
<br/>

## Variables and Outcomes by Research Type

<p/>
<br/>

```sql study_types
select 
    replace(upper(substring(study_outcomes.outcome, 1, 1)) || lower(substring(study_outcomes.outcome, 2, strlen(study_outcomes.outcome))), '_', ' ') as outcome,
    replace(upper(substring(studies.research_type, 1, 1)) || lower(substring(studies.research_type, 2, strlen(studies.research_type))), '_', '-') as research_type,
    count(research_type) as count
from
literature_db.studies
join
literature_db.study_outcomes
on studies.id = study_outcomes.study_id
group by outcome, research_type
```

<BarChart
    data={study_types}
    x=outcome
    y=count
    series=research_type
    xFmt=id
    swapXY=true
/>

<br/>

```sql data_types
select distinct data_type
from studies
order by data_type asc
```

```sql research_types
select distinct research_type
from studies
order by research_type asc
```

```sql outcomes
select distinct outcome
from study_outcomes
order by outcome asc
```

<Dropdown
    data={data_types}
    name=data_type_dropdown
    value=data_type
    title="Data type"
    order="data_type asc"
    multiple=true
    selectAllByDefault=true
/>

<Dropdown
    data={research_types}
    name=research_type_dropdown
    value=research_type
    title="Research type"
    order="research_type asc"
    multiple=true
    selectAllByDefault=true
/>

<Dropdown
    data={outcomes}
    name=outcome_dropdown
    value=outcome
    title="Outcome"
    order="outcome asc"
    multiple=true
    selectAllByDefault=true
/> 

```sql papers_by_study
select  
    first_author, 
    publication_year, 
    title,
    data_type,
    research_type,
    array_to_string(array_distinct(array_agg(study_outcomes.outcome)), ', ') as outcomes,
    '/papers/' || papers.id as link
from papers
join studies
on studies.paper_id = papers.id
join study_outcomes
on studies.id = study_outcomes.study_id
where studies.research_type in lower(${inputs.research_type_dropdown.value}) and studies.data_type in lower(${inputs.data_type_dropdown.value}) and study_outcomes.outcome in ${inputs.outcome_dropdown.value}
group by all
order by first_author, publication_year, title asc
```

<DataTable data={papers_by_study} rows=25 link=link>
    <Column id=first_author />
    <Column id=publication_year fmt=id />
    <Column id=title />
    <Column id=data_type />
    <Column id=research_type />
    <Column id=outcomes />
</DataTable>

<br/>

## Player Map

```sql countries_count
select countries.name as country, count(countries.name) as count
from studies_to_countries
join countries
on countries.id = studies_to_countries.country_id
group by countries.name
```

<AreaMap
    data={countries_count}
    areaCol=country
    geoJsonUrl='https://d2ad6b4ur7yvpq.cloudfront.net/naturalearth-3.3.0/ne_110m_admin_0_countries.geojson'
    geoId=name_long
    value=count
    startingZoom=4
    height=420
    name=player_map
/>

```sql papers_by_country
select
    first_author,
    publication_year, 
    title,
    '/papers/' || papers.id as link
from papers
join studies
on studies.paper_id = papers.id
join studies_to_countries
on studies.id = studies_to_countries.study_id
join countries
on studies_to_countries.country_id = countries.id
where countries.name = '${inputs.player_map.country}'
order by first_author, publication_year, title asc
```

<DataTable data={papers_by_country} rows=25 link=link emptySet=pass emptyMessage="No records: make a country selection">
    <Column id=first_author />
    <Column id=publication_year fmt=id />
    <Column id=title />
</DataTable>
