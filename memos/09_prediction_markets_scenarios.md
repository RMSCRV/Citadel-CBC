# Prediction markets and the Hormuz scenario engine

This layer turns traded beliefs about the Strait of Hormuz into scenario weights for the CPI inflation, Ofgem-cap and policy-loss layers. It draws on risk-neutral density extraction from option prices (second strike-derivative of call prices); scenario-weight robustness after Hansen & Sargent (2008); oil-to-CPI inflation mapping from Kilian (2009) and Jordà (2005), as the SVAR/LP memo documents.

## Question it answers
How likely is each Hormuz outcome? Do event contracts and oil options, the two markets that price it, agree? At submission, the crisis had no resolution after 54 days.

## Specification
**Contracts.** Polymarket and Kalshi binaries on (i) weekly Hormuz tanker transits above or below set counts, (ii) Brent, WTI and Dubai settling above strike $`K_j`$ by date $`T`$, (iii) ceasefire or deal by a given date.

**De-vigging.** For a set of mutually exclusive outcomes with best-ask prices $`q_k`$, the overround is $`O=\sum_k q_k-1`$. We normalise proportionally:

```math
p_k=\frac{q_k}{\sum_m q_m},\qquad \text{binary: } p=\frac{q^{\text{yes}}}{q^{\text{yes}}+q^{\text{no}}}.
```

We use bid–ask mids where depth allows. We drop quotes with no recent trade.

**Thresholds to a CDF.** Each "above $`K_j`$" contract gives a survival point $`S(K_j)=P(B_T>K_j)`$, so $`\hat F(K_j)=1-p_j`$. Raw points can be non-monotone across venues. We project them onto non-decreasing sequences (isotonic regression, pool-adjacent-violators). We then interpolate linearly in $`K`$ with flat tails.

**Options leg.** From Brent call prices $`C(K,T)`$ across strikes, the risk-neutral density is

```math
f_Q(K)=e^{rT}\,\frac{\partial^2 C(K,T)}{\partial K^2},\qquad F_Q(K)=1+e^{rT}\,\frac{\partial C(K,T)}{\partial K},
```

with finite differences on a smoothed implied-volatility smile.

**Scenario weights.** Each scenario $`s`$ is a region of (Brent, transits, deal-timing) space. Its probability is the mass in that region, e.g. $`p_s=F(\bar K_s)-F(\underline K_s)`$ for a price band, checked against the transit and ceasefire contracts.

## Data
Polymarket and Kalshi public APIs (daily snapshots); Brent futures and options; EIA RBRTE for Brent history.

## Key result
| Scenario | Q3 CPI | July Ofgem cap | Options-implied probability |
|---|---|---|---|
| Rapid resolution | 4.2% | £1,877 | 29% |
| Limbo | 4.3% | £1,941 | 54% |
| Collapse | 4.8% | £2,053 | 17% |

Prediction markets put 46% on collapse. Oil options put 17% on collapse. We read this gap as hedging bias: producer and consumer hedgers dominate commodity option markets, and these markets under-price extreme outcomes. 76% of the probability mass sat above BoE projections. All three paths keep CPI inflation above target through Q3.

**Mapping to the README scenarios**: We split Rapid resolution equally into Peace deal 14.5% and Normalisation 14.5%. Limbo maps to Status quo (54%). Collapse maps to Escalation (17%). The expected Bank Rate is $`E[i]=\sum_s p_s\, i_s`$. Prediction-market weights move mass to Escalation and raise $`E[i]`$. This result supports keeping June live.

## Why this method
Options give deep, continuously priced distributions, but only over price. They also carry risk premia from hedging. Event contracts price the political trigger (transits, deal timing) directly. We use both and treat their gap as a measure of model uncertainty. A single analyst-set scenario tree would hide that disagreement.

## Limits
- Neither measure is physical probability. Option densities embed risk premia, and event prices embed fees, long-shot bias and capital constraints.
- Event contracts are thin. A few large orders can move a threshold price. We require trades in the snapshot window, cross-check venues and treat single-venue moves as noise.
- Contract definitions (settlement source, dates, transit definition) differ across venues and from our scenario boundaries.
- Tail interpolation above the highest listed strike is an assumption.
