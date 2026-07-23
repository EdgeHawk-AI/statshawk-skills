# StatsHawk REST endpoint reference

Base URL `https://api.statshawk.ai` — auth header `X-API-Key: $STATSHAWK_API_KEY`.
Tier `paid` = returns `403 TIER_REQUIRES_PAID` on the free tier.

| Endpoint | Summary | Tier | Query params |
|---|---|---|---|
| `GET /v1/analysis/compare-teams` | clubs OR national teams. | paid | a, b, season, as_of, edition, days |
| `GET /v1/analysis/player-prop` | person-keyed. | paid | person_id, stat, line, competition, season, edition, from, to, stage |
| `GET /v1/analysis/stat-board` | slate over-rate | paid | competition, date, stat, window, limit, min_games |
| `GET /v1/assets/team-logo/{team_id}` | Get team logo PNG. | any | — |
| `GET /v1/competitions` | the competition index (all competitions, flat). | any | — |
| `GET /v1/competitions/{comp}` | sport, kind, name (resolved by slug). | any | — |
| `GET /v1/competitions/{comp}/capabilities` | the per-phase measure inventory | any | — |
| `GET /v1/competitions/{comp}/editions` | editions of a competition. | any | — |
| `GET /v1/competitions/{comp}/editions/{year}` | GET /v1/competitions/{comp}/editions/{year} | any | — |
| `GET /v1/competitions/{comp}/editions/{year}/contests` | GET /v1/competitions/{comp}/editions/{year}/contests?date= | any | date |
| `GET /v1/competitions/{comp}/editions/{year}/games` | GET /v1/competitions/{comp}/editions/{year}/games?date= | any | date |
| `GET /v1/competitions/{comp}/editions/{year}/matchups` | the | any | date |
| `GET /v1/competitions/{comp}/editions/{year}/standings` | the app-materialized | any | — |
| `GET /v1/competitions/{comp}/editions/{year}/teams` | participants. | any | — |
| `GET /v1/contests/{contest_id}` | assembled team-Match row. | any | — |
| `GET /v1/contests/{contest_id}/boxscore` | appearances + typed stat lines. | any | — |
| `GET /v1/contests/{contest_id}/detail` | generalized contest row. | any | — |
| `GET /v1/contests/{contest_id}/matchup` | the pregame matchup card | any | — |
| `GET /v1/contests/{contest_id}/play-by-play` | the Statcast play-by-play | any | detail, pitcher_id, batter_id |
| `GET /v1/persons` | the person index / name search, paged. Store-only: a | any | q, limit, offset |
| `GET /v1/persons/{person_id}` | assembled profile (one row, no joins). | any | — |
| `GET /v1/persons/{person_id}/appearances` | GET /v1/persons/{per_id}/appearances | any | competition, season, edition, from, to, role, phase, stage |
| `GET /v1/persons/{person_id}/capabilities` | which phases/measures this person | any | competition, season |
| `GET /v1/persons/{person_id}/game-log` | the person's chronological game log | any | competition, season, edition, from, to, role, phase, stage |
| `GET /v1/persons/{person_id}/memberships` | GET /v1/persons/{per_id}/memberships | any | — |
| `GET /v1/persons/{person_id}/stats` | GET /v1/persons/{per_id}/stats | any | competition, season |
| `GET /v1/teams` | the team index, filtered by sport / competition / team | any | sport, competition, traits, limit, offset |
| `GET /v1/teams/{team_id}` | assembled team row. | any | — |
| `GET /v1/teams/{team_id}/game-log` | the team's chronological game log | any | competition, season, edition, from, to |
| `GET /v1/teams/{team_id}/games` | GET /v1/teams/{team_id}/games | any | competition, season, edition, from, to |
| `GET /v1/teams/{team_id}/roster` | the as-of roster surface. | any | season, as_of, edition, days |
| `GET /v1/teams/{team_id}/stats` | the materialized | any | competition, season |

Pagination (`limit`/`offset`) exists only on `/v1/persons` and `/v1/teams`.
`/v1/analysis/stat-board`'s `limit` is a row cap, not pagination.

Interactive playground + full schemas: https://statshawk.ai/docs/api
