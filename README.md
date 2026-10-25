# CFB Résumé Re-ranker

A single-page tool that rebuilds the AP Top 25 from game results, lets you set your own weights, works out how the AP itself is weighting teams, and flags where the AP contradicts itself.

Open `index.html` in any browser. There is nothing to install and no server. Everything runs in the page.

Data covers the 2026 season through Oct 24, compared against the Oct 25 AP poll. It tracks 37 teams: every team ranked in any AP poll this season, including the 12 that have since dropped out. Teams outside the top 25 are listed below it on the Rankings tab and marked NR.

---

## Setting custom weights

All weighting happens on the **Rankings** tab with four sliders.

| Slider | What it measures | Higher setting rewards |
|---|---|---|
| **H2H résumé** | Who a team beat and lost to, scaled by how good each opponent was and where the game was played | Quality wins, close losses to good teams |
| **Offense** | Points scored, counted up against good opponents and down against weak ones | Scoring on strong defenses |
| **Defense** | Points allowed, counted up against weak opponents and down against strong ones | Holding good offenses down |
| **Preseason reputation** | The team's preseason AP points. This is not a result | Teams voters rated highly in August |

### The sliders are relative, not percentages

The four values don't need to add to 100. The page divides each one by the total to get its share. For example:

| Sliders (H2H / Off / Def / Pre) | Actual share |
|---|---|
| 60 / 20 / 20 / 0 | 60% / 20% / 20% / 0% |
| 60 / 60 / 0 / 0 | 50% / 50% / 0% / 0% |
| 100 / 50 / 50 / 0 | 50% / 25% / 25% / 0% |
| 10 / 10 / 10 / 10 | 25% each |

The colored bar under the sliders always shows the real share, and the formula line shows the exact math being applied.

### Where to start

| If you believe... | Try |
|---|---|
| Rankings should be about who you beat | 80 / 10 / 10 / 0 |
| The best team, not the best résumé, should be No. 1 | 30 / 35 / 35 / 0 |
| Defense wins championships | 40 / 10 / 50 / 0 |
| You want to see what the AP is doing | Press **Match the AP** |
| You want the default | Press **Reset** (60 / 20 / 20 / 0, recency 0) |

Keep Preseason at 0 if you want a ranking built only on results. Raise it to see how much reputation it takes to reproduce the AP's order.

### Recency

The Recency slider, under the four weights, controls how much recent games count compared with early ones. It changes how each team's H2H, offense and defense averages are built, not the shares between them.

| Recency | Last week's game | A week 1 game |
|---|---|---|
| 0 | 100% | 100% (every game equal) |
| 25 | 100% | 80% |
| 50 | 100% | 60% |
| 75 | 100% | 40% |
| 100 | 100% | 20% |

Games in between are scaled by date. Use it if you think a team's last few weeks say more than its opener did.

### Sharing your settings

Press **Copy link to these settings** at the bottom of the Weights card. The link carries the four weights, recency, the tiebreak setting, any conference values you changed, and the tab you're on. Anyone who opens it sees your exact ranking. The page address also updates as you change settings, so copying it from the browser bar works too.

Link format, for reference: `#w=60-20-20-0&r=50&tb=0&c=B12:0.9,ACC:0.9&fcs=0.2&tab=ana`. Only settings that differ from the defaults are included.

### Reading what happened after a move

- **What the last move did** lists the biggest risers and fallers after each slider change and names the stat that drove each one.
- **Agreement with the AP poll** shows three correlation scores between your ranking and the AP. 1.00 means identical order, 0 means unrelated.
- Each team row breaks its score into colored segments, one per factor. A segment to the right of the center line helps the team and one to the left hurts it.

### The head-to-head tiebreak

When the checkbox is on, a team that beat another ranked team is placed directly above it, as long as the loser's score is less than 6 points higher. Rows moved this way are tagged "beat [team]", which is why the score column can be out of order. Turn it off for a pure score ranking.

---

## Setting conference weights

The **Conferences** tab controls how much an opponent's record is worth based on its conference. Every tab recalculates when you move a slider here.

Each opponent's win percentage is multiplied by its conference value to get q, the opponent strength score.

| Value | Meaning |
|---|---|
| 1.00 | Take the record at face value |
| 0.70 | A 6-1 record counts about like a 4-3 record in a 1.00 conference (the default for non-Power conferences) |
| Above 1.00 | That conference's records are harder to earn than face value |
| Below 0.50 | Records there barely count |

Each conference has its own slider, with the number of games the tracked teams played against it. Conferences no tracked team has played are hidden. A separate slider sets a fixed q for every FCS opponent, regardless of record.

### Presets

| Preset | Power 4 and Notre Dame | Other FBS | FCS |
|---|---|---|---|
| Default | 1.00 | 0.70 | 0.10 |
| No adjustment | 1.00 | 1.00 | 0.25 |
| Tiered Power 4 | SEC, Big Ten, Notre Dame 1.00. Big 12, ACC 0.90 | American 0.70. Mountain West, Pac-12, Sun Belt 0.65. MAC, Conference USA 0.60 | 0.10 |
| Harsh on G5 | 1.00 | American 0.55. Mountain West, Pac-12, Sun Belt 0.50. MAC, Conference USA 0.45 | 0.05 |

**Who moves** lists the teams whose rank changes compared with the default values at your current Rankings sliders. **Schedule strength** shows each team's average opponent q now and at the defaults. Teams that played many non-Power opponents move the most.

---

## How the score is built

Every team ends up with a score centered on 50. The steps are:

**1. Opponent strength (q).** Each opponent gets a value, usually from 0 to 1. It is the opponent's win percentage with the game against this team removed, multiplied by the opponent's conference value from the Conferences tab. FCS opponents get the fixed FCS value.

