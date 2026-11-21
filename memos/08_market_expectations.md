# Market-Based Inflation Expectations: Swaps, 1y1y, 5y5y

This module turns UK inflation-linked market prices into a CPI-consistent term structure of expectations and passes an anchoring signal to the regime layer, the DSGE expectations block and the policy rule. It builds on Joyce, Lildholdt & Sorensen (2015), BoE Staff Working Paper 551, extended for the 2030 RPI reform; Kalman (1960); Durbin & Koopman (2012).

## Question it answers
Does the energy shock move only near-term inflation compensation, or does it reach the long-horizon anchor the MPC cares about? Put differently: is this a headline problem or a credibility problem?

## Specification
The MPC targets CPI. Index-linked gilts and UK inflation swaps both pay on RPI, so every market breakeven is an RPI number. Read raw, it overstates CPI expectations. For a gilt breakeven at maturity $`n`$:

```math
\text{BEI}(n,t) = E_t\left[\pi^{CPI}_{t,n}\right] + w(n,t) + \rho(n,t) - \ell(n,t)
```

The four terms are the CPI expectation we want, the RPI-CPI wedge $`w`$, the inflation risk premium $`\rho`$, and the liquidity premium $`\ell`$ that index-linked gilts carry relative to nominal gilts.

**The wedge.** RPI has run 70-100bp above CPI. Two things drive it. The formula effect, worth 50-60bp and permanent under current methods, comes from RPI averaging prices arithmetically where CPI uses a geometric mean. The rest is housing: RPI includes mortgage interest payments and CPI does not, so the wedge widens when mortgage rates rise.

**Liquidity from swaps.** A swap needs no balance-sheet commitment from either side and carries no material liquidity premium. At the same maturity, the gap between the two breakevens identifies the gilt liquidity premium directly:

```math
\text{BEI}^{swap}(n,t) - \text{BEI}^{gilt}(n,t) = \ell_t
```

Without swap data this term has to be calibrated from historical estimates. Swaps also reprice daily, while the BoE gilt curves arrive with a one-month lag.

**The 2030 reform and a horizon-specific wedge.** In November 2020 the UK Statistics Authority confirmed that RPI will be aligned to CPIH from February 2030. That shrinks the wedge from 0.74pp to 0.28pp, the CPIH-CPI gap. A single spot wedge is therefore wrong for any contract that spans the reform date. We use

```math
w(n,t) = \frac{m^{pre}_{n,t}\, w^{pre}_t + m^{post}_{n,t}\, w^{post}}{12n}
```

with $`m^{pre} = \min(12n,\ \text{months to Feb 2030})`$, $`m^{post} = 12n - m^{pre}`$, $`w^{pre}_t`$ the 12-month rolling mean of the realised RPI-CPI wedge, and $`w^{post} = 0.28`$pp. The rolling mean smooths RPI's seasonal and housing noise and stops a past mortgage-rate spike being projected forward. As of April 2026 the 5y5y window (2031-36) sits entirely after the reform. Subtracting the spot 0.74pp wedge there would over-correct by 46bp and push the anchoring gap down by the same amount.

**Forwards.** With $`s_n`$ the zero-coupon RPI swap rate at maturity $`n`$:

```math
\pi^{1y1y}_t = \frac{(1+s_{2,t})^2}{1+s_{1,t}} - 1, \qquad \pi^{5y5y}_t = \left(\frac{(1+s_{10,t})^{10}}{(1+s_{5,t})^{5}}\right)^{1/5} - 1
```

The short end reads headline pass-through. The 5y5y forward reads perceived persistence, and it is the measure the Bank itself watches for anchoring.

**State-space term structure.** Four latent factors, $`f_t = [\mu_t, \lambda_t, \rho_t, \ell_t]'`$: the level of expected CPI inflation (the anchor), a Nelson-Siegel slope, the inflation risk premium, and the gilt liquidity premium. Each follows a stationary AR(1), $`f_t = A f_{t-1} + \eta_t`$ with diagonal $`A`$. The wedge is removed before measurement:

```math
y^{swap}_{n,t} - w(n,t) = \mu_t + g(n)\lambda_t + \rho_t + \varepsilon_{n,t}, \qquad y^{gilt}_{n,t} - w(n,t) = \mu_t + g(n)\lambda_t + \rho_t - \ell_t + \varepsilon_{n,t}
```

with $`g(n) = (1-e^{-n/\tau})/(n/\tau)`$. The gilt-swap spread pins down $`\ell_t`$. Cross-maturity shape pins down $`\lambda_t`$. Level and risk premium both load one-for-one, so they are separated by persistence: the premium is set to mean-revert faster than the near-permanent anchor. A Kalman filter handles missing tenors by dropping rows from the update, and a Rauch-Tung-Striebel smoother produces the full-sample states. If swap data are unavailable, the model falls back to a gilt-only two-state version with the liquidity and risk premia calibrated from SWP 551. Output: the anchoring gap $`\mu_t - 2\%`$.

## Data
RPI zero-coupon swaps at 1y/2y/5y/10y and the 5y5y forward; BoE nominal, real and breakeven gilt curves; SONIA (BoE IUQASOIA); ONS RPI and CPI (D7BT) indices for the realised wedge.

## Key result
Smoothed and wedge-adjusted, March 2026: 1y ≈ 2.53%, 5y5y ≈ 2.21%, anchoring gap ≈ +0.09pp. The short end moved more than the long end and the 5y5y held. We read that as partial de-anchoring at short horizons with the long-run anchor intact.

In the regime layer, the small gap is the expectations cross-check on the stagflation read: energy is pushing prices up, but markets are not yet pricing a break in credibility. In the DSGE, the smoothed 1y and 5y5y points enter as extra observables, matched to the model's average inflation forecast over the first year and over years six to ten, so model-consistent expectations are disciplined by what markets price. In the policy layer, the gap sets an effective inflation target, $`\pi^*_t = 2 + \phi \max(0,\ \mu_t - 2 - \kappa)`$, that only rises once the gap clears a threshold $`\kappa`$. At +0.09pp it does not bind. That supported a hold with a tightening bias rather than an immediate hike, and it made the 1y RPI swap versus 5y5y steepener the natural trade if escalation fed through to the short end.

## Why this method
Raw breakevens mix expectations, risk, liquidity and the RPI basis. Read directly, they overstate long-run drift, and a flat wedge overstates the correction at the long end. Fitting the whole curve jointly separates the pieces, copes with missing maturities and gives uncertainty bands. Surveys are slower and lower frequency. A full affine no-arbitrage model would add nominal-curve factors and estimation cost without changing the anchoring read.

## Limits
The risk and liquidity premia are identified by loading and persistence restrictions; nobody observes them. The post-2030 wedge assumes the reform lands as announced. State dynamics are calibrated, not estimated by maximum likelihood. Swap liquidity is thinner at the 1y tenor. The model reads market pricing, not household or firm expectations.
