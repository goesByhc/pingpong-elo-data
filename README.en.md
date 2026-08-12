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

- Dates use `YYYY-MM-DD`.
- When `score_a > score_b`, `player_a` is the winner.
- Tournament tiers use keys defined in `tiers.json`.
- A single file should not contain fully duplicated same-date matches between the same players.
- Raw name fields should be preserved where possible so normalized data can be traced back to its source.

### Match Fields

| Field | Meaning |
|---|---|
| `tournament` | Tournament display name |
| `date` | Match date, `YYYY-MM-DD` |
| `player_a` | Side A, usually the winner |
| `player_b` | Side B, usually the loser |
| `score_a` / `score_b` | Games won by each side |
| `game_scores` | Per-game points, for example `11,3,11,11,11\|8,11,9,5,5` |
| `round` | Round code |
| `tier` | Tournament tier key |
| `sub_event` | Discipline code, such as `MS` / `WS` / `MT` / `WT` / `MD` / `WD` / `XD` |
| `event_id` | Event id from the source site |
| `stage` | Stage, such as `Qualification` / `Main Draw` / `Position Draw` |
| `player_a_raw` / `player_b_raw` | Original player names before normalization |

### Round Codes

- `QUAL`: qualification. `QR128` through `QR2` are qualification rounds, with smaller numbers closer to the main draw.
- `GROUP`: group stage.
- `R128` / `R64` / `R32` / `R16`: main-draw rounds by remaining player count.
- `QF` / `SF` / `F`: quarterfinal, semifinal, final.
- Numeric rounds such as `2` / `4` / `8` / `16` / `256` also encode remaining player count. In `Position Draw`, `2` can mean the bronze-medal match.

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