**2. Per-game values.**
- H2H: a win is worth `1 + q`, plus 0.10 on the road or 0.05 at a neutral site. A loss is worth `q − 0.5`, minus 0.10 at home, plus 0.10 if it was within 7 points.
- Offense: `min(points, 56) × (0.6 + 0.8q)`
- Defense: `points allowed × (1.4 − 0.8q)`, where lower is better

Each is averaged across the team's games.

**3. z-scores.** Each stat is converted to standard deviations from the average of all 37 tracked teams, so 0 is average, +1 is clearly better and −1 is clearly worse. Defense is flipped so positive is always better.

**4. Weights.** `Score = 50 + 15 × (H2H share × zH + Off share × zO + Def share × zD + Pre share × zPre)`

The **Stats** tab walks through steps 1 to 4 for any team you pick, using that team's real games and numbers.

---

## Changing the defaults in the code

Everything lives in `index.html`. Search for the text in the left column to find the line.

| To change | Search for | Notes |
|---|---|---|
| Slider starting values | `id="wh" min="0" max="100" step="1" value="60"` (and `wo`, `wd`, `wp`) | Set each `value` |
| Reset button values | `setW(60,20,20,0,0)` | The fifth number is recency |
| Recency curve | `return 1-(rec/100)*0.8*age` | The 0.8 sets how low the oldest game can go at recency 100 (0.8 means 20%) |
| Default conference values | `const CONF_INFO=[` | Each entry is `[code, name, default value, P or G]`. The third number is what the Default preset and the "Who moves" comparison use |
| Default FCS value | `const FCS_DEF=0.10` | |
| Conference presets | `const PRESETS={` | Add a preset here and add a matching button next to `data-p="harsh"` |
| Win and loss values | `let v=w?1+q:q-0.5;` | The road, neutral, home-loss and close-loss adjustments are on the next line |
| Points cap for offense | `Math.min(pf,56)` | Stops blowouts of weak teams from inflating offense |
| Defense scaling | `pa*(1.4-0.8*q)` | |
| Tiebreak margin | `S[lo].s-S[name].s<6` | Raise it to apply head-to-head more often |

Save the file and refresh the browser to see changes.

---

## Updating for a new week

Four data blocks need updating:

1. **`const DATA=`** holds every tracked team's games. Each team is `[name, conference, wins, losses, [games]]` and each game is `[opponent, site H/A/N, points for, points against, opponent win % without this game, opponent conference code, opponent wins, opponent losses, date]`. Conference codes: SEC, B1G, B12, ACC, IND (Notre Dame), AAC, MWC, PAC, SBC, MAC, CUSA, FBSI, FCS.
2. **`const POLLS=`** holds every AP poll by date as `[team, rank, points]`. Add the new week's poll as a new date key.
3. **`const ORV=`** holds the "others receiving votes" points for each poll by date as `[team, points]`. These give unranked teams a real AP value instead of zero.
4. **`const AP=`** holds the current week's AP rank for each ranked team. Tracked teams missing from it are treated as unranked.

When a team enters the poll for the first time, add it to `DATA`. Tracked teams stay in even after they drop out. Everything on the Analysis tab updates automatically from these four blocks.

---

## The Analysis tab

The tab is split into five sections, picked from the bar at the top. The section you're on is saved in share links.

**Insights.** Four headline numbers: how well the AP's implied model fits, how much weight reputation carries, how much credit the AP gives a Group of 5 opponent, and the count of contradictions. Below them, **What stands out** shows up to six findings as cards, each tagged by type (inconsistency, ignored result, double standard, reputation, conference, contrarian) and backed by the specific games and polls. Findings only appear when the data clears a threshold, and they're ranked by strength of evidence.

**How the AP ranks.** The AP's implied weights: for each poll, every slider mix in 5% steps is tested at five recency settings and 33 conference settings, and the best fit to that week's AP points is kept. That gives the AP's weight on each factor, its recency bias and its conference discount, week by week. **Match the AP** on the Rankings tab loads all of it into your sliders and the Conferences tab. Also here: agreement scores between your ranking and the AP, and a scatter of résumé score against AP points.

**Contradictions.**
- **Head-to-head inversions:** teams ranked behind a team they beat, how many straight polls it has stood, and whether the record explains it.
- **More losses, ranked ahead:** pairs where the team with more losses sits higher, split by whether the H2H résumé justifies it.
- **How losses were punished:** every loss by a ranked team, the actual drop against the drop a regression on opponent strength, margin and starting rank predicts, and a test for whether preseason favorites get protected.
- **Fell after a win:** ranked teams that dropped 2 or more spots in a week they won.

**Season trends.** AP rank history for every tracked team, poll inertia (week-to-week correlation, AR(1) persistence, preseason decay) and preseason anchoring.

**Stats lab.** Correlation matrix, preseason-ranked vs unranked test (Fisher's exact, used in place of chi-square because several cells have fewer than 5 teams), weight sensitivity across 1,330 slider settings, and method notes.

## Data sources

Game scores come from the SportRadar feed, with gaps filled from theScore, Wikipedia and team athletics sites. AP polls come from LSU Athletics' weekly rankings posts, NCAA.com, FootballScoop and FBSchedules. Point totals for the Sep 8 to Sep 27 polls each rest on a single source. Others-receiving-votes points come from LSU Athletics (Sep 8 to Oct 18), NCAA.com (Oct 25) and FBSchedules (preseason). Every game between two tracked teams was checked against both teams' schedules.

## Hosting

The page is one self-contained file. On GitHub Pages: Settings → Pages → Deploy from a branch → `main` / root. It is served at `https://<username>.github.io/<repo>/`.
