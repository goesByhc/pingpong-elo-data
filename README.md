# PingPong ELO 数据集 / PingPong ELO Dataset

乒乓球 ELO 排名的公开开放数据集（数据 + 比赛 CSV），由 [PingPong ELO](https://github.com/goesByhc/pingpong-elo) 站点维护发布。
本仓库作为该站点的数据子模块（submodule）独立维护，应用代码、爬虫、ELO 引擎与部署配置保留在私有的主仓库中。

Public open dataset (data + match CSVs) for table-tennis ELO ratings, maintained and published by the
[PingPong ELO](https://github.com/goesByhc/pingpong-elo) site. This repo is maintained independently as the
data submodule of that site; application code, scrapers, the Elo engine, and deployment config stay private.

## 目录结构 / Layout

```text
data/
├── players.csv            # 选手元数据：别名、种子分、画像字段 / Player metadata, aliases, seed Elo, profile fields
├── tiers.json             # 赛事级别、K 系数、轮次系数 / Tournament tiers, K factors, round multipliers
├── tournaments.csv        # 赛事规范元数据（slug/名称/日期/级别）/ Canonical tournament metadata
├── tournament_names.json  # 英文→中文赛事名词典（四级）/ EN→ZH display-name dictionary
├── matches/               # 比赛结果 CSV / Match-result CSV files
│   ├── 2026_WTT_...
│   └── team/              # 团体赛个人盘次 CSV / Optional team-event CSV files
├── avatars/               # 选手头像原图 / Player avatar source images
├── sources/               # 采集阶段原始文件 / Raw or auxiliary collection-stage files
├── backup_clean/          # 清洗前备份（不发布）/ Pre-cleaning backups (not published)
└── logs/                  # 运行日志（爬虫/脚本，不发布）/ Runtime logs (not published)
```

## 比赛 CSV 规则 / Match CSV Rules

比赛文件位于 `data/matches/`（团体赛个人盘次在 `data/matches/team/`，命名风格相同）。

- 当 `score_a > score_b` 时 `player_a` 为胜者；日期格式 `YYYY-MM-DD`。
- 级别缺失或为 `OTHER` 时从 `tiers.json` 读取。
- When `score_a > score_b`, `player_a` is the winner. Dates use `YYYY-MM-DD`; tournament tier is read from
  `tiers.json` when missing or set to `OTHER`.

### 列说明 / Columns

| Column | Meaning / 含义 |
|---|---|
| `tournament` | Tournament display name (EN) / 赛事显示名（英文） |
| `date` | Match date, `YYYY-MM-DD` / 比赛日期 |
| `player_a` | Winner (when `score_a > score_b`) / 胜者 |
| `player_b` | Loser / 败者 |
| `score_a` / `score_b` | Games won (e.g. `4`,`1`) / 局数 |
| `game_scores` | Per-game points, e.g. `11,3,11,11,11\|8,11,9,5,5` / 每局小分 |
| `round` | Stage within the event. See round codes below / 轮次，见下方轮次代码 |
| `tier` | Tournament tier key (see `tiers.json`) / 赛事级别键 |
| `sub_event` | Discipline code: `MS`/`WS`/`MT`/`WT`/`MD`/`WD`/`XD` etc. / 项目代码 |
| `event_id` | ITTF/WTT event id when scraped / 抓取时的赛事 id |
| `stage` | Draw phase: `Qualification` / `Main Draw` / `Position Draw` / 阶段 |
| `player_a_raw` / `player_b_raw` | Original name strings (pre-normalization) / 归一化前的原始姓名 |

### 轮次代码 / Round Codes

- `QUAL` — 资格赛；`QR128`…`QR2` — 资格赛轮次（数字越小越靠后）。
- `GROUP` — 小组赛。
- `R128` / `R64` / `R32` / `R16` — 正赛轮次（剩余人数）。
- `QF` / `SF` / `F`（含 `QuarterFinal` / `SemiFinal` / `Final`）— 淘汰赛。
- 数字轮次（`2`/`4`/`8`/`16`/`256`）编码**剩余人数**（如 `2`=决赛、`4`=半决赛、`8`=四分之一决赛）；
  `Position Draw` 中表示名次赛（`2`=铜牌战）。

### 算分顺序 / Elo Computation Order

比赛按 **date → stage → round** 顺序处理，球员积分沿真实赛程演进（先资格赛、再正赛、最后名次赛）。
详见 loader 中的 `_stage_rank` / `_round_rank`。

Matches are processed in **date → stage → round** order so ratings evolve along the real schedule
(qualification first, then main draw, placement matches last). See `_stage_rank` / `_round_rank` in the loader.

### 赛事中文名 / Tournament Chinese Names

导出时由 `scripts/tournament_display.py::tournament_name_zh()` 按四级词典生成：`exact`（全名精确）→
`phrases`（赛事类型短语，最长优先）→ `events`（项目词，如 `Men's Singles`→`男单`）→ `places`（城市/大洲/国家）。
新增中文名请编辑 `tournament_names.json`（长键优先匹配，如 `Tunisia` 放在 `Tunis` 之前）。

### 赛事规范表 / Tournaments CSV

`tournaments.csv` 为规范赛事元数据：`slug`、`name`、`name_en`、`name_zh`、`tier`、`country`、`city`、
`start_date`、`end_date`、`aliases`。loader 用抓取的赛事名匹配 `name` / `aliases`，匹配失败则回退到自动 slug（带警告）。

## 如何贡献 / Contributing

### 报告数据问题 / Report a data issue

发现姓名、国籍、比分、赛事归类等数据错误时，请在本仓库提交 issue（网站页面上的「提交数据修正」按钮
已自动预填问题模板与上下文，指向本仓库的 issues）：

- 中文：[新建 Issue（自动填入模板）](https://github.com/goesByhc/pingpong-elo-data/issues/new?title=%5B%E6%95%B0%E6%8D%AE%E4%BF%AE%E6%AD%A3%5D&body=%E8%AF%B7%E6%8F%8F%E8%BF%B0%E4%BD%A0%E5%8F%91%E7%8E%B0%E7%9A%84%E6%95%B0%E6%8D%AE%E9%97%AE%E9%A2%98)
- English: [New Issue (pre-filled template)](https://github.com/goesByhc/pingpong-elo-data/issues/new)

If you find wrong names, nationalities, scores, or tournament classification, please open an issue in this
repo. The "提交数据修正" (report data fix) button on the site links here with a pre-filled template.

### 直接修数据 / Fix data directly

欢迎 fork 本仓库并提交 Pull Request：

1. Fork 并克隆，修改对应 CSV（`players.csv` / `matches/*.csv` / `tournaments.csv` 等）。
2. 保持列格式、日期 `YYYY-MM-DD`、胜者在前（`score_a > score_b` 时 `player_a` 为胜者）。
3. 提交说明用中文或英文均可；PR 标题请加 `[data]` 前缀，例如 `[data] fix score of ...`。

欢迎任何数据修正的 Pull Request。确认后会同步到站点数据并感谢贡献。
