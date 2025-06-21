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

__<Value data={comparator_percent_query} column=percent fmt=pct2 />__ of all studies use comparators. 

__<Value data={control_group_percent_query} column=percent fmt=pct2 />__ of all studies use control groups. 

__<Value data={pre_post_measures_percent_query} column=percent fmt=pct2 />__ of all studies use pre/post measures.

<br/>

---

<br/>

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

<p/>
<br/>

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

<p/>
<br/>

## Variables and Outcomes by Research Type

<p/>
<br/>

```sql study_types
select study_outcomes.outcome as outcome, research_type, count(research_type) as count
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