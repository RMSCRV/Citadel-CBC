# Local Projections of CPI and Output Responses

This memo gives a direct, horizon-by-horizon estimate of how UK CPI inflation responds to an oil shock measured in US dollar Brent. It is a cross-check on the recursive VAR benchmark, and it supplies the response function that turns Brent scenario paths into CPI inflation projections.
Sources: Jordà (2005, AER); Plagborg-Møller & Wolf (2021, Econometrica); Montiel Olea & Plagborg-Møller (2021, Econometrica); Li, Plagborg-Møller & Wolf (2024, J. Econometrics); Kilian (2009, AER) for the oil-shock framing.

## Question it answers
If Brent in dollars rises 10% this month, how much higher is UK CPI inflation $`h`$ months later, for $`h = 0,\dots,24`$? We want an answer that does not rely on a VAR getting its dynamics right two years out, at a time when the regime layer puts most of its weight on stagflation.

## Specification
For each horizon $`h`$ we run a separate OLS regression:

```math
\pi_{t+h} - \pi_{t-1} \;=\; \alpha_h + \beta_h\, s_t + \sum_{\ell=1}^{12} \gamma_{h,\ell}' \, w_{t-\ell} + u_{t+h}, \qquad h = 0,\dots,24
```

- $`\pi_t`$: CPI inflation in pp, built from the monthly CPI series (D7BT).
- Seasonality: ONS CPI is not seasonally adjusted, so the monthly CPI and CPI energy changes have their month-of-year means removed before estimation. Otherwise the regular seasonal pattern in monthly prices would leak into the lag terms.
- $`s_t`$: the shock, Brent in USD (EIA RBRTE) % m/m, as 100 × log difference. No contemporaneous controls enter, so $`\beta_h`$ is the response to the part of $`s_t`$ the 12-lag information set does not predict. That is the same timing assumption as a recursive VAR with oil ordered first.
- $`w_t = (s_t,\ \Delta \log \text{CPI energy}_t,\ \Delta \log \text{CPI}_t,\ i_t)'`$: 12 lags of all four variables, with $`i_t`$ the Bank Rate level.
- The impulse response is $`\{\beta_h\}_{h=0}^{24}`$, scaled ×10 to read per 10% shock.
- Inference: Newey-West HAC standard errors with bandwidth $`h+1`$, for the MA($`h`$) overlap in $`u_{t+h}`$. The figure shows ±1 s.e. (68%) bands.
- Extension, not estimated here: state dependence can enter as an interaction, $`\beta_h s_t + \delta_h\, s_t \cdot p^{\text{stag}}_{t}`$, where $`p^{\text{stag}}_t`$ is the HMM stagflation probability. Each horizon stays linear in parameters, so no regime-switching VAR is needed.

For scenario projections, let $`e_\tau`$ be the % m/m innovations implied by a Brent path. The CPI projection is the VAR baseline plus the LP responses:

```math
\hat\pi^{\,\text{scen}}_{T+k} = \hat\pi^{\,\text{VAR}}_{T+k} + \sum_{\tau = 1}^{k} \tfrac{\beta_{\min(k-\tau,\,24)}}{10}\, e_{T+\tau}.
```

Dividing the per-10% response by 10 gives the response per 1% innovation; beyond $`h=24`$ the response is held at $`\beta_{24}`$. Bands come from 2,000 residual-bootstrap VAR paths and reflect VAR baseline forecast error only.

## Data
Monthly, 1997-01 to 2026-02. EIA RBRTE (Brent, USD), ONS D7BT (CPI all items) and D7CH (CPI electricity, gas and other fuels), and BoE Bank Rate.

## Key result
- A 10% Brent shock raises CPI inflation by 0.37pp at the peak (month 11) and 0.36pp at month 12. The recursive VAR benchmark on the same data (oil, energy CPI, CPI, Bank Rate) gives 0.35pp at month 11 and 0.33pp at month 12. Each IRF sits inside the other's 68% band (`figures/lp_vs_svar_irf.png`).
- The deck's rule of thumb is about 40bp of CPI inflation per sustained \$10/bbl. That is the same order of magnitude as the LP estimate, though not the same number: one is in dollar levels, the other in percentage moves, and the samples differ.
- Scenario projections, VAR baseline plus the LP response to each Brent path (68% band about ±1.0pp in 2026Q4; `figures/cpi_projection.png`):

| Scenario | Brent peak | CPI 2026Q3 | CPI 2026Q4 | Peak | CPI 2027Q4 |
|---|---|---|---|---|---|
| Peace deal | \$78 | 2.7% | 2.9% | 3.1% | 2.6% |
| Normalisation | \$95 | 3.1% | 3.4% | 3.5% | 2.5% |
| Status quo | \$100 | 3.3% | 3.7% | 3.8% | 2.6% |
| Escalation | \$125 | 3.8% | 4.4% | 4.8% | 3.2% |

## Why this method
- A VAR extrapolates its 12-lag dynamics out to $`h=24`$. An LP estimates each horizon directly, so errors in the lag structure do not compound. That matters most when the regime is shifting.
- In population, LPs and VARs estimate the same impulse responses (Plagborg-Møller & Wolf, 2021). In finite samples LPs trade lower bias for higher variance (Li, Plagborg-Møller & Wolf, 2024). The two estimates here are close, which we read as support for both.
- Including lags of the regressors gives valid inference at long horizons and under high persistence (Montiel Olea & Plagborg-Møller, 2021). The 12 lags of all four variables, the shock included, do that here.
- State dependence costs one interaction term rather than a full nonlinear system.

## Limits
- The shock is a recursive Brent innovation, not an external instrument, so oil supply and global demand shocks are pooled (Kilian, 2009). An LP with an external instrument is the obvious next step.
- The sample ends before the March 2026 Brent jump. A plain monthly VAR nowcast of March 2026 CPI gives 3.07% against a 3.3% outturn, which is why the nowcast itself runs on MIDAS with daily data.
- Bands are wide at long horizons, and the scenario bands carry VAR baseline error only. Adding LP sampling error would widen them.
- The projection is linear. It picks up second-round wage effects only to the extent the sample history contains them.
