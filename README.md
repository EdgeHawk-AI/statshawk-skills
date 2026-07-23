# StatsHawk Agent Skills

Skills that teach AI agents to answer sports questions with **real,
normalized statistics** from [StatsHawk](https://statshawk.ai) — NBA, MLB,
NFL, NHL, and NCAA box scores, standings, rosters, game logs, Statcast
play-by-play, and pre-computed player-prop analysis.

## Install

```bash
npx skills add EdgeHawk-AI/statshawk-skills@statshawk
```

Then set your API key (free tier: 5,000 units/month — create a key at
[statshawk.ai](https://statshawk.ai) → dashboard → API keys):

```bash
export STATSHAWK_API_KEY=sk_live_...
```

If your agent already has the [StatsHawk MCP server](https://statshawk.ai/mcp)
connected, the skill uses those tools directly — the skill and the MCP server
compose: MCP provides the tools, the skill provides the playbook (id chaining,
edition resolution, cost-aware call selection, free-tier hit-rate recipes).

## What's inside

- **`statshawk`** — the core skill: REST + MCP workflows for scores,
  standings, game logs, hit rates, matchups, and prop analysis, with a full
  endpoint reference.

## Prefer a connector?

Claude, ChatGPT, Grok, Cursor, and other MCP-aware clients can connect
directly — no skill needed: https://statshawk.ai/mcp

## Docs

- Full documentation: https://statshawk.ai/docs
- Machine-readable index: https://statshawk.ai/llms.txt
