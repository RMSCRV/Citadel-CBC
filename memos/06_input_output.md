# Leontief input-output cost-push model

This model traces an energy price shock through the UK production network to sector-level cost increases, showing where the shock lands alongside the aggregate SVAR/LP pass-through estimates. It builds on Leontief (1936) price model; Kilian (2009) for the oil-shock transmission context.

## Question it answers

If energy input prices rise, how much do unit costs rise in every other industry? The answer counts indirect exposure: energy embodied in intermediate inputs bought from other sectors. Which parts of the CPI basket carry that pressure?

## Specification

Let $`Z_{ij}`$ be purchases of industry $`i`$'s output by industry $`j`$ and $`x_j`$ gross output of $`j`$. Technical coefficients are $`a_{ij} = Z_{ij}/x_j`$, collected in $`A`$ ($`n \times n`$). Primary-input (value-added plus imports and taxes) cost per unit of output is $`v_j`$. With competitive, cost-plus pricing, each sector's unit price equals intermediate plus primary cost:

```math
p_j = \sum_i a_{ij}\,p_i + v_j \quad\Longrightarrow\quad p = (I - A^\top)^{-1} v = L^\top v,
```

where $`L = (I-A)^{-1}`$ is the Leontief inverse. Prices are normalised to $`p = \mathbf{1}`$ in the base year.

**Shock.** The energy shock enters as a proportional increase $`\delta`$ in the primary-input cost of the energy sectors $`E`$ (mining and quarrying incl. oil and gas extraction; coke and refined petroleum; electricity and gas supply):

```math
\Delta v_j = \delta\, v_j \,\mathbb{1}[j \in E].
```

**Total cost increase** by sector, direct plus indirect:

```math
\Delta p = (I - A^\top)^{-1} \Delta v, \qquad \text{indirect}_j = \Delta p_j - \Delta v_j .
```

**CPI mapping.** Sector price changes are bridged to consumption categories with CPI basket weights $`w_k`$ and a sector-to-COICOP concordance $`B`$ (rows sum to one):

```math
\Delta \pi^{CPI} \approx \sum_k w_k \,(B\,\Delta p)_k .
```

**Identification and estimation.** The model has no econometric estimation. We read $`A`$ from the published table. The result is an accounting identity, conditional on the pricing assumption. The key assumption is **full, immediate pass-through** with fixed technical coefficients (no substitution, no margin compression). Thus $`\Delta p`$ is an upper bound on the eventual cost-push.

## Data

- ONS UK input-output analytical tables (IOAT), **industry-by-industry**: intermediate flows $`Z`$, gross output $`x`$, primary inputs.
- ONS CPI basket weights by COICOP class for the consumption bridge; CPI all items (D7BT) and CPI electricity, gas and other fuels (D7CH) for context.

## Key result

The largest cost pressures fall on **chemicals, non-metallic mineral products, and utilities**, all energy-intensive processes. The sector map sits alongside the deck's aggregate rule of thumb from the SVAR/LP work: every sustained $10/bbl increase in oil adds ~40bp to CPI inflation. For comparison, a 10% Brent (USD) shock raises CPI inflation by 0.37pp at peak (month 11) in the LP and 0.35pp at peak (month 11) in the SVAR.

## Why this method (vs alternatives)

The SVAR and LP measure *how much* CPI moves after an oil shock but not *through which industries*. The Leontief price model uses official data directly and makes the indirect channel visible. That matters for persistence: costs that pass through the supply chain over successive pricing rounds can embed into inflation rather than fade with the commodity price. A CGE or multi-sector DSGE allows substitution and margin responses. But it adds calibration risk, and this question only needs the cost map.

## Limits

- Fixed coefficients and full pass-through overstate the effect; firms substitute, absorb margins, or hedge.
- The model is static: it gives the cumulative level of cost-push, not its timing. Timing comes from the LP/SVAR impulse responses.
- IOAT vintages lag several years, so the network may not reflect the current energy mix.
- The model treats the import content of energy and the sterling exchange rate as part of $`v`$. It does not model them separately.
