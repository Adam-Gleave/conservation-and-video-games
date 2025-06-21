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
Published in <Value data={papers_filtered} column=publication_name />
<br/>
<Value data={papers_filtered} column=publication_type />
<p/>
<br/>
<Value data={papers_filtered} column=source />
