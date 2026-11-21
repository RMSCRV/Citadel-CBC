# Gas Prices, Storage and the Energy-Cost Pipeline

**Role in the stack:** turns the wholesale gas market (NBP forwards, storage, LNG exposure) into a dated path for the household energy bill and the CPI energy component, and feeds that path to the scenario engine and the policy layer.
**Based on:** Kilian (2009) and Baumeister & Hamilton (2019) for treating energy as a supply shock rather than a demand shock; Leontief (1936) for the downstream cost propagation; Ghysels, Santa-Clara & Valkanov (2004) for bringing daily energy prices into the monthly nowcast.

## Question it answers
When does a gas price shock reach UK CPI, by how much at each reset, and how much of the July cap has already been set by prices the market has printed?

## Specification
**Cap replication.** Ofgem sets the wholesale component of the cap for delivery period $`c`$ by averaging forward prices over a fixed observation window $`\mathcal{O}_c`$ that closes before the cap is announced. We rebuild it from the NBP forward month curve:

```math
W_c=\frac{1}{|\mathcal{O}_c|}\sum_{t\in\mathcal{O}_c}\;\sum_{m\in\mathcal{D}_c}\omega_m\,F_t(m),\qquad \sum_m\omega_m=1,
```

where $`F_t(m)`$ is the NBP forward for delivery month $`m`$ quoted on day $`t`$, $`\mathcal{D}_c`$ the delivery months of period $`c`$, and $`\omega_m`$ a seasonal demand profile. The cap moves as

```math
\text{Cap}_c=\text{Cap}_{c-1}+\kappa\,(W_c-W_{c-1})+\Delta N_c,
```

with $`\kappa`$ the pass-through of wholesale cost to the typical bill and $`\Delta N_c`$ the change in network and policy costs. Partway through a window, the realised days fix part of $`W_c`$. Only the remaining days are scenario-dependent:

```math
W_c=\tfrac{k}{|\mathcal{O}_c|}\,\bar F_{\text{obs}}+\Big(1-\tfrac{k}{|\mathcal{O}_c|}\Big)\,\mathbb{E}_s\!\left[\bar F_{\text{rem}}\right].
```

**CPI mapping.** The cap change enters household energy CPI (D7CH) in the month the cap takes effect, as a discrete step in the price level. Its contribution to month-on-month CPI inflation is $`\pi^{\text{cap}}_t\approx s_{\text{dom}}\,\Delta\ln\text{Cap}_c\,\mathbf{1}[t=\text{start}_c]`$, where $`s_{\text{dom}}`$ is the weight of household gas and electricity in CPI. Motor fuels (D7EC) reprice from Brent within weeks. Gas reaches CPI inflation only at the quarterly reset, after the observation-window lag.

**Storage and LNG.** We track UK storage as a share of capacity against its own 3-year history. A low starting level raises refill demand through the injection season and supports the summer strip. LNG sets the marginal supply cargo, so NBP moves with the global LNG market.

## Data
NBP forward month curve (GBX/therm), daily; UK gas inventories as % of storage capacity; Ofgem cap announcements; ONS D7CH, D7EC, D7BT; EIA RBRTE (Brent). Downstream sector costs come from the ONS industry-by-industry input-output tables (IOAT).

## Key result
- The Q2 2026 cap was £1,641/yr, 6.6% below Q1, because its observation window closed before the conflict started. July is the first reset that prices the conflict.
- July cap by Hormuz scenario: £1,877 / £1,941 / £2,053 (Rapid resolution / Limbo / Collapse). Futures-implied weights are 29% / 54% / 17%, and Q3 CPI is 4.2% / 4.3% / 4.8%. The expected cap is $`\mathbb{E}[\text{Cap}]=\sum_s p_s\,\text{Cap}_s`$.
- Q3 CPI differs little between Rapid resolution and Limbo (4.2% vs 4.3%), consistent with part of the July window already being fixed by realised prices.
- Storage was at 3-year lows before the injection season. The UK imports about 40% of its energy, and oil and gas prices were up 35% since the conflict began.
- In the input-output tables, the largest cost pressures fall on chemicals, non-metallic minerals and utilities.

## Why this method
Replicating the regulator's own formula gives the date and size of each CPI step. A reduced-form regression of CPI on gas prices blurs the quarterly reset and the window lag into a smooth distributed lag. Separate oil and gas channels also show why headline CPI can fall in Q2 while the shock builds.

## Limits
The observation-window dates, the demand profile $`\omega_m`$ and the pass-through $`\kappa`$ approximate Ofgem's published methodology. The analysis also ignores fixed-tariff switching and any government bill support. Storage enters only as a qualitative tightness indicator, with no structural supply-demand model.
