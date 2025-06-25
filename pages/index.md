# Overview

Welcome to the interactive database that complements Blake et al.'s (2025) manuscript: “_How commercial video games engage with biodiversity and conservation: a systematic map of literature_”. The manuscript is being peer-reviewed; a preprint can be found <Link 
        url="https://osf.io/preprints/socarxiv/f2qs7_v1"
        label="here"
    />. 

The manuscript provides useful context for this research, as well as suggestions for future work based on our findings. 

Every table and figure on each page can be downloaded for use offline. Visit the _About Us_ page to get in touch with comments or questions - we would love any feedback!

---

## Project description

Mass media is widely used to engage audiences with the natural world. Given their extensive reach and attention to ecological topics, there has been growing interest in using commercial video games for conservation outreach. However, there has not yet been a holistic overview of work on this topic.

We produced the first systematic map on how commercial video games engage with biodiversity and conservation. This includes 380 papers, covering five languages as well as both empirical and non-empirical literature (i.e., the paper either does or does not involve data collection).

Overall, the field is even more diverse and multidisciplinary than expected: 450 different games have been studied, and a variety of ways to depict or engage with real-world ecology have been scrutinised. Ecocriticism dominates non-empirical work – often evaluating the extent that game design encourages or condemns exploitation of nature. Empirical papers are overwhelmingly non-experimental; research has most often explored how video games represent real-world wildlife, influence players’ ecological perceptions, or inform interactions with virtual nature. 

Despite the volume of literature, there are glaring omissions. Few studies employed pre/post-test assessments or even control groups, therefore making it difficult to infer the causal effects of video games. Most notably, there is not yet evidence on whether games make players more or less likely to take real-world pro-ecological action.

Video games clearly have the potential to represent biodiversity and convey ecological messages – whether these be helpful or harmful for conservation. However, only by taking more rigorous approaches can we better understand the impact of games for this purpose. This map highlights several promising areas for future study. As examples, in our manuscript we strongly recommend that researchers consider the role of player differences on the impact of games, observe actual behaviours and how these change after play, and pursue interdisciplinary collaborations with the games industry for more thorough, applicable, and novel findings.

---

## Papers included

All 380 papers included in the dataset. Click on a row to view more information.

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
