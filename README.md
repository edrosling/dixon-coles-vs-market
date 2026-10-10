# Can a Dixon-Coles model beat the Premier League betting market?

A walk-forward test of a goals-based forecasting model against bookmaker closing odds, 2020/21 onwards.

[![Open in Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/edrosling/dixon-coles-vs-market/blob/main/dixon_coles_vs_market.ipynb)
&nbsp;**[Live dashboard](https://edrosling.github.io/dixon-coles-vs-market/)**

## Summary

**Question.** Using only public match results, does a Dixon-Coles model add anything to the forecasts embedded in bookmaker closing odds?

**Answer.** On this evidence, no. The model captures about 80% of the market's forecasting skill (8.1% against 10.2% log-loss improvement over a base-rate forecaster) but trails it in every one of six seasons, on two independent benchmarks.

| Benchmark | Matches | Model log loss | Market log loss | Gap (model minus market) | 95% interval | Seasons worse |
|---|---|---|---|---|---|---|
| Pinnacle closing odds | 2,110 | 0.9767 | 0.9557 | **+0.0210** | +0.0133 to +0.0288 | 6 of 6 |
| Market-average closing odds | 2,280 | 0.9839 | 0.9622 | **+0.0217** | +0.0144 to +0.0289 | 6 of 6 |

Lower log loss is better, so a positive gap means the model is worse. A season-level test (six seasons) gives t = 4.8 on Pinnacle odds and t = 6.4 on average odds, so the result does not rest on matches being independent within a season.

## Why I built this

This grew out of my 2024 Extended Project Qualification on whether big business in football is unfair. It was a qualitative essay, but it discussed data-driven clubs such as Brentford and cited the Dixon and Coles (1997) paper that much of this literature builds on. This project is where I implemented that model properly and tested it against the market.

## Data

* **Source:** [football-data.co.uk](https://www.football-data.co.uk/englandm.php), Premier League seasons 2014/15 to 2026/27: results, bookmaker odds (Pinnacle earlier and closing prices, and market-average closing prices) and shot counts.
* **Benchmarks:** closing odds with the bookmaker margin removed (Shin method). Pinnacle closing odds cover every test season until part-way through 2025/26, where the series stops. Market-average closing odds cover every match, so they are the second benchmark and the only one for the current season.
* The raw files are downloaded by the notebook. `pl_matches.csv` is a derived copy included for convenience; see the source site for its terms.

## Method

* **Model:** Dixon-Coles attack and defence strengths, a home-advantage term, a low-score correction and a separate home advantage for the crowd-free period (2020/21). Strengths are fitted by ridge-penalised weighted maximum likelihood with exponential time decay (half-life 365 days) on a rolling four-year window.
* **Walk-forward:** every match is predicted using only earlier results, with the model refitted every 7 days.
* **Promoted clubs:** shrunk harder towards the average first-season profile of past promoted teams, estimated only from earlier seasons.
* **No look-ahead in tuning:** half-life and shrinkage were chosen on 2017/18 to 2019/20 only. The test seasons, 2020/21 onwards, were never used for tuning. The current season is a separate forward test.
* **Metrics:** log loss (main), Brier score and ranked probability score (RPS).

## Results

### The gap is consistent but modest

| Season | Matches | Model | Market | Gap | SE |
|---|---|---|---|---|---|
| 2020/21 | 380 | 1.0152 | 0.9979 | +0.0174 | 0.0106 |
| 2021/22 | 380 | 0.9517 | 0.9363 | +0.0155 | 0.0090 |
| 2022/23 | 380 | 0.9991 | 0.9620 | +0.0371 | 0.0094 |
| 2023/24 | 380 | 0.9241 | 0.8984 | +0.0257 | 0.0088 |
| 2024/25 | 380 | 0.9836 | 0.9666 | +0.0171 | 0.0091 |
| 2025/26 | 210 | 0.9945 | 0.9869 | +0.0076 | 0.0113 |

Pinnacle closing odds. No single season is decisive; the evidence is that all six have the same sign.

### How much of the market's skill does the model have?

| Forecaster | Log loss | Brier | RPS | Skill vs base rate |
|---|---|---|---|---|
| Base rate (past results only) | 1.0709 | 0.6485 | 0.2342 | 0.0% |
| Dixon-Coles v1 | 0.9857 | 0.5868 | 0.2040 | 8.0% |
| Dixon-Coles v2 (weekly refits, final) | 0.9839 | 0.5857 | 0.2034 | 8.1% |
| Market (average closing odds) | 0.9622 | 0.5711 | 0.1964 | 10.2% |

The RPS gap is +0.0070 (SE 0.0012), so the ranking holds on all three metrics. Both the model and the market are well calibrated; the market is sharper, putting more probability on its favourites and still being right.

### Why does the market win?

Tests of explanations, each reported whether or not it helped:

* **Not mainly late news.** The model trails even the earlier Pinnacle price (+0.0163, SE 0.0036). Only about 23% of the total gap arrives between that price and the close (+0.0047, SE 0.0016). The market already had a better read of team quality earlier. ("Earlier" means when the data source recorded the price, not necessarily the opening line.)
* **No complementary information.** The best blend of model and market puts weight 0.00 on the model. Log loss rises by +0.0006 at a 10% model weight and +0.0062 at 50%.
* **Simple shot counts do not explain it.** A second set of strengths fitted on shot-based pseudo-goals was tested on validation seasons: log loss rose from 0.9436 (goals only) to 0.9551 (shots only), so no blend was adopted. This uses shot counts, not location-based xG.
* **Promoted clubs help a little.** Targeted shrinkage cut the gap in matches involving promoted clubs from +0.0208 to +0.0161 (648 matches) and left other matches essentially unchanged (+0.0246 to +0.0239, 1,632 matches). The model is still behind the market in both groups.
* **Fresher refits help a little.** Moving from 14-day to weekly refits reduced the gap from +0.0228 to +0.0217. Together, the two changes closed about 8% of the original gap (+0.0236).

### Economic significance check

Flat 1-unit stakes at Pinnacle closing odds on the outcome with the largest model edge gave an ROI of **-3.6%** (95% interval -10.9% to +3.9%, 2,025 bets). That sits inside the range from picking a random outcome in every match (-10.7% to +2.6%) and close to the average bookmaker margin of 2.7%. There is no evidence of a tradeable edge.

## Limitations

* The model uses goals only: no lineups, injuries, squad values or shot location.
* Confidence intervals treat matches as independent, which is somewhat optimistic; the season-level test uses only six seasons.
* Promoted-club ratings rest on few matches and a prior, and clubs returning after time in the Championship carry stale ratings from their last top-flight season.
* The betting simulation assumes closing prices are obtainable, and bets at each threshold are nested.
* The current-season forward test is small, so it is an update log rather than evidence.

## What I would do next

* Location-based xG and squad-value data, to test whether richer information closes the gap.
* Second-division results to rate promoted clubs better.
* Block-bootstrap intervals and an Elo baseline.
* Link the model's rating spread to the wage and competitive-balance questions from my EPQ.

## Reproduce

1. Open the notebook in Colab (badge above) and use **Runtime, then Run all**. A full run takes roughly 10 to 20 minutes, depending on Colab's speed.
2. The first cell optionally mounts Google Drive so files survive session resets. Set `USE_DRIVE = False` to skip it.
3. Optional switches in section 1: `REFRESH_DATA` (re-download after new matches), `RUN_TUNING` (re-derive the settings), `RUN_SHOTS` (re-run the shot experiment), `FAST_DEMO` (smoke test).
4. The last cells write `index.html`, the dashboard, which works offline and on GitHub Pages.

**Requirements:** Python 3 with pandas, numpy, scipy and plotly, all preinstalled in Colab.

## Repository contents

| File | Purpose |
|---|---|
| `dixon_coles_vs_market.ipynb` | The full analysis, top to bottom |
| `index.html` | Interactive dashboard (also served by GitHub Pages) |
| `dashboard_data.json` | All numbers the dashboard displays |
| `predictions.csv` | Match-by-match forecasts for v1, v2 and weekly v2 |
| `pl_matches.csv` | Cleaned match, odds and shot data |

## References

* Dixon, M. J. and Coles, S. G. (1997). *Modelling association football scores and inefficiencies in the football betting market.* Journal of the Royal Statistical Society, Series C, 46(2).
* Shin, H. S. (1993). *Measuring the incidence of insider trading in a market for state-contingent claims.* The Economic Journal.

## Licence

Code: MIT licence (see `LICENSE`). Data: football-data.co.uk, used for research; see the source site for its terms.

## Disclaimer

Educational research, not betting or investment advice. This project is not affiliated with or endorsed by the Premier League or any club.

*Built by Ed, BSc Economics, University of Bristol.*
