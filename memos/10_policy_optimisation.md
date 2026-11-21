# Policy optimisation under model uncertainty

This memo uses the upstream scenario, regime and model outputs to make a Bank Rate decision. It minimises the Bank's own loss across states of the world and across models.
It draws on BoE Macro Technical Paper No.4 (Alati et al. 2025) for the loss function; BoE Macro Technical Paper No.1 for the smoothed rule; Taylor (1993); Woodford (2003); Svensson (2003); Brainard (1967); Hansen & Sargent (2008).

## Question it answers

We do not know where oil and wages settle, or which model is correct. Which Bank Rate setting at the April 2026 MPC gives the lowest expected loss? And how far would the state of the world have to move before the answer changes?

## Specification

We use a standard quadratic intertemporal loss in the form of MTP No.4:

```math
\mathcal{L}(i \mid s, m) = \mathbb{E}_t \sum_{h=0}^{H} \beta^{h}\Big[(\pi_{t+h}-\pi^*)^2 + \lambda\, y_{t+h}^2 + \mu\,(\Delta i_{t+h})^2\Big],
```

where $`\pi`$ is CPI inflation, $`\pi^* = 2\%`$, $`y`$ is the output gap, $`\Delta i`$ is the change in Bank Rate (so the $`\mu`$ term penalises rate volatility), and $`\beta`$ is the discount factor. The relative weights $`\lambda`$ and $`\mu`$ come from the paper. We do not change them.

The state $`s = (\text{oil}, \text{wages})`$ runs over a grid of plausible Brent levels and regular pay growth rates. Each $`(s, m)`$ pair produces paths for $`\pi`$ and $`y`$ under each candidate policy $`i \in \{\text{cut}, \text{hold}, \text{hike}\}`$. Model $`m`$ indexes the transmission models in the stack (LP/SVAR scenario projections and the TANK DSGE).

We choose the policy with the lowest expected loss. We then check it against the worst case:

```math
i^{\star} = \arg\min_i \sum_{m} \omega_m \sum_{s} p_s\, \mathcal{L}(i \mid s, m), \qquad i^{\text{rob}} = \arg\min_i \max_{m} \sum_s p_s\, \mathcal{L}(i \mid s, m).
```

Here $`p_s`$ are the scenario weights implied by oil options and $`\omega_m`$ are the model weights. The spread of outcomes across $`m`$ captures multiplicative uncertainty about transmission. This spread pushes policy toward gradualism (Brainard 1967). The min-max criterion is the robust-control check (Hansen & Sargent 2008).

The benchmark is the smoothed Taylor rule from MTP No.1:

```math
i_t = \rho\, i_{t-1} + (1-\rho)\big[r^* + \pi^* + \phi_\pi(\pi_t-\pi^*) + \phi_y\, y_t\big],\qquad \rho = 0.91,\; \phi_\pi = 1.357,\; \phi_y = 1.072.
```

The regime-adjusted variant weights the CPI inflation and output-gap inputs by the regime vector (Stagflation 66%). The Iran-conditional variant uses inputs from the $125 oil path.

## Data

The inputs are CPI (ONS D7BT, D7CH), real GDP (ABMI) for the output gap, regular pay growth, Bank Rate (IUQABEDR) and Brent (EIA RBRTE). Scenario weights come from the Hormuz engine and the regime vector from the regime layer.

## Key result

HOLD minimises expected loss under current conditions. HIKE becomes optimal only under sustained oil escalation (around $125).

| Rule | Implied rate | Gap vs 3.75% |
|---|---|---|
| Standard (no smoothing) | 4.40% | +65bp |
| Smoothed ($`\rho = 0.91`$) | 3.81% | +6bp |
| Regime-adjusted (HMM 66% stagflation) | 3.86% | +11bp |
| Iran-conditional ($125 oil) | 3.97% | +22bp |

The MTP No.1 rule box reports 3.84% (+9bp). Every smoothed variant sits within about 25bp of 3.75%. The decision flips to HIKE only if oil reaches about $120 or more, or wage growth exceeds 4.5–5%. AWE growth was 3.8% and falling at the time (regular pay +3.6%).

In effect this is a reduced-form reaction function under uncertainty, derived from the BoE's own objective.

## Why this method (vs alternatives)

A single Taylor rule gives a point answer from one model and one state. The +65bp gap of the unsmoothed rule shows that this answer can mislead. We minimise the Bank's stated loss over a state grid and across models. The trade-off is then explicit, and the flip thresholds come out of the calculation. The Brainard and robust-control results explain why the answer is a hold with a tightening bias rather than a pre-emptive hike.

## Limits

- The state grid and the scenario weights $`p_s`$ are discrete and partly judgemental. Prediction-market weights put more mass on collapse than oil options do.
- We do not estimate the model weights $`\omega_m`$.
- The loss weights come from the paper. We do not re-estimate them for an energy-shock regime.
- The output gap is unobserved and subject to revision. This is important because $`\phi_y > 1`$.
