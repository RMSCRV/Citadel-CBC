# Policy recommendation, rate path and market implications

This layer takes the regime, scenario, expectations and loss-function layers and turns them into one Bank Rate call, a probability distribution for Bank Rate at every MPC meeting from April 2026 to March 2027, and a comparison of that distribution with market pricing. It draws on BoE Macro Technical Paper No.4 (Alati et al. 2025) loss function; BoE Macro Technical Paper No.1 smoothed rule; Taylor (1993); Svensson (2003); Brainard (1967); Hansen & Sargent (2008).

## Question it answers
What should the MPC do in April 2026? How likely is each Bank Rate level after each meeting through March 2027? Where does our path differ from what the market prices?

## Specification
We choose $`i`$ to minimise expected loss over a grid of oil and wage states $`s`$:

```math
i^\star=\arg\min_i \sum_s p_s\,\mathbb{E}\big[L(i\mid s)\big],\qquad L=\sum_t \beta^t\big[(\pi_t-\pi^\ast)^2+\lambda\,y_t^2+\mu\,(\Delta i_t)^2\big]
```

The weights come from the BoE objective and we leave them alone. The smoothed rule $`i_t=\rho i_{t-1}+(1-\rho)[r^\ast+\pi^\ast+\phi_\pi(\pi_t-\pi^\ast)+\phi_y y_t]`$, with $`\rho=0.91`$, $`\phi_\pi=1.357`$, $`\phi_y=1.072`$, is a cross-check. It does not set the rate.

For each meeting $`m`$ we put probabilities $`p_{m,k}`$ on Bank Rate levels $`k`$ in 25bp steps and report

```math
\mathbb{E}[i_m]=\sum_k p_{m,k}\,k,\qquad \sigma_m=\Big(\sum_k p_{m,k}\,(k-\mathbb{E}[i_m])^2\Big)^{1/2}
```

with cumulative tightening measured against the current 3.75%.

## Data
SONIA (BoE IUQASOIA) and meeting-dated OIS; RPI swaps at 1y and 5y5y; the gilt curve (BoE IADB); oil-options scenario weights; the regime vector from the HMM, Kalman and NLP layers; mortgage refix volumes and quoted fixed rates.

## Key result
**HOLD at 3.75% in April, with a conditional tightening bias. June is live and is our base case for the first hike.**

Probability (%) of each Bank Rate level after each meeting (figure `policy_path.png`):

| Level | Apr 26 | Jun 26 | Jul 26 | Sep 26 | Nov 26 | Dec 26 | Feb 27 | Mar 27 |
|---|---|---|---|---|---|---|---|---|
| 5.25% | | | | | | | 2 | 3 |
| 5.00% | | | | | 1 | 3 | 6 | 7 |
| 4.75% | | | | 3 | 7 | 12 | 14 | 15 |
| 4.50% | | | 6 | 14 | 22 | 27 | 25 | 23 |
| 4.25% | | 15 | 27 | 33 | 34 | 31 | 25 | 22 |
| 4.00% | 20 | 50 | 45 | 38 | 26 | 19 | 15 | 14 |
| 3.75% | 80 | 35 | 22 | 12 | 8 | 6 | 7 | 8 |
| 3.50% | | | | | 2 | 2 | 4 | 5 |
| 3.25% | | | | | | | 2 | 3 |
| **Mean** | 3.80 | 3.95 | 4.04 | 4.14 | 4.23 | 4.31 | 4.33 | 4.32 |
| **Std dev (bp)** | 10 | 17 | 21 | 24 | 29 | 31 | 41 | 45 |
| **vs 3.75% (bp)** | +5 | +20 | +29 | +40 | +48 | +56 | +58 | +57 |

April is an 80% hold with a 20% hike tail. In June half the mass sits at 4.00%. Most of the tightening comes in June, July and September, and expected tightening reaches about 56bp by December. After that the mean barely moves while the spread keeps growing: the standard deviation goes from 10bp in April to 45bp by March 2027. In 2027 the weight runs both ways, with cuts to 3.25–3.50% and hikes to 5.00–5.25% both in play.

Why we hold in April:
- Mortgages are already doing some of the tightening. Bank Rate sat at 0.1% until December 2021, and many fixed deals were struck that year at very low rates. In 2026, 1.8m deals expire and refix at an average 5y fixed rate near 5.75%. Each refix lifts monthly payments and cuts disposable income without any new move in Bank Rate, and most of that is still to come. 10y gilts are also up 50bp.
- Bank Rate is a slow tool against an imported energy shock. The energy shock reaches CPI quickly; policy disinflation arrives with a long lag.
- The costs are lopsided. A premature hike piles a real-income squeeze on top of the energy hit and is costly to unwind. A hike delayed by one meeting can be made up while 5y5y stays put.
- Wage persistence is not in the data yet. AWE is 3.8% and falling.

We took the hike case seriously. Services CPI is 4.5% and the regime read is 66% stagflation. Even so, the loss minimum stays at HOLD under current conditions, and it flips to HIKE only with oil at about \$120 or more, or wage growth above 4.5–5%. The Taylor variants mostly agree: smoothed 3.81% (+6bp), regime-adjusted 3.86% (+11bp), \$125 oil 3.97% (+22bp). Only the unsmoothed rule, at 4.40%, asks for a move now.

At submission the market priced about 56bp of tightening by end-2026 (roughly 2¼ hikes) and three hikes to 4.50% by the 18 March 2027 meeting. We get the same 2026 total, but we put it earlier, in June, July and September. By March 2027 our mean is 4.32%, below the 4.50% priced, and our distribution has fatter tails on both sides.

**Trade expression (entry logic and invalidation only, no P&L):**
- Pay June and July 2026 meeting-dated OIS. The bet is on timing: we expect the hikes sooner than the curve does. A June hold alongside oil easing toward the \$78 peace-deal path would kill it.
- Receive the March 2027 meeting. We think the third hike is less certain than priced. Oil heading toward the \$125 escalation path, or wage growth breaking above 5%, would invalidate it.
- Use options for the 2027 distribution. With a 41–45bp standard deviation and weight at both 3.25% and 5.25%, owning convexity fits our view better than a straight directional position.

Catalysts to watch are the June MPC decision, vote split and guidance; services CPI against the 4.5% print; AWE and regular pay, where a move above 4.5–5% flips the decision; Brent relative to the \$95 and \$125 scenario peaks; 5y5y holding near 2.21%; and the pace of 2026 mortgage refixes.

## Why this method
Minimising loss over a state grid makes the reaction function something we derive from the BoE's own objective, so we don't have to assume it. Putting probabilities on every meeting shows when we expect the moves and how unsure we are, and both matter for meeting-dated trades. A single modal path hides both. Brainard's argument favours holding when transmission is uncertain. Robust control (Hansen & Sargent) is why the tightening bias is stated openly.

## Limits
The meeting probabilities are a judgement-weighted mapping from scenarios to rate paths. They are not estimated from market prices, and they inherit the oil-options weights, which we think under-price extreme outcomes. Rates move in 25bp steps. The trades are conditional framings and have not been backtested. Market pricing is a snapshot at submission and changes daily.
