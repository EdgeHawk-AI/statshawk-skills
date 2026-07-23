# Deriving analysis results from free-tier endpoints

The three `/v1/analysis/*` endpoints are paid-tier (10× units each) — but
every input they compute from is available on the free tier. These recipes
reproduce their results client-side. Each notes its real unit cost so you
can make the tradeoff deliberately: some derivations are cheap, some burn
through a 5,000-unit month fast.

Response shapes: list endpoints return `data.items`; game-log rows expose
stats as `line.phases[].measures.<stat>` (e.g. `measures.pts`).

## Player prop card → search + game-log (~3–4 units)

Reproduces the core of `/v1/analysis/player-prop` (season / last-5 / last-10
averages and hit rates vs a line):

```python
import os, requests
KEY = os.environ["STATSHAWK_API_KEY"]
BASE = "https://api.statshawk.ai/v1"
H = {"X-API-Key": KEY}

pid = requests.get(f"{BASE}/persons", headers=H,
                   params={"q": "Jayson Tatum", "limit": 1}).json()["data"]["items"][0]["id"]
games = requests.get(f"{BASE}/persons/{pid}/game-log", headers=H,
                     params={"competition": "nba", "season": 2026}).json()["data"]["items"]

line, stat = 27.5, "pts"
vals = [p["measures"][stat] for g in games for p in g["line"]["phases"] if stat in p["measures"]]
def window(vs): return {"avg": sum(vs)/len(vs), "hit_rate": sum(v > line for v in vs)/len(vs)}
print({"season": window(vals), "last5": window(vals[-5:]), "last10": window(vals[-10:])})
```

What the paid card adds over this: home/away splits (deriving those needs a
per-game contest lookup — many extra calls), matchup context, and one call
instead of two. For a single player question, DIY is fine; for repeated prop
work, the card is faster and cheaper in practice.

## Slate stat-board → contests + rosters + game-logs (200+ units — be careful)

Reproduces `/v1/analysis/stat-board` (rank a day's players by over-rate):

1. `GET /competitions/{comp}/editions/{year}/contests?date=` — the slate (1×)
2. For each contest, both team rosters: `GET /teams/{id}/roster` (3× each)
3. For each candidate player, `GET /persons/{id}/game-log` (2× each)
4. Compute each player's over-rate for the stat/line over your window; sort.

Honest math for a 10-game slate: 1 + 20 rosters × 3 + ~100 game-logs × 2 ≈
**260+ units** to reproduce what the stat-board returns for 10. Narrow the
candidate list (stars only, one position) to cut cost. If you do this
regularly, the paid stat-board pays for itself almost immediately.

## Team comparison → stats + game-logs + standings (~9 units)

Reproduces `/v1/analysis/compare-teams` for two teams:

1. `GET /teams/{a}/stats?competition=&season=` and the same for `{b}` (2× each)
2. `GET /teams/{a}/game-log` and `{b}` for recent form (2× each)
3. `GET /competitions/{comp}/editions/{year}/standings` for rank context (1×)

Then present side-by-side: record, points for/against, recent form (last-5
results from the game logs), standings position. Cost is comparable to the
paid card; the card's value here is the pre-computed framing, not savings.

## Rules of thumb

- Single player, single question → DIY from the game log; it's cheap.
- Anything slate-shaped or repeated → the derivation cost explodes; that is
  exactly what the analysis tier is for.
- Always tell the user which path you took and roughly what it cost in units
  (their dashboard shows the running total).
