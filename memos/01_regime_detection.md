# Multi-layer regime detection

**Role in the stack:** classifies the current UK macro state into Stagflation / Demand shock / Disinflation and passes that regime vector to the policy rule and loss-function layers.
**Based on:** Hamilton (1989); Kalman (1960); Durbin & Koopman (2012); Shah et al. (2022) FLANG; Liu et al. (2019) RoBERTa; Araci (2019) FinBERT.

## Question it answers

Which macro regime is the UK in now? How certain is that result? Does the real-time and text evidence move before the slow structural data?

## Specification

Three layers run at three frequencies. The model pools their regime probabilities with fixed weights (50% HMM, 35% Kalman, 15% NLP):

```math
p_{k,t} = 0.50\,p^{\text{HMM}}_{k,t} + 0.35\,p^{\text{KF}}_{k,t} + 0.15\,p^{\text{NLP}}_{k,t}, \qquad k \in \{\text{Stag}, \text{Dem}, \text{Dis}\}.
```

**1. Structural layer (HMM).** A Gaussian hidden Markov model on a standardised 17-variable monthly macro panel $`x_t`$. The latent state is $`z_t \in \{1,\dots,K\}`$ with transition matrix $`P_{ij} = \Pr(z_t=j \mid z_{t-1}=i)`$, and $`x_t \mid z_t=k \sim \mathcal{N}(\mu_k, \Sigma_k)`$. EM (Baum-Welch) estimates the model. The Hamilton recursion gives the filtered probabilities:

```math
\xi_{t|t} = \frac{\xi_{t|t-1} \odot \eta_t}{\mathbf{1}'(\xi_{t|t-1} \odot \eta_t)}, \qquad \xi_{t+1|t} = P'\xi_{t|t},
```

where $`\eta_t`$ stacks the state densities. After estimation, the signs of the inflation and activity means give each state its economic label. This labelling is the identification step.

**2. Real-time layer (Kalman).** A linear-Gaussian state-space model on higher-frequency signals (energy prices, market-implied inflation, activity nowcasts):

```math
y_t = Z\alpha_t + \varepsilon_t,\quad \alpha_{t+1} = T\alpha_t + \nu_t,
```

Maximum likelihood estimates the model through the prediction-error decomposition. The regime-conditional distributions from layer 1 map the filtered states $`\alpha_{t|t}`$ (inflation and growth momentum) to $`p^{\text{KF}}_{k,t}`$. This layer showed the active oil shock before the monthly panel did.

**3. NLP layer (FLANG-RoBERTa).** Text is the fastest input. The pipeline is built to give a clean signal under API rate limits.

- *Sources:* news APIs (NewsAPI/GNews), web scraping (trafilatura), X/Twitter (X API), and MPC minutes, statements and speeches; 1,000+ articles/day.
- *Filters:* a trusted-domain allow-list, then a macro keyword filter (inflation, Bank Rate, wages, energy, gilts and similar terms).
- *Key-sentence extraction:* the pipeline keeps only forward-looking sentences (expectation and outlook verbs, future tense, conditional constructions). This step and the ingestion filters cut irrelevant content by 65%.
- *Classifier:* FLANG-RoBERTa (Shah et al., 2022) outputs a three-class distribution $`p = (P_{\text{up}}, P_{\text{neu}}, P_{\text{down}})`$ for each sentence, where "up" means hawkish or inflationary.

Sentence score and confidence:

```math
s = P(\text{up}) - P(\text{down}), \qquad H = -\sum_{i=1}^{3} p_i \log p_i, \qquad H_{\text{norm}} = \frac{H}{\log 3},
```

```math
w = \lvert P(\text{up}) - P(\text{down}) \rvert \,(1 - H_{\text{norm}}).
```

The weight is zero for a balanced up/down split and for a uniform (maximum-entropy) prediction. Ambiguous sentences therefore count for nothing. Aggregation goes from sentence to article to day:

```math
S_{\text{article}} = \frac{\sum_i w_i s_i}{\sum_i w_i}, \qquad S_{\text{day}} = \frac{\sum_j W_j S_j}{\sum_j W_j},
```

where $`W_j`$ is the article-level weight. A rolling window smooths $`S_{\text{day}}`$ to give $`\bar S_t`$.

*Entry into the regime vector:* the NLP layer is not a fourth regime model. The model uses $`\bar S_t`$ as an observation with regime-conditional density $`f(\bar S_t \mid k)`$. The HMM's historical regime labels give the fit for this density:

```math
p^{\text{NLP}}_{k,t} = \frac{f(\bar S_t \mid k)\, p^{\text{HMM}}_{k,t}}{\sum_m f(\bar S_t \mid m)\, p^{\text{HMM}}_{m,t}}.
```

## Data

The panel uses data from the ONS time-series API, the BoE IADB and EIA (e.g. D7BT, D7CH, ABMI, IUQABEDR, IUQASOIA, RBRTE). The real-time layer uses daily energy and market-implied inflation series. Text comes from the sources listed in layer 3.

## Key result

April 2026: **Stagflation 66%, Demand shock 34%, Disinflation 0%.** Before the conflict, the HMM showed disinflation. The Kalman layer showed the active oil shock and forward inflation risk. The NLP read of the MPC moved from a cut bias (Dec 2025) to a dovish hold (Feb 2026) to a firm hold (Mar 2026). The 66% stagflation probability goes into the regime-adjusted Taylor rule. That rule gives a Bank Rate of 3.86% (+11bp vs 3.75%).

## Why this method

A single HMM on monthly data is identified but slow. The Kalman layer adds speed and keeps that structure. The NLP layer finds the reaction function before the data shows it. FLANG-RoBERTa uses pre-training on financial-domain text. We benchmarked it against BERT and FinBERT (Araci, 2019). With entropy-weighted aggregation, it gave a more robust short-term signal than the BERT baseline.

## Limits

- Judgement sets the 50/35/15 weights. They are not estimated.
- A short sample makes a 17-dimensional emission covariance hard to estimate. The state labels are interpretive.
- Text coverage and source mix change over time. The trusted-domain list adds curation bias.
- The NLP likelihood is only as good as the historical HMM labels it is fitted on.
