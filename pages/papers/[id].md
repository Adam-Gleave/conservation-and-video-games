---
queries:
   - papers: papers.sql
---

```sql papers_filtered
select * from ${papers}
where id = '${params.id}'
```

<DataTable data={papers_filtered}/>

## <Value data={papers_filtered} column=title />

<p/>
First author: <Value data={papers_filtered} column=first_author/>
<br/>
Year: <Value data={papers_filtered} column=publication_year fmt=id/>
<p/>
<br/>

### Abstract
<Value data={papers_filtered} column=abstract fmt=id/>

<p/>
<br/>
Language: <Value data={papers_filtered} column=language />
<br/>
Country of Affiliation: 
