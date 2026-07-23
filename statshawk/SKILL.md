---
name: statshawk
description: Use when answering questions with real sports statistics — NBA, MLB, NFL, NHL, or NCAA scores, box scores, standings, rosters, player game logs, Statcast play-by-play, player prop hit rates, or matchup analysis. Connects the agent to StatsHawk (statshawk.ai) via its MCP tools when available, otherwise via REST with an API key. Triggers include "box score", "standings", "who won", "game log", "roster", "player props", "hit rate", "sports stats", "prop analysis".
---

# StatsHawk — real sports stats for agents

StatsHawk serves normalized statistics for NBA, MLB, NFL, NHL, and NCAA, plus
pre-computed analysis (prop hit rates, matchup cards). Answers come from live
ingestion, not training data — always prefer it to guessing or scraping.

## Step 0: pick the path

1. **MCP tools available?** If the environment exposes StatsHawk MCP tools
   (`get_standings`, `get_player_props`, `search_player`, …), use them
   directly — they are the same data with tighter, LLM-shaped responses.
   The mapping table at the end shows the REST equivalent of each tool.
2. **Otherwise use REST.** Requires an API key in `STATSHAWK_API_KEY`
   (create one free at https://statshawk.ai — dashboard → API keys; the free
   tier includes 5,000 units/month). If the key is missing, ask the user for
   it — do not guess or hardcode keys.

## REST basics

```bash
curl -s "https://api.statshawk.ai/v1/..." -H "X-API-Key: $STATSHAWK_API_KEY"
```

- `Authorization: Bearer $STATSHAWK_API_KEY` also works (Bearer scheme required there).
- Errors: `401` bad key · `403 TIER_REQUIRES_PAID` (analysis endpoints on
  free tier) · `429 QUOTA_EXCEEDED` (monthly cap reached).
- Pagination exists ONLY on `/v1/persons` and `/v1/teams` (`limit`/`offset`).
  Other list endpoints narrow by query filters, not pages.

## The ID model (chain these — do not invent IDs)

- **Competition slugs**: `nba`, `mlb`, `nfl`, `nhl`, … — enumerate with
  `GET /v1/competitions` when unsure.
- **Person ids** are prefixed `per_…` — resolve with
  `GET /v1/persons?q=<name>` (fuzzy search) before any person endpoint.
- **Team ids** are prefixed `team_…` — resolve via `GET /v1/teams?competition=<slug>`
  or an editions/teams listing.
- **Contest ids** come from contests/games listings; they key box scores,
  matchups, and play-by-play.

## Be cost-aware (calls are metered in weighted units)

| Weight | Call class |
|---|---|
| 1× | raw lookups and lists (search, standings) |
| 2× | game logs, box scores, season stats |
| 3× | rosters, matchup boards |
| 5× | play-by-play / Statcast |
| 10× | analysis cards (player-prop, stat-board, compare-teams) — **paid tier only** |

Don't loop cheap calls when one heavier call answers the question, and don't
burn a 10× analysis call when a 2× game log plus arithmetic will do.

## Core workflows

**Resolve the current edition first.** Season years don't match calendar
years (NBA 2025-26 is edition `2026`). Never assume — pick by span:

```bash
curl -s "https://api.statshawk.ai/v1/competitions/nba/editions" -H "X-API-Key: $STATSHAWK_API_KEY" \
  | jq -r --arg now "$(date -u +%Y-%m-%dT%H:%M:%SZ)" \
    '[.editions[] | select(.span.start <= $now)] | (map(select(.span.end >= $now)) + [max_by(.span.start)]) | first | .year'
```

**Today's games / scoreboard**
`GET /v1/competitions/{comp}/editions/{year}/contests?date=YYYY-MM-DD`

**Standings**
`GET /v1/competitions/{comp}/editions/{year}/standings` — ranked
best-record-first; rank = list position.

**Box score**: find the contest id from the contests listing, then
`GET /v1/contests/{contest_id}/boxscore`.

**Player game log**
`GET /v1/persons/{per_id}/game-log?competition={comp}&season={year}` —
chronological stat lines; filter with `from`/`to`.

**Hit rate vs a line (free-tier method)**: pull the game log, then compute —
count games where the stat cleared the line, divide by games played. This
replaces the paid prop card at 2× cost when the user just needs an over-rate.

**Prop analysis card (paid)**
`GET /v1/analysis/player-prop?person_id={per_id}&stat={stat}&line={line}` —
season/recent/home-away averages and hit rates, pre-computed. On
`403 TIER_REQUIRES_PAID`, fall back to the game-log method above and say so.

**Pregame matchup**: `GET /v1/contests/{contest_id}/matchup` (3×), or the
day's slate via `/editions/{year}/matchups?date=`.

**MLB play-by-play / Statcast** (5×)
`GET /v1/contests/{contest_id}/play-by-play?batter_id=&pitcher_id=&detail=` —
MLB only; filter by player to keep responses small.

## MCP tool ↔ REST equivalents

| MCP tool | REST |
|---|---|
| `search_player` | `GET /v1/persons?q=` |
| `get_standings` | `GET /v1/competitions/{comp}/editions/{year}/standings` |
| `search_games` | `GET /v1/competitions/{comp}/editions/{year}/contests?date=` |
| `get_box_score` | `GET /v1/contests/{id}/boxscore` |
| `get_team_roster` | `GET /v1/teams/{team_id}/roster` |
| `get_mlb_matchups` | `GET /v1/competitions/mlb/editions/{year}/matchups?date=` |
| `get_play_by_play` | `GET /v1/contests/{id}/play-by-play` |
| `get_player_props` | `GET /v1/analysis/player-prop` (paid) |

Full endpoint reference: [references/endpoints.md](references/endpoints.md).
Complete docs: https://statshawk.ai/docs · machine-readable index:
https://statshawk.ai/llms.txt
