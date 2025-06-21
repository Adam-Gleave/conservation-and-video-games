All papers included in the dataset. Click on a row to view more information.

```sql papers_with_link
select 
   first_author, 
   publication_year,
   title,
   publications.name as publication,
   '/papers/' || papers.id as link
from literature_db.papers
join literature_db.publications
on publication = publications.id
```

<DataTable data={papers_with_link} link=link rows=50 sort="first_author asc" search=true>
   <Column id=first_author />
   <Column id=publication_year fmt=id />
   <Column id=title />
   <Column id=publication />
</DataTable>
