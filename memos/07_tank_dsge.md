# TANK-DSGE with energy, expectations and mortgage channels

This is the structural layer of the stack. The reduced-form models (recursive VAR, LP, Bayesian SVAR) measure how much energy shocks pass into CPI; this model shows the mechanism and what a policy rule has to do about it. It builds on Galí, López-Salido & Vallés (2007, JEEA); Galí & Monacelli (2005, REStud); Erceg, Henderson & Levin (2000, JME); Cloyne, Ferreira & Surico (2020, REStud); Burgess et al. (2013, BoE WP 471, COMPASS); Albuquerque et al. (2025, BoE Macro Technical Paper No. 1); Milani (2007, JME); Sims (2002); Waggoner & Zha (1999); Alati et al. (2025, MTP No. 4); Haberis, Masolo & Reinold (2025, MTP No. 5).

## Question it answers
How far, and for how long, does an imported energy shock spread into UK CPI when some households cannot smooth consumption, mortgage payments reset only as fixed deals expire, and inflation expectations come from markets? And which Bank Rate path is consistent with that?

## Specification
We start from the MTP No. 1 version of COMPASS and change it in three places: a two-agent household block, a mortgage cash-flow term, and market-based expectations at the observation level. The model is a log-linear small open economy written in the Sims (2002) canonical form, with 36 states and 10 structural shocks:

```math
\Gamma_0 s_t = \Gamma_1 s_{t-1} + \Psi \varepsilon_t + \Pi \eta_t, \qquad s_t = T s_{t-1} + R \varepsilon_t, \qquad y_t = Z s_t + D + u_t,\ u_t \sim N(0,H)
```

**Households.** Optimising households have external habits. Their Euler equation carries one extra term, the change in the effective mortgage rate. Hand-to-mouth households spend current real labour income. In the calibration 16.8% of households are hand-to-mouth, at the low end of the Ampudia et al. (2018) range for the UK.

```math
c^o_t = A_f E_t c^o_{t+1} + A_b c^o_{t-1} - A_r\big(r_t - E_t\pi^{CPI}_{t+1} - E_t\gamma^z_{t+1} + \varepsilon^b_t\big) - \phi_m\,\Delta r^{mort}_t
```

```math
c^{h}_t = \alpha_{lab}\,(w_t + l_t - p^c_t), \qquad c_t = \omega_o c^o_t + (1-\omega_o)\,c^h_t
```

**Mortgage cash flow.** Cloyne et al. (2020) find that consumption responds to policy almost entirely through mortgagors. We build the effective mortgage rate from a variable-rate share plus a fixed-rate share that reprices over eight quarters:

```math
r^{mort}_t = \omega_{var} BR_t + \omega_{fix}\,\tfrac{1}{8}\textstyle\sum_{k=0}^{7} BR_{t-k} + \text{spread}
```

Its change is a state variable, follows an AR(1), and is observed. The filter sees the actual repricing path. That matters for 2026: deals struck in 2021, when Bank Rate sat at 0.1%, are still rolling off, and 1.8m fixed-rate deals expire this year.

**Prices and wages.** Hybrid Calvo curves for goods prices and for wages (Erceg et al., 2000), each with persistent and transitory markup shocks. The real wage cumulates wage minus price inflation:

```math
\pi^z_t = \kappa_Z\,mc^z_t + \beta_b\,\pi^z_{t-1} + \beta_f\,E_t\pi^z_{t+1} + \mu^z_t + \mu^{z,temp}_t, \qquad w_t = w_{t-1} + \pi^W_t - \pi^z_t
```

**Energy.** Energy enters at three levels: directly in the CPI basket, in firms' marginal cost, and as a separate term in the policy rule. The world energy price is AR(1). Export and import volumes respond to the relative energy price.

```math
\pi^{CPI}_t = \pi^z_t + \Delta p^c_t,\quad p^c_t = \alpha_{ec}\,p^e_t, \qquad p^e_t = \rho_E\,p^e_{t-1} + \varepsilon^{pe}_t
```

**Policy.** A smoothed Taylor rule. It responds mainly to inflation excluding the energy contribution $`econ_t`$, so first-round energy effects are largely looked through:

```math
r_t = \theta_r r_{t-1} + (1-\theta_r)\big[\theta_\pi\big(\pi^{CPI}_t - (1-\theta_E)\,econ_t\big) + \theta_y\,\tilde y_t\big] + \varepsilon^r_t
```

**Expectations.** We keep rational expectations inside the structural equations, so the Blanchard-Kahn solution survives. Market expectations enter only as observables that pin down the forward-looking states:

```math
\pi^{exp,1Y}_t = \tfrac14\pi^z_t + \tfrac34 E_t\pi^z_{t+1} + e^{1Y}_t, \qquad \pi^{exp,5Y5Y}_t = \tfrac1{10}\pi^z_t + \tfrac9{10}E_t\pi^z_{t+1} + e^{5Y5Y}_t
```

When the anchoring gap from the expectations layer passes a threshold $`\tau`$, the effective target moves, $`\pi^*_t = 2\% + \kappa\,\text{gap}_t\,\mathbb{1}[\text{gap}_t > \tau]`$, following MTP No. 5.

**Solution and estimation.** We solve with QZ (gensys) and check the Blanchard-Kahn condition on every draw. The Kalman filter supplies the likelihood. Random-walk Metropolis-Hastings samples the posterior, with priors centred on MTP No. 1 values. COVID-era quantity observables are treated as missing, not imputed.

## Data
148 quarters, 1987Q1 to 2023Q4, and 15 observables: real GDP (ONS ABMI) and its expenditure components, hours, trade, CPI (D7BT), energy CPI weighted by MM23 shares, AWE, Bank Rate (IUQABEDR), sterling ERI, world activity, prices and rates. The effective mortgage rate change comes from our own pipeline. The 1y and 5y5y expectations come from the gilt and swap term-structure model, after the RPI-CPI wedge adjustment.

## Scenario design
Every scenario starts from the smoothed state and actual values at the forecast origin. The baseline is the unconditional forecast, with policy set by the rule. Energy scenarios are paths for the energy shock that differ in size and persistence. Rate paths are imposed with Waggoner-Zha hard conditioning, which also tells us how large the policy shocks needed to hold them would be. Hold-versus-hike impulse responses run over 12 quarters, across a range of oil shocks. The policy call is then scored with the MTP No. 4 optimal-policy loss, $`L_t = (\pi_t-\pi^*)^2 + \lambda(y_t-y^*)^2 + \gamma(\Delta i_t)^2`$, and stress-tested by minimax over an oil-by-wages uncertainty set. We do not publish point outputs from the model.

## Why this method
A VAR or LP gives the size of pass-through. It cannot split first-round from second-round effects or show how a rule should respond. The model adds the two channels a representative-agent COMPASS leaves out, hand-to-mouth spending and lagged mortgage repricing. Together they make the output response to tightening slower and longer, with a delayed trough. Staying close to COMPASS keeps the model readable for anyone who knows the Bank's own suite.

## Limits
- The hand-to-mouth share, energy weights and fixed-rate share are calibrated, not estimated.
- Variance decompositions were computed at the prior mode, before full MCMC convergence. We read them for direction only.
- The energy calibration is small, so the model understates the energy share that the SVAR finds.
- The estimation sample ends in 2023Q4, before the 2026 energy shock.
- The model is linear and the two-agent split is fixed, so it misses state dependence at large shocks.
- Risk and liquidity premia reach the expectations observables only through measurement error.

For magnitudes we rely on the reduced-form models; the DSGE is used for direction and mechanism.
