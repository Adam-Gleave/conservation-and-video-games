---
title: Papers
queries:
   - papers: papers.sql
---

Click on an item to see more detail


```sql papers_with_link
select *, '/papers/' || id as link
from ${papers}
```

<DataTable data={papers_with_link} link=link/>
