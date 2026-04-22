
# ⚽ AMSA Premier Division — Season Predictor

> A single-file, browser-based soccer league analyzer and end-of-season predictor built with vanilla HTML/CSS/JavaScript. No frameworks, no dependencies, no server required.

![League Dashboard](https://img.shields.io/badge/Season-Fall%202025%20%2B%20Spring%202026-c8ff00?style=flat-square&labelColor=1a2e1a)
![Simulation](https://img.shields.io/badge/Monte%20Carlo-1%2C000%20runs-f0c040?style=flat-square&labelColor=1a2e1a)
![Status](https://img.shields.io/badge/Status-Active%20Season-brightgreen?style=flat-square&labelColor=1a2e1a)

---

## 📋 Overview

This project tracks the **AMSA 2025–2026 Premier Division** Sunday league season across two halves (Fall 2025 and Spring 2026). It ingests all match results, computes live standings, and runs a **Monte Carlo simulation** to predict how the season will end — including title probabilities and relegation risk for every team.

All logic runs entirely in the browser. Open the HTML file and everything works offline.

---

## ✨ Features

- **Live Standings Table** — points, W/D/L, goal difference, and recent form dots for all 10 teams
- **Monte Carlo Season Predictor** — 1,000 full-season simulations aggregated into stable, meaningful statistics
- **Title Probability %** — how often each team wins the league across all simulations
- **Relegation Probability %** — how often each team finishes 9th or 10th (the relegation zone)
- **Top 3 Probability %** — likelihood of a podium finish
- **Points Range** — 10th–90th percentile spread showing prediction certainty vs. uncertainty
- **Most Likely Position** — the rank a team lands in most frequently, with confidence %
- **Full Results Archive** — every played match organized by week, for both Fall 2025 and Spring 2026
- **Remaining Fixtures** — all 20 unplayed Spring 2026 matches (Weeks 10–14)
- **Form & Stats Cards** — per-team breakdown with goals scored/conceded per game, win rate, and pts per game
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

## 🔮 How the Simulation Works

The predictor uses a **Monte Carlo method** to simulate the remaining 20 fixtures 1,000 times and aggregate the results.

### Algorithm

For each unplayed match, the outcome is determined probabilistically:

```
Home win probability  = (home PPG + 0.2) / (home PPG + 0.2 + away PPG) × 0.60
Draw probability      = 0.22
Away win probability  = 1 - home win probability - draw probability
```

Where:
- **PPG** = current points per game for each team (based on all played results)
- **+0.2** = home advantage factor
- The 0.60 and 0.22 constants weight outcomes toward decisive results, consistent with Sunday league scoring patterns

### Aggregation

After 1,000 runs, for each team the following are recorded:
- Final points in every run → used to compute **median**, **min**, **max**, **p10**, **p90**
- Final rank in every run → used to compute **title %**, **top 3 %**, **relegation %**, and **most likely position**

This means results are **stable across page loads** — not a single random snapshot.

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

All match data lives in two JavaScript arrays near the top of `league-predictor.html`:

```javascript
// Fall 2025 results
const FALL = [
  {w:1, wl:'Week 1 — Sep 7', h:'ATX United - Red', a:'TBD FC', hg:4, ag:2},
  // ...
];

// Spring 2026 results (played)
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

Save the file and refresh — standings and predictions update automatically.

---

## 📊 Dashboard Tabs

| Tab | Description |
|---|---|
| **Current Standings** | Live points table with form indicators |
| **Season Prediction** | Aggregated simulation results — title %, relegation %, points ranges |
| **Results** | Full match history for Fall 2025 and Spring 2026 |
| **Remaining Fixtures** | All 20 unplayed Spring 2026 matches |
| **Form & Stats** | Per-team stats: goals/game, win rate, pts/game, form streak |

---

## 🗂️ Project Structure

```
league-predictor.html   # Everything — HTML, CSS, JS, and data in one file
README.md               # This file
```

The entire project is intentionally a **single self-contained file** for maximum portability. No build step, no npm, no bundler.

---

## 🛠️ Technical Details

- **Language:** Vanilla JavaScript (ES6+)
- **Styling:** CSS custom properties, CSS Grid, CSS animations
- **Fonts:** Bebas Neue (display) + DM Mono (body) via Google Fonts
- **Simulation:** Monte Carlo, 1,000 iterations, runs synchronously on page load
- **No external dependencies** beyond Google Fonts (fully functional offline if fonts are cached)
- **Browser support:** All modern browsers (Chrome, Firefox, Safari, Edge)

---

## 📈 Interpreting Results

| Metric | What it means |
|---|---|
| **Median pts** | The "expected" final points — half of simulations landed above, half below |
| **Pts range (p10–p90)** | Narrow range = predictable outcome; wide range = volatile/uncertain |
| **Title %** | 40%+ means strong favorite; under 10% means long shot |
| **Relegation %** | Over 50% = in serious danger; over 80% = almost certain |
| **Most likely pos** | The single most common finishing rank across all 1,000 runs |

---

## 🔧 Customization

### Change number of simulation runs

```javascript
const RUNS = 1000;  // increase for more accuracy, decrease for speed
```

### Adjust home advantage

```javascript
const hs = hp + 0.2;  // change 0.2 to tune home advantage strength
```

### Change relegation zone

The relegation zone is defined as finishing 9th or 10th (`i >= 8` in zero-indexed rank):

```javascript
if(i >= 8) agg[t.team].relCount++;  // change 8 to adjust zone size
```

---

## 📝 License

MIT — free to use, modify, and share.

---

*Built for AMSA Sunday League · Austin, TX · 2025–2026 Season*
