---
queries:
   - papers: papers.sql
---

```sql papers_filtered
select 
   papers.*,
   countries.name as country_name,
   publications.name as publication_name,
   publications.publication_type,
from literature_db.papers
left join literature_db.countries
on first_author_country = countries.id
join literature_db.publications
on papers.publication = publications.id
where papers.id = '${params.id}'
```

## <Value data={papers_filtered} column=title />

<p/>
First author: <Value data={papers_filtered} column=first_author/>
<br/>
Year: <Value data={papers_filtered} column=publication_year fmt=id/>
<p/>
<br/>

### Abstract
<Value data={papers_filtered} column=abstract />

<p/>
<br/>

### Details

<p/>
Language: <Value data={papers_filtered} column=language />
<br/>
Country of affiliation: <Value data={papers_filtered} column=country_name />
<p/>
<br/>
Published in: <Value data={papers_filtered} column=publication_name />
<br/>
Publication type: <Value data={papers_filtered} column=publication_type />
<p/>
<br/>
Source: <Value data={papers_filtered} column=source />

<p/>
<br/>

---

<p/>
<br/>

## Studies

```sql studies
select 
   studies.id,
   description,
   research_type,
   data_type,
   case when comparator is not null then comparator else 'none' end as comparator,
   case when control_group is true then 'yes' else 'no' end as control_group,
   case when pilot_study is true then 'yes' else 'no' end as pilot_study,
   case when pre_post_measures is true then 'yes' else 'no' end as pre_post_measures,
   case when follow_up is true then 'yes' else 'no' end as follow_up,
   sample_type,
   sample_size,
   case when power_analysis is true then 'yes' else 'no' end as power_analysis,
   array_to_string(array_distinct(array_agg(countries.name order by countries.name asc)), ', ') as countries,
   array_to_string(array_distinct(array_agg(games.name order by games.name asc) filter (where games.type = 'game')), ', ') as games,
   array_to_string(array_distinct(array_agg(games.name order by games.name asc) filter (where games.type = 'franchise')), ', ') as franchises
from studies
left join studies_to_countries
on studies.id = studies_to_countries.study_id
left join countries
on studies_to_countries.country_id = countries.id
left join games_to_studies
on studies.id = games_to_studies.study_id
left join games
on games.id = games_to_studies.game_id
where studies.paper_id = '${params.id}'
group by all
```

```sql studied_games
select
   games.*, study_id
from games
join games_to_studies
on games.id = games_to_studies.game_id
where games.type = 'game'
order by games.name asc
```

{#if studies.length !== 0}
   {#each studies as study}
      <Details 
         title="Study {study.id}" 
         open=true
      >
         Description: {study.description}
         <br/>
         <br/>
         <p>
            Research type: {study.research_type}
            <br/>
            Data type: {study.data_type}
         </p>
         <br/>
         <p>
            Comparator: {study.comparator}
            <br/>
            Control group: {study.control_group}
            <br/>
            Pilot study: {study.pilot_study}
            <br/>
            Pre/post measures used: {study.pre_post_measures}
            <br/>
            Follow-up: {study.follow_up}
         </p>
         <br/>
         <p>
            Sample type: {study.sample_type}
            <br/>
            Sample size: {study.sample_size}
            <br/>
            Power analysis: {study.power_analysis}
            <br/>
            Sample countries: {study.countries}
         </p>
         <br/>
         <p>
            Games studied: {study.games}
         </p>
         <br/>
         <p>
            Franchises studied: {study.franchises}
         </p>
      </Details>
   {/each}
{:else}
No studies associated with this paper.
{/if}
