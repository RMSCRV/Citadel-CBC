# MIDAS nowcast of GDP and CPI

This layer sets the starting point for the structural layers: current-quarter GDP and the latest monthly CPI inflation print. It uses every observation released up to the forecast date. It follows Ghysels, Santa-Clara & Valkanov (2004) MIDAS and Foroni, Marcellino & Schumacher (2015) U-MIDAS (JRSS-A).

## Question it answers

Where are UK activity and inflation *now*, before the ONS publishes them? Quarterly GDP and monthly CPI arrive with a lag. Energy prices move every day, and in March 2026 they moved sharply.

## Specification

Let $`y_t`$ be the low-frequency target: quarterly real GDP growth, or monthly CPI inflation. Let $`x^{(m)}_{t-j/m}`$ be a regressor sampled $`m`$ times per low-frequency period (daily within the month, monthly within the quarter). The MIDAS regression is

```math
y_{t+h} = \beta_0 + \sum_{i=1}^{N} \beta_i \sum_{j=0}^{K_i-1} w_i(j;\theta_i)\, x^{(m_i)}_{i,\,t-j/m_i} + \sum_{p=1}^{P} \lambda_p\, y_{t-p} + \varepsilon_{t+h},
```

with the exponential Almon lag polynomial

```math
w(j;\theta_1,\theta_2) = \frac{\exp(\theta_1 j + \theta_2 j^2)}{\sum_{k=0}^{K-1}\exp(\theta_1 k + \theta_2 k^2)} .
```

Each high-frequency block costs two shape parameters plus one scale $`\beta_i`$, whatever the lag length $`K_i`$. The weights sum to one, so $`\beta_i`$ gives the total effect of block $`i`$. We estimate $`(\beta,\theta,\lambda)`$ by nonlinear least squares. For blocks with a small frequency ratio (monthly into quarterly), we use the U-MIDAS variant, where each lag gets its own coefficient and OLS estimates them:

```math
y_{t+h} = \beta_0 + \sum_{j=0}^{K-1} \gamma_j\, x^{(m)}_{t-j/m} + \varepsilon_{t+h}.
```

Identification comes from timing alone. These are forecasting regressions and carry no structural interpretation.

Series end on different dates (the ragged edge). On a given nowcast date, each indicator's lag window starts at its own last release. We set the horizon $`h`$ per vintage and re-estimate the same equation for each information set. Missing within-period high-frequency observations are not imputed; the lag window for those series just shifts back.

## Data

- Targets: ONS ABMI (real GDP), D7BT (CPI all items), with D7CH (electricity, gas and other fuels) and D7EC (fuels and lubricants) as components.
- High-frequency (daily): EIA RBRTE Brent converted to sterling (x USDGBP); NBP gas; road-fuel pump prices.
- Monthly: ONS activity, labour-market and price indicators, plus BoE IADB rates (IUQASOIA SONIA).

## Key result

- Q1 2026 GDP nowcast: **+0.15% q/q**, after Q4 2025 at +0.1%.
- The deck's Q1 2026 CPI inflation range was 3.0–3.3%. March CPI inflation came in at 3.3%.
- As a benchmark, a plain monthly VAR in Brent (USD), CPI energy, CPI inflation and Bank Rate, fitted on 1997-01 to 2026-02, nowcasts March 2026 CPI inflation at **3.07%**. Against the 3.3% outturn that is a miss of **−0.2pp**. Its information set ends at the February monthly average, so it cannot see the March Brent jump.

## Why this method (vs alternatives)

- A monthly or quarterly VAR has to time-aggregate first, which throws away the within-period path where an energy shock shows up.
- A full state-space dynamic factor model also handles mixed frequencies, but it needs a joint model for every series. MIDAS gets parameter economy from the Almon polynomial and keeps one equation per target, so we can re-estimate it quickly for each data vintage.
- Where the frequency ratio is small we use U-MIDAS, since unrestricted lags cost little there and impose no weight shape.
- Bridge equations would need a forecast of the missing months. MIDAS uses the observed daily data directly.

## Limits

- Reduced form: the nowcast attributes no cause. Causal attribution belongs to the SVAR and LP layers.
- The Almon shape is smooth, with at most one turning point (the exponent is quadratic in $`j`$). Administered prices that step at discrete dates, such as the Ofgem cap, enter as calendar information; the lag polynomial does not capture them.
- Short samples per vintage and a structural break in energy pass-through make the scale coefficients unstable. We re-estimate them at each vintage but do not test them formally.
- We do not model GDP revisions. The nowcast targets the first release.
