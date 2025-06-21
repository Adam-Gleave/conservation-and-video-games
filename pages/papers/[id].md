---
queries:
   - papers: papers.sql
---

# {params.id}

```sql papers_filtered
select * from ${papers}
where id = '${params.id}'
```

<DataTable data={papers_filtered}/>
