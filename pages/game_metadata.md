# Game Metadata
This page provides details regarding the games and franchises that were addressed by the papers in our map. Where specified, 151 different game franchises and 450 unique titles released between 1976-2024 were included across the literature. Game franchises refer to a set of related video games (e.g., _Pokémon_; Game Freak, 1996-present), whilst unique titles are specific instalments within a franchise (e.g., _Pokémon Go_; Niantic, 2016). 

Use the tables to search for games, franchises, platforms, and genres to find what papers addressed them.

---


## Frequency of games and franchises

The number of times a game or franchise was addressed by each type of paper. Only those covered in at least three separate papers are shown here.

<!-- All games associated with a given paper (whether by paper or studies), deduplicated -->
```sql papers_to_games
select 
    coalesce(p_id, s_id) as paper_id,
    replace(upper(substring(papers.type, 1, 1)) || lower(substring(papers.type, 2, strlen(papers.type))), '_', '-') as paper_type,
    papers.first_author as author,
    papers.publication_year as year,
    papers.title as title,
    array_distinct(array_concat(p_names, s_names)) as games
from
(
    select papers.id as p_id, array_agg(distinct games.name) as p_names
    from literature_db.papers
    join literature_db.games_to_papers
    on papers.id = games_to_papers.paper_key
    join literature_db.games
    on games.id = games_to_papers.game_id
    where games.type = 'game'
    group by papers.id
)
full join
(
    select studies.paper_id as s_id, array_agg(distinct games.name) as s_names
    from literature_db.studies
    join literature_db.games_to_studies
    on studies.id = games_to_studies.study_id
    join literature_db.games
    on games.id = games_to_studies.game_id
    where games.type = 'game'
    group by studies.paper_id
)
on p_id = s_id
join literature_db.papers
on papers.id = coalesce(p_id, s_id)
group by all
```

```sql game_counts_over_paper_types
select game['unnest'] as game_name, paper_type as paper_type, count(game) as game_count
from ${papers_to_games},
unnest(games) as game
group by game_name, paper_type
order by game_name asc
```

```sql game_totals
select game_name, sum(game_count) as total,
from ${game_counts_over_paper_types}
group by game_name
```

```sql full_game_counts
select game_name, paper_type, game_count, total
from ${game_totals}
natural join ${game_counts_over_paper_types}
where total >= 3
```

```sql game_paper_details
select 
    author, 
    year, 
    title, 
    game['unnest'] as game_name,
    '/papers/' || paper_id as link
from ${papers_to_games},
unnest(games) as game
group by all
order by game_name, author, year asc
```

<!-- All franchises associated with a given paper (whether by paper or studies), deduplicated -->
```sql papers_to_franchises
select 
    coalesce(p_id, s_id) as paper_id,
    replace(upper(substring(papers.type, 1, 1)) || lower(substring(papers.type, 2, strlen(papers.type))), '_', '-') as paper_type,
    papers.first_author as author,
    papers.publication_year as year,
    papers.title as title,
    array_distinct(array_concat(p_names, s_names)) as games
from
(
    select papers.id as p_id, array_agg(distinct games.name) as p_names
    from literature_db.papers
    join literature_db.games_to_papers
    on papers.id = games_to_papers.paper_key
    join literature_db.games
    on games.id = games_to_papers.game_id
    where games.type = 'franchise'
    group by papers.id
)
full join
(
    select studies.paper_id as s_id, array_agg(distinct games.name) as s_names
    from literature_db.studies
    join literature_db.games_to_studies
    on studies.id = games_to_studies.study_id
    join literature_db.games
    on games.id = games_to_studies.game_id
    where games.type = 'franchise'
    group by studies.paper_id
)
on p_id = s_id
join literature_db.papers
on papers.id = coalesce(p_id, s_id)
group by all
```

```sql franchise_counts_over_paper_types
select game['unnest'][:-4] as franchise, paper_type, count(game) as franchise_count
from ${papers_to_franchises},
unnest(games) as game
group by franchise, paper_type
order by franchise asc
```

