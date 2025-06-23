# Game Metadata

Another introduction here

## Frequency

The number of times a game or franchise was referenced by different literature.

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

```sql game_counts
select game['unnest'] as game_name, paper_type, count(game) as game_count
from ${papers_to_games},
unnest(games) as game
group by all
having count(game) >= 3
order by game_count desc, game_name asc
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
order by game_name asc
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

```sql franchise_counts
select game['unnest'][:-4] as franchise, paper_type, count(game) as game_count
from ${papers_to_franchises},
unnest(games) as game
group by all
having count(game) >= 3
order by game_count desc, franchise asc
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
order by franchise asc
```

<Tabs fullWidth=true>
    <Tab label="Games">
        <BarChart
            data={game_counts}
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
            data={franchise_counts}
            x=franchise
            y=game_count
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

## Games by Platform

All games included in the map, broken down by platform.

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

## Games by Genre

All games included in the map, broken down by genre.

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
