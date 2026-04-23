# ⚽ AMSA Premier Division — Season Predictor

> A single-file, browser-based soccer league analyzer and end-of-season predictor built with vanilla HTML/CSS/JavaScript. No frameworks, no dependencies, no server required.

![League Dashboard](https://img.shields.io/badge/Season-Fall%202025%20%2B%20Spring%202026-c8ff00?style=flat-square&labelColor=1a2e1a)
![Simulation](https://img.shields.io/badge/Poisson%20Model-10%2C000%20runs-f0c040?style=flat-square&labelColor=1a2e1a)
![Status](https://img.shields.io/badge/Status-Active%20Season-brightgreen?style=flat-square&labelColor=1a2e1a)

---

## 📋 Overview

This project tracks the **AMSA 2025–2026 Premier Division** Sunday league season across two halves (Fall 2025 and Spring 2026). It ingests all match results, computes live standings, and uses a **Poisson Distribution model** with 10,000 Monte Carlo simulations to predict how the season ends — including title probabilities, relegation risk, and detailed team strength ratings for every team.

All logic runs entirely in the browser. Open the HTML file and everything works offline.

---

## ✨ Features

- **Live Standings Table** — points, W/D/L, goal difference, and full-season form dots for all 10 teams
- **Poisson Season Predictor** — 10,000 full-season simulations aggregated into stable, meaningful statistics
- **Title Probability %** — how often each team wins the league across all 10,000 simulations
- **Relegation Probability %** — how often each team finishes 9th or 10th (the relegation zone)
- **Top 3 Probability %** — likelihood of a podium finish
- **Points Range** — 10th–90th percentile spread showing prediction certainty vs. uncertainty
- **Most Likely Position** — the rank a team lands in most frequently, with confidence %
- **📐 Team Strength Tab** — Poisson attack/defense ratings with visual bars and expected goals per game
- **Remaining Fixtures** — all 20 unplayed Spring 2026 matches with Poisson win probabilities and expected goals (λ) per fixture
- **Full Results Archive** — every played match organized by week, for both Fall 2025 and Spring 2026
- **Full Form & Stats Cards** — complete season form (every game, not just recent), home/away splits, biggest win/loss, clean sheets, current streak, and Poisson ratings per team
- **Relegation Zone Highlighting** — teams with >50% relegation probability are visually flagged in red

---

## 🏟️ Teams

| Team | Division |
|---|---|
| ATX United - Red | Premier |
| ATX United - Gold | Premier |
| ATX United - White | Premier |
| Austin Athletic Club | Premier |
| Austin Thunder | Premier |
| Caledonians | Premier |
| Lokomotiv Zilker | Premier |
| Merit FC | Premier |
| TBD FC | Premier |
| VTX United FC | Premier |

---

## 📅 Season Structure

| Season | Weeks | Status |
|---|---|---|
| Fall 2025 | Weeks 1–14 (Sep 7 – Feb 1) | ✅ Complete |
| Spring 2026 | Weeks 1–9 (Feb 8 – Apr 19) | ✅ Played |
| Spring 2026 | Weeks 10–14 (Apr 26 – May 24) | 🔮 Remaining |

**Relegation rule:** Teams finishing **9th and 10th** at the end of Spring 2026 are relegated.

---

## 🔮 How the Prediction Works

### Why Poisson Distribution?

The **Poisson Distribution** is the standard model used by professional sports analysts and betting companies for soccer prediction. Unlike a simple points-per-game approach, Poisson models *goals* — the fundamental unit of soccer — and derives win/draw/loss probabilities from the full distribution of possible scorelines.

It is a significant upgrade over basic Monte Carlo because it accounts for *how* a team scores, not just whether they win.

### Step 1 — Calibrate Team Strength Ratings

From all played results, each team receives two ratings:

```
Attack strength  = ((home goals scored / home games) / league avg home goals) × 0.55
                 + ((away goals scored / away games) / league avg away goals) × 0.45

Defense strength = ((home goals conceded / home games) / league avg away goals) × 0.55
                 + ((away goals conceded / away games) / league avg home goals) × 0.45
```

A value **> 1.0** means above average. A defense strength **< 1.0** means the team concedes fewer goals than average (better defense).

### Step 2 — Compute Expected Goals (λ) Per Match

For each remaining fixture, expected goals are derived from the two teams' ratings:

```
λ_home = attack_home × defense_away × league_avg_home_goals
λ_away = attack_away × defense_home × league_avg_away_goals
```

The home advantage is built into the league averages — home teams naturally score more on average, so `league_avg_home_goals > league_avg_away_goals`.

### Step 3 — Build a Scoreline Probability Matrix

Using the Poisson formula `P(X = k) = e^(−λ) × λ^k / k!`, a full 9×9 probability matrix is computed for every possible scoreline from 0–0 to 8–8. Win, draw, and loss probabilities are derived by summing the relevant cells:

```
P(home win) = sum of all cells where home goals > away goals
P(draw)     = sum of all cells where home goals = away goals
P(away win) = sum of all cells where away goals > home goals
```

### Step 4 — Simulate 10,000 Full Seasons

For each of the 10,000 simulation runs, every remaining fixture is resolved by randomly sampling a scoreline from the Poisson distributions for that specific match. The full season table is then computed and each team's final rank is recorded.

### Step 5 — Aggregate Results

After all 10,000 runs, per team:

| Metric | How it's computed |
|---|---|
| **Median pts** | Middle value of the 10,000 final point totals |
| **Pts range (p10–p90)** | 10th and 90th percentile of the points distribution |
| **Title %** | % of runs where the team finished 1st |
| **Top 3 %** | % of runs where the team finished in the top 3 |
| **Relegation %** | % of runs where the team finished 9th or 10th |
| **Most likely position** | The single most frequent final rank across all runs |

---

## 🔄 Data Refresh

When you refresh the page, the simulation reruns from scratch — all 10,000 Poisson simulations execute again in your browser. Here's exactly what happens:

**What stays the same every refresh:**
- All played match results (hardcoded in the JS arrays)
- Current standings — deterministic, always identical
- Team strength ratings (Attack/Defense) — calculated from real results
- Home/Away splits, biggest win/loss, clean sheets — all from real data
- Form dots for all played games

**What varies slightly between refreshes:**
- The 10,000 simulations use `Math.random()` for each remaining fixture, so the random seed differs every time
- Title %, Relegation %, Top 3 %, Median pts, Points range — these will fluctuate by roughly ±0.5–1% each refresh

However, because it's aggregated over 10,000 runs, the variance is very small. For example:
- A team showing 42.3% title chance might show 41.8% or 42.7% on the next refresh
- The most likely position for each team almost never changes between refreshes
- Rankings in the predicted table are stable

> **In short:** The predictions are statistically stable — refreshing gives you essentially the same answer every time, just with tiny random noise. If you want perfectly identical results on every refresh, a fixed random seed (seeded PRNG) can be added to make the simulation 100% deterministic — see the Customization section below.

---

## 🚀 Usage

### Run locally

```bash
# No installation needed. Just open the file.
open league-predictor.html
```

Or serve it with any static file server:

```bash
npx serve .
# then open http://localhost:3000/league-predictor.html
```

### Update with new results

All match data lives in three JavaScript arrays near the top of `league-predictor.html`:

```javascript
// Fall 2025 results (complete)
const FALL = [
  {w:1, wl:'Week 1 — Sep 7', h:'ATX United - Red', a:'TBD FC', hg:4, ag:2},
  // ...
];

// Spring 2026 results (played weeks)
const SPRING = [
  {w:1, wl:'Week 1 — Feb 8', h:'Merit FC', a:'VTX United FC', hg:3, ag:1},
  // ...
];

// Spring 2026 remaining fixtures (no scores yet)
const REMAINING = [
  {w:10, wl:'Week 10 — Apr 26', h:'ATX United - Red', a:'ATX United - White'},
  // ...
];
```

**To add a new result:** move the fixture from `REMAINING` to `SPRING`, adding `hg` (home goals) and `ag` (away goals):

```javascript
// Before (in REMAINING):
{w:10, wl:'Week 10 — Apr 26', h:'ATX United - Red', a:'ATX United - White'}

// After (move to SPRING with score):
{w:10, wl:'Week 10 — Apr 26', h:'ATX United - Red', a:'ATX United - White', hg:2, ag:1}
```

Save the file and refresh — standings, Poisson ratings, and predictions all update automatically.

---

## 📊 Dashboard Tabs

| Tab | Description |
|---|---|
| **Current Standings** | Live points table with full-season form dots |
| **🔮 Season Prediction** | Aggregated Poisson simulation — title %, relegation %, points ranges, team cards |
| **📐 Team Strength** | Poisson attack/defense ratings with visual bars and expected goals per game |
| **Results** | Full match history organized by week for Fall 2025 and Spring 2026 |
| **Remaining Fixtures** | All 20 unplayed Spring 2026 matches with Poisson win % and λ per game |
| **Form & Stats** | Full-season form trail, home/away split, biggest win/loss, streaks, Poisson ratings |

---

## 📈 Interpreting Results

| Metric | What it means |
|---|---|
| **Median pts** | The "expected" final points — half of 10,000 simulations landed above, half below |
| **Pts range (p10–p90)** | Narrow range = predictable; wide range = volatile/uncertain season finish |
| **Title %** | 40%+ = strong favorite; under 5% = long shot |
| **Relegation %** | Over 50% = in serious danger; over 80% = almost certain relegation |
| **Most likely pos** | The single most common finishing rank, with % confidence |
| **Attack strength** | > 1.0 = scores more goals than league average |
| **Defense strength** | < 1.0 = concedes fewer goals than league average (lower is better) |
| **xG per game** | Expected goals per game based on Poisson model — more reliable than raw averages |
| **λ (lambda)** | Expected goals in a specific upcoming fixture, shown on the Fixtures tab |

---

## 🗂️ Project Structure

```
league-predictor.html   # Everything — HTML, CSS, JS, data, and simulation in one file
README.md               # This file
```

The entire project is intentionally a **single self-contained file** for maximum portability. No build step, no npm, no bundler.

---

## 🛠️ Technical Details

- **Language:** Vanilla JavaScript (ES6+)
- **Prediction model:** Poisson Distribution with 10,000-run Monte Carlo aggregation
- **Styling:** CSS custom properties, CSS Grid, CSS animations
- **Fonts:** Bebas Neue (display) + DM Mono (body) via Google Fonts
- **Simulation:** Runs synchronously in the browser on every page load (~80ms)
- **No external dependencies** beyond Google Fonts (fully functional offline if fonts are cached)
- **Browser support:** All modern browsers (Chrome, Firefox, Safari, Edge)

---

## 🔧 Customization

### Change number of simulation runs

```javascript
const RUNS = 10000;  // increase for more accuracy, decrease for speed
```

### Adjust the maximum scoreline considered

```javascript
const MAX_GOALS = 8;  // 0–8 goals per team considered in the Poisson matrix
```

Increasing this slightly improves accuracy for high-scoring leagues but has negligible effect for most Sunday leagues.

### Change relegation zone size

The relegation zone is defined as finishing 9th or 10th (`i >= 8` in zero-indexed rank):

```javascript
if (i >= 8) agg[t.team].relCount++;  // change 8 to adjust zone size
```

### Add a fixed random seed (fully reproducible results)

Replace `Math.random()` with a seeded PRNG for identical results on every page load:

```javascript
// Simple seeded PRNG (mulberry32)
function mulberry32(seed) {
  return function() {
    seed |= 0; seed = seed + 0x6D2B79F5 | 0;
    let t = Math.imul(seed ^ seed >>> 15, 1 | seed);
    t = t + Math.imul(t ^ t >>> 7, 61 | t) ^ t;
    return ((t ^ t >>> 14) >>> 0) / 4294967296;
  };
}
const rand = mulberry32(42);  // change 42 to any seed
// then replace Math.random() calls with rand()
```

---

## 📝 License

MIT — free to use, modify, and share.

---

*Built for AMSA Sunday League · Austin, TX · 2025–2026 Season*
