# PingPong ELO Open Dataset

This is an open dataset for evaluating table-tennis playing strength. It contains player metadata, tournament metadata, and match-by-match results for Elo-based rankings, historical trend analysis, and head-to-head comparison.

Main site: [https://pingpong-elo.com/](https://pingpong-elo.com/)

中文版本: [README.md](README.md)

## What This Dataset Contains

The dataset focuses on one core question: who beat whom, when, in which event, and by what score. These match-level records are used to calculate Elo ratings, ranking movement, head-to-head records, and tournament impact.

It currently includes:

- **Player data**: names, English names, countries/regions, gender, seed Elo, aliases, avatars, and profile fields.
- **Tournament data**: event names, dates, locations, tiers, aliases, and Chinese/English display names.
- **Match data**: match results, game scores, rounds, disciplines, stages, and original scraped names.
- **Rating parameters**: tournament tiers, K factors, round multipliers, and singles/doubles/team weights.
- **Display dictionaries**: normalized mappings for Chinese tournament names, places, disciplines, and event-type phrases.

## Why This Dataset Exists

The ITTF/WTT world ranking is essentially a **tour points table**: players accumulate official points by entering events, advancing, and earning results, and those points expire over time. It works well for event qualification, seeding, and tour operations, but it is not always the same thing as true playing strength.

| Limitation of official points | Impact on strength evaluation |
|---|---|
| Points expire | Injury, rest, or fewer entries can lower ranking without a real strength drop |
| Entry volume matters | Frequent entries can accumulate points while selective players may be underrated |
| Event weights change | Official points across different years are hard to compare directly |
| Opponent strength is not measured directly | Beating a weaker player and beating an elite player are not separated strongly enough |

Elo treats each match as evidence about relative strength: beating a stronger opponent gains more, losing to a lower-rated opponent costs more, entry volume alone does not stack points, and inactivity does not directly reset a rating.

## Vision

PingPong ELO aims to provide a long-term, auditable, reusable foundation for table-tennis match results and strength ratings.

This dataset serves three goals:

1. **Reflect true strength**: evaluate players from match outcomes and opponent strength, not only official points.
2. **Preserve historical context**: make player performance traceable and comparable across eras and tournament systems.
3. **Accept public corrections**: keep data, fields, and calculation assumptions transparent enough for fans and researchers to audit.

## Layout

```text
data/
├── players.csv            # Player metadata: names, countries, gender, aliases, seed Elo, avatars
├── tiers.json             # Tournament tiers, K factors, round multipliers, discipline weights
├── tournaments.csv        # Canonical tournament metadata: slug, names, dates, tier, location, aliases
├── tournament_names.json  # Chinese display-name dictionary for tournaments, places, events, phrases
├── matches/               # Normalized match-result CSV files
│   ├── 2026_WTT_...
│   └── team/              # Individual match records from team events
├── avatars/               # Player avatar source images
├── sources/               # Raw or auxiliary collection-stage files
├── backup_clean/          # Pre-cleaning backups, not release data
└── logs/                  # Runtime logs, not release data
```

## Core Files

### `players.csv`

Player metadata. Common fields include:

| Field | Meaning |
|---|---|
| `slug` | Stable URL identifier |
| `name` | Chinese or canonical display name |
| `name_en` | English name or common international result name |
| `country` | Three-letter country/region code |
| `gender` | `M` / `F` |
| `seed_elo` | Initial Elo |
| `aliases` | Alternate spellings used for name normalization |
| `avatar_url` | Avatar path or source |

### `tournaments.csv`

Canonical tournament metadata. It is used to normalize event names from different sources into stable tournament entities.

Common fields include `slug`, `name`, `name_en`, `name_zh`, `tier`, `country`, `city`, `start_date`, `end_date`, and `aliases`.

### `tiers.json`

Tournament and round weighting configuration. Elo calculation adjusts K factors by tournament tier, round, and discipline type.

### `matches/`

Match-by-match results. This is the core of the dataset. Each CSV usually represents one tournament or one tournament collection.

## Match CSV Rules

Match files live in `data/matches/`. Individual matches from team events live in `data/matches/team/` and follow the same naming style.

### Files and Encoding

- CSVs are **UTF-8 (with BOM)**, first row is the header, one match record per line.
- File names look like `{year}_{EventName}.csv` (e.g. `2026_WTT_Champions_Doha.csv`), generated by `scripts/import_sources.py` from `sources/`; domestic Chinese events look like `{year}_CN_{EventName}.csv` (generated by `scraper/domestic_baike.py`).
- Dates use `YYYY-MM-DD`.
- Tournament tiers use the keys defined in `tiers.json`.
- Raw name fields (`player_a_raw` / `player_b_raw` / `winner_raw`) should be preserved where possible so normalized data can be traced back to its source.

### Match Fields (15 columns)

Standard match files have 15 columns:

| Field | Meaning | Required |
|---|---|---|
| `tournament` | Tournament display name | ✅ |
| `date` | Match date, `YYYY-MM-DD` | ✅ |
| `player_a` | Side A; the winner when `score_a > score_b` | ✅ |
| `player_b` | Side B; the winner when `score_a < score_b` | ✅ |
| `score_a` / `score_b` | Games won by each side; may be equal (see "Winners and Draws") | ✅ |
| `game_scores` | Per-game points, canonical format below | ⬜ (may be empty) |
| `round` | Round code (see "Round Codes") | ✅ |
| `tier` | Tournament tier key | ✅ |
| `sub_event` | Discipline code (see "Discipline Codes") | ⬜ |
| `event_id` | Event id from the source site | ⬜ |
| `stage` | Stage: `Qualification` / `Main Draw` / `Position Draw` / `Consolation`, etc. | ⬜ |
| `player_a_raw` / `player_b_raw` | Original player names before normalization | ⬜ |
| `winner_raw` | Winner name as explicitly recorded by the source; used only to judge whether a tied row is real, never as the direction of a draw | ⬜ |

Domestic Chinese event files may omit `winner_raw` (14 columns); all other columns stay the same.

### Canonical `game_scores` Format

- Canonical format is **`A-side game points|B-side game points`**: sides are separated by `|`, games within each side by commas, and the number of games on each side must equal `score_a` / `score_b`.
- Example: `11,11,8,11|6,9,11,8` means side A won 3 games (11, 11, 8, 11) and side B won 1 (6, 9, 11, 8), i.e. `score_a=3, score_b=1`.
- ⚠️ **Do not use the alternating format** (e.g. `11,6,11,9,8,11,11,8`), and do not use the raw ITTF website notation `11:1 11:6 11:4` (in source files that is `games_raw`; convert it to the canonical `11,11,11|1,6,4` before writing to `matches/`).
- Game points may be left empty, but when present, the game count per side must match `score_a` / `score_b`.

### Winners and Draws

- `score_a > score_b` → `player_a` wins; `score_a < score_b` → `player_b` wins.
- **A tie (`score_a == score_b`) is a legal result**: formats such as best-of-4 World Cup group matches (2-2) or best-of-5 team matches (3-3) can have no winner. Such rows are rated as 0.5 in Elo, `winner_id` stays empty, and no win/loss or head-to-head records are updated.
- A tied row is kept only when `winner_raw` exists and equals one of the two sides; however **`winner_raw` is never used to decide the direction of a tie** (the WTT source always fills the left `player_a` on 2-2, which is unreliable).
- In doubles/mixed, `player_a` / `player_b` join the two partners with **`/`** (e.g. `王楚钦/孙颖莎`). On load, the row is split into 1v1 sub-records per partner pairing, then merged again in the frontend; do not use any other separator.

### Discipline Codes (sub_event)

| Category | Codes |
|---|---|
| Singles | `MS` (men's) / `WS` (women's) |
| Doubles | `MD` (men's) / `WD` (women's) |
| Mixed doubles | `XD` |
| Team | `MT` (men's) / `WT` (women's) / `XT` (mixed) |
| Junior/youth | `U21MS`/`U21WS`, `JBS`/`JGS`, `CBS`/`CGS`, `MCBS`/`MCGS`, `HBS`/`HGS`, etc. (B = boys, G = girls) |

Gender inference (used to mark player gender): code contains `G` → female (`W`); contains `B` → male (`M`); ends with `XD`/`XT` → mixed (`X`, no gender info). Note `MCGS` starts with M but is actually a women's event — never judge by the first character alone.

### Round Codes

- `QUAL`: qualification. `QR128` through `QR2` are qualification rounds, with smaller numbers closer to the main draw.
- `GROUP`: group stage.
- `R128` / `R64` / `R32` / `R16`: main-draw rounds by remaining player count.
- `QF` / `SF` / `3RD` / `F`: quarterfinal, semifinal, bronze-medal match, final.
- `5-8` / `9-16`: placement rounds.
- Numeric rounds such as `2` / `4` / `8` / `16` / `256` also encode remaining player count. In `Position Draw`, `2` = bronze-medal match (`3RD`), `4` = 5-8, `8` = 9-16. `normalize_round` unifies all of these into canonical codes before writing.

### Deduplication Rules

The cross-file dedup key is **(tournament, date, sorted player pair, sub_event, round)**:

- Comparing the sorted player pair makes the key direction-agnostic: mirror rows that record the same match with A/B swapped are kept once.
- `sub_event` / `round` are part of the key, so a genuine rematch between the same pair in a different discipline (e.g. WD + WS) or a different round (e.g. group stage + knockout) is a real second match and is never dropped.
- A single file should also not contain fully duplicated same-date matches between the same players.

### Source File Format (sources/)

`data/sources/ittf_*.csv` / `wtt_*.csv` are raw scraper outputs with **22 columns**:
`tournament,date,player_a,player_b,country_a,country_b,score_a,score_b,game_scores,round,tier,sub_event,event_id,stage,player_a_raw,player_b_raw,player_x_raw,player_y_raw,result_raw,games_raw,winner_raw,year_raw`

- Player names use **English uppercase + country-code suffix** (e.g. `AGBODJAN Elias (TOG)`).
- `game_scores` in source files uses the raw notation (e.g. `11:1 11:6 11:4`); `import_sources.py` strips the extra columns and normalizes the header to the 15-column standard before writing to `matches/`.
- New source files must follow the same format (22 columns, English player names) or `import_sources.py` cannot process them.

### Team-Event Individual Matches (team/)

`data/matches/team/` holds individual rubbers from team events (e.g. German Bundesliga, Champions League) with **only the first 9 columns**: `tournament,date,player_a,player_b,score_a,score_b,game_scores,round,tier`. They are scored as team/individual entities on load (Elo weight ×0.7 for team events).

## Calculation Scope

Matches are processed in **date → stage → round** order to follow the real schedule as closely as possible: qualification first, then main draw, then placement matches.

Core Elo principles:

- Every match changes both players' ratings.
- Beating a stronger opponent is worth more.
- Upset losses cost more.
- Higher-tier events carry higher match weight.
- New or low-sample players carry higher uncertainty, so their ratings converge faster.

## What You Can Use It For

- Build table-tennis Elo rankings or historical rankings.
- Analyze player form, peaks, streaks, and long-term trends.
- Study head-to-head records and common-opponent performance.
- Measure how different event tiers affect player ratings.
- Create table-tennis visualizations, articles, leaderboards, or research projects.

## Corrections and Contributions

If you find incorrect names, nationalities, scores, tournament classifications, rounds, or dates, please open an issue or pull request.

When submitting a correction, please include:

- The file and row number.
- The correct match result or player information.
- A verifiable source link or screenshot note.
- For name issues, the Chinese name, English name, and common aliases where possible.

[Open a data correction issue](https://github.com/goesByhc/pingpong-elo-data/issues/new)

You can also start from the relevant player or tournament page on the main site: [https://pingpong-elo.com/](https://pingpong-elo.com/)