```sql franchise_totals
select franchise, sum(franchise_count) as total,
from ${franchise_counts_over_paper_types}
group by franchise
```

```sql full_franchise_counts
select franchise, paper_type, franchise_count, total
from ${franchise_totals}
natural join ${franchise_counts_over_paper_types}
where total >= 3
```

```sql franchise_paper_details
select 
    author, 
    year, 
    title, 
    game['unnest'][:-4] as franchise,
    '/papers/' || paper_id as link
from ${papers_to_franchises},
unnest(games) as game
group by all
order by franchise, author, year asc
```

<Tabs fullWidth=true>
    <Tab label="Games">
        <BarChart
            data={full_game_counts}
            x=game_name
            y=game_count
            series=paper_type
            xFmt=id
            swapXY=true
        />

        <DataTable data={game_paper_details} search=true rows=25 link=link>
            <Column id=game_name />
            <Column id=author />
            <Column id=year />
            <Column id=title />
        </DataTable>
    </Tab>
    <Tab label="Franchises">
        <BarChart
            data={full_franchise_counts}
            x=franchise
            y=franchise_count
            series=paper_type
            xFmt=id
            swapXY=true
        />

        <DataTable data={franchise_paper_details} search=true rows=25 link=link>
            <Column id=franchise />
            <Column id=author />
            <Column id=year />
            <Column id=title />
        </DataTable>
    </Tab>
</Tabs>

<p/>
<br/>

---

<p/>
<br/>

## Platforms

All platforms included in the map. Of the unique game titles, these were most often recorded* as available on console (66.4%), followed by PC (64.2%), mobile (23.8%), arcade (3.8%), then browser (3.6%). 


```sql platforms_query
select
    case when platform = 'pc' then 'PC' else upper(substring(platform, 1, 1)) || lower(substring(platform, 2, strlen(platform))) end as platform,
    count(platform) as platform_count
from literature_db.game_platforms
group by platform
order by platform asc
```

<BarChart
    data={platforms_query}
    x=platform
    y=platform_count
    sort=false
/>

## Genres

All genres included in the map. At least one genre was listed* for 417 unique game titles: the three most common were action (44.8%), role-playing (23.3%), and simulation (18.9%).


```sql genres_query
select genres.name as genre, count(games_to_genres.genre_id) as genre_count
from literature_db.games_to_genres
join literature_db.genres
on games_to_genres.genre_id = genres.id
join literature_db.games
on games.id = games_to_genres.game_id
where games.type = 'game'
group by genres.name 
order by genres.name asc
```

```sql games_genre_platform_query
select 
    games.name as game, 
    array_to_string(array_distinct(array_agg(case when platform = 'pc' then 'PC' else upper(substring(platform, 1, 1)) || lower(substring(platform, 2, strlen(platform))) end)), ', ') as platforms,
    array_to_string(array_distinct(array_agg(genres.name)), ', ') as genre_list
from literature_db.games
join literature_db.game_platforms
on games.id = game_platforms.game_id
join literature_db.games_to_genres
on games.id = games_to_genres.game_id
join literature_db.genres
on games_to_genres.genre_id = genres.id
where type = 'game' and platform in lower(${inputs.platform_dropdown.value}) and genres.name in ${inputs.genre_dropdown.value}
group by all
order by game asc
```



<BarChart
    data={genres_query}
    x=genre
    y=genre_count
    sort=false
/>

#### *Data for platforms and genres was largely collected from MobyGames, so may not be fully accurate. 

<br/>

<Dropdown
    data={platforms_query}
    name=platform_dropdown
    value=platform
    title="Platform"
    order="platform asc"
    multiple=true
    selectAllByDefault=true
/>

<Dropdown
    data={genres_query}
    name=genre_dropdown
    value=genre
    title="Genre"
    order="genre asc"
    multiple=true
    selectAllByDefault=true
/>



<DataTable data={games_genre_platform_query} rows=25>
    <Column id=game />
    <Column id=platforms />
    <Column id=genre_list title=Genres />
</DataTable>
