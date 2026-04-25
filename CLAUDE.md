# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Diamond EV+ is a single-file vanilla JS web app (`index.html`) that surfaces MLB home run prop bet picks by computing an expected value (EV) score for each batter.

## Development

No build step, package manager, or dev server is required. Open `index.html` directly in a browser or serve it with any static file server:

```bash
python3 -m http.server 8080
```

There are no tests, linters, or CI pipelines configured.

## Architecture

Everything lives in `index.html`: HTML structure, CSS, and JavaScript in a single `<script>` block.

### Data pipeline (`load()`)

Six async fetches run in parallel via `Promise.all`, each wrapped in a 4-second timeout via `safe()`:

| Source | Function | What it provides |
|---|---|---|
| MLB Stats API | `getPlayers()` | Season hitting stats (ISO, SLG, HR, PA, K%) |
| MLB Stats API | `getPitchers()` | Season pitching stats (HR/9 per pitcher) |
| MLB Stats API | `getGames()` | Today's schedule + probable pitchers |
| MLB Stats API | `getLineups()` | Active roster to filter to starters |
| The Odds API | `getOdds()` | Best available HR prop odds per player |
| Open-Meteo | `getWeather()` | Current weather at a hardcoded lat/lng |

### Scoring model

`score(player, pitcher, weather)` combines:
- Batter power metrics: ISO × 50, SLG × 35, HR rate × 40, K% × −20
- Pitcher HR/9 × 30 (higher = easier matchup)
- `parkBoost` map (hardcoded per home team)
- `weatherBoost`: +5 if temp > 85°F, +5 if wind > 10 mph

`calcEV(score, odds)` converts the score to an implied probability (capped at 70%), then computes `(prob × payout) − (1 − prob)`.

### Key constants / config points

- `ODDS_KEY` — The Odds API key, hardcoded at the top of the script
- `parkBoost` — Static map of team name → run-environment adjustment
- Weather coordinates — Hardcoded to Cincinnati (lat 39.1, lng −84.5); not game-specific
- EV cap: `score/150` is the probability formula; 150 is the normalization ceiling
- Top 15 players by EV are rendered
