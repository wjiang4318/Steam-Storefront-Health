# Steam Storefront Health
 
An end-to-end pipeline analyzing 925K Steam reviews across 286 games — Databricks medallion architecture (bronze → silver → gold star schema) feeding a Power BI dashboard.
 
**The brief:** As a Steam executive, understanding the health of the storefront and its reputation with players is important. Using historical data — 991K player reviews across 286 games, plus genre sales leaderboards and Steam's own review labels — 
surface which games the storefront should promote, which top sellers are quietly hurting player trust, and how satisfaction has moved over time.
 
**The finding:** using sales as a metric for game success is the wrong guide. Top sellers land above and below the satisfaction average in roughly equal numbers, and 18 games ranked in a genre's top 10 score below the 81% catalogue average, spread across Action, Simulation, and Sports & Racing. 
Review labels do hold up, tracking sentiment closely. Against a backdrop of recommendation rates falling 22 points since 2013, sentiment is the signal worth promoting on. 
 
![Dashboard](images/dashboard.png)

## Data
 
[Steam Games, Reviews, and Rankings](https://www.kaggle.com/datasets/mohamedtarek01234/steam-games-reviews-and-rankings) (Kaggle) — 286 games, ~991K reviews, and genre leaderboard rankings.
 
## Architecture
 
### Repo structure
 
```
steam_analytics/
├── bronze/
│   └── stg_ingestion.py          raw CSV → Delta, no transformation
├── silver/
│   ├── silver_games              list parsing, game_key normalization
│   ├── silver_reviews            date parsing, type casting, text cleaning
│   └── silver_rankings           game_key normalization
├── gold/
│   ├── dim_games                 surrogate game_id via ROW_NUMBER
│   ├── fact_reviews              joined to dim_games on game_key
│   ├── dim_rankings              snowflake arm off dim_games
│   └── bridge_game_genre         EXPLODE of the genres array
└── images/
```
 
### Data flow
 
```
                    Kaggle CSVs
                         │
                         ▼
              ┌────────────────────┐
              │       BRONZE       │   raw ingestion, no transformation
              │  games · reviews   │
              │      rankings      │
              └─────────┬──────────┘
                        │   type casting · date parsing
                        │   list parsing · game_key normalization
                        ▼
              ┌────────────────────┐
              │       SILVER       │   cleaned, typed, joinable
              │  silver_games      │
              │  silver_reviews    │
              │  silver_rankings   │
              └─────────┬──────────┘
                        │   surrogate keys · dimensional modeling
                        ▼
              ┌────────────────────┐
              │        GOLD        │   star schema
              │                    │
              │  dim_games ──┬── fact_reviews
              │              ├── dim_rankings        (snowflake arm)
              │              └── bridge_game_genre   (game ↔ genre)
              └─────────┬──────────┘
                        │
                        ▼
                     Power BI
```
