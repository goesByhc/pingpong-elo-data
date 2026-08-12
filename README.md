# PingPong ELO Data

This directory is designed to be publishable as a standalone open data package.
The application code, scrapers, Elo engine, and deployment configuration can stay private.

## Layout

```text
data/
├── players.csv          # Player metadata, aliases, seed Elo, profile fields
├── tiers.json           # Tournament tiers, K factors, round multipliers
├── tournaments.csv      # Tournament slugs / names / dates / tiers (canonical)
├── tournament_names.json# EN→ZH display-name dictionary (four levels)
├── matches/             # Match-result CSV files
│   ├── 2026_WTT_...
│   └── team/            # Optional team-event CSV files
├── avatars/             # Player avatar source images
├── sources/             # Raw or auxiliary collection-stage files
├── backup_clean/        # Pre-cleaning / pre-import backups (not published)
└── logs/                # Runtime logs (crawlers/scripts); not published
```

## Public Repo Boundary

Safe to publish:

- `players.csv`
- `tiers.json`
- `matches/*.csv`
- `matches/team/*.csv`
- `sources/*.csv` if you want to expose raw/source provenance
- `avatars/`
- data documentation and contribution templates

Keep private or ignore:

- SQLite databases such as `pingpong.db`
- cookies, API keys, login exports
- generated frontend JSON if not needed by contributors
- local logs and backups

## Match CSV Rules

Match files live under `data/matches/`.
Team-event match files live under `data/matches/team/` and use the same
per-tournament CSV naming style.

`player_a` should be the winner when `score_a > score_b`.
Dates use `YYYY-MM-DD`.
Tournament tier is read from `tiers.json` when missing or set to `OTHER`.

### Match CSV Columns

| Column        | Meaning                                                                  |
|---------------|--------------------------------------------------------------------------|
| `tournament`  | Tournament display name (EN).                                            |
| `date`        | Match date, `YYYY-MM-DD`.                                                |
| `player_a`    | Winner (when `score_a > score_b`).                                       |
| `player_b`    | Loser.                                                                   |
| `score_a` / `score_b` | Games won (e.g. `4`,`1`).                                       |
| `game_scores` | Per-game points, e.g. `11,3,11,11,11\|8,11,9,5,5`.                       |
| `round`       | Stage within the event. See round codes below.                           |
| `tier`        | Tournament tier key (see `tiers.json`).                                  |
| `sub_event`   | Discipline code: `MS`/`WS`/`MT`/`WT`/`MD`/`WD`/`XD` etc.                 |
| `event_id`    | ITTF/WTT event id when scraped.                                          |
| `stage`       | Draw phase: `Qualification` / `Main Draw` / `Position Draw`.             |
| `player_a_raw` / `player_b_raw` | Original name strings from the source (pre-normalization). |

### Round Codes

`round` uses several encodings depending on source:

- `QUAL` — qualification; `QR128`…`QR2` — qualifying rounds (fewer digits = later).
- `GROUP` — group stage.
- `R128` / `R64` / `R32` / `R16` — main-draw rounds (players remaining).
- `QF` / `SF` / `F` (also `QuarterFinal` / `SemiFinal` / `Final`) — knockouts.
- Numeric rounds (`2`/`4`/`8`/`16`/`256`) encode **players remaining** in that
  round (e.g. `2` = final, `4` = semi-final, `8` = quarter-final). In
  `Position Draw` they denote placement matches (`2` = bronze match).

### Elo Computation Order

`scripts/load_scraped_matches.py` processes matches in **date → stage → round**
order, so a player's rating evolves along the real schedule within a day
(qualification first, then main draw, placement matches last). See
`_stage_rank` / `_round_rank` in the loader.

### Tournament Chinese Names (`tournament_names.json`)

Chinese display names are generated at export time by
`scripts/tournament_display.py::tournament_name_zh()`, using four
dictionary levels in order:

1. `exact` — full-name exact match wins.
2. `phrases` — event-type phrases (e.g. `South American Championships`,
   `WTT Cup Finals`); replaced longest-first to avoid substring hits.
3. `events` — discipline words (`Men's Singles` → `男单`).
4. `places` — city / continent / country names (`Santiago` → `圣地亚哥`).

Add new entries to `tournament_names.json` (not to the DB) when a tournament
name stays in English; longer keys match first, so put `Tunisia` before
`Tunis` and `European` after `Europe`-style conflicts are resolved by length.

### Tournaments CSV (`tournaments.csv`)

Canonical tournament metadata: `slug`, `name`, `name_en`, `name_zh`, `tier`,
`country`, `city`, `start_date`, `end_date`, `aliases`. The loader matches
scraped `tournament` names against `name` / `aliases`; a missing entry just
falls back to an auto-generated slug (with a warning).
