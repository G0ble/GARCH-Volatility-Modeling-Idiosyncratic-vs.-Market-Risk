# GARCH(1,1) Volatility Modeling — Idiosyncratic vs. Market Risk
 
Estimating and interpreting the time-varying volatility of a single equity (Nike, `NKE`) with a **GARCH(1,1)** model, then separating **stock-specific risk** from **broad market fear** by benchmarking the fitted conditional volatility against the VIX.
 
**Headline result:** the fitted model yields a volatility persistence of **α + β ≈ 0.955**, confirming strong volatility clustering, and recovers fat-tailed return behavior (Student's t with **ν ≈ 3.4** degrees of freedom) that a Gaussian model would badly misprice.
 
---
 
## Why this project
 
Volatility is the central input to option pricing, risk management, and position sizing — and unlike returns, it is *partially predictable*. Financial returns exhibit **volatility clustering**: turbulent days follow turbulent days. GARCH models capture this directly by letting today's variance depend on yesterday's shock and yesterday's variance.
 
This project does three things:
 
1. **Diagnoses** the return distribution to justify the modeling choice (rather than assuming normality).
2. **Fits** a GARCH(1,1) model via maximum likelihood and interprets every parameter economically.
3. **Contextualizes** the output by asking a real question: when `NKE` volatility spikes, is it Nike-specific news, or is the whole market scared? The VIX overlay answers this visually.
---
 
## The model
 
GARCH(1,1) models the conditional variance as:
 
$$\sigma^2_t = \omega + \alpha\, \epsilon^2_{t-1} + \beta\, \sigma^2_{t-1}$$
 
| Parameter | Meaning | Fitted value |
|-----------|---------|--------------|
| **ω** (omega) | Baseline / long-run variance floor | 0.214 |
| **α** (alpha) | Reactivity to the most recent shock | 0.036 |
| **β** (beta) | Persistence of past volatility | 0.918 |
| **ν** (nu) | Degrees of freedom (tail thickness) | 3.44 |
 
The long-run (unconditional) variance is recovered as $\sigma^2 = \omega / (1 - \alpha - \beta)$, valid only because **α + β < 1** (the stationarity condition holds here, at 0.955).
 
### Reading the parameters
 
- **β = 0.918** — volatility is highly persistent. A shock today still echoes in the variance many days later. This is typical of equity markets.
- **α = 0.036** — the model reacts modestly but measurably to fresh shocks.
- **α + β = 0.955** — close to 1, the hallmark of slowly-decaying volatility regimes. Markets stay calm for long stretches, then stay turbulent for long stretches.
- **ν = 3.44** — very fat tails. With only ~3.4 degrees of freedom, extreme moves are far more likely than a normal distribution predicts. This is *why* the Gaussian assumption was rejected (see below).
---
 
## Methodology
 
1. **Data** — daily `NKE` close prices (2021-01 to 2026-01), ~1,254 observations, via `yfinance`.
2. **Returns** — simple returns converted to log returns, scaled ×100 for numerical stability of the optimizer.
3. **Distributional diagnostic** — a **QQ-plot** against the normal distribution shows clear departure from normality in both tails, with the lower tail heavier than the upper (negative shocks larger than positive ones — the well-known leverage/asymmetry effect in equities). This rules out a Gaussian likelihood.
4. **Estimation** — GARCH(1,1) fitted by maximum likelihood with a **Student's t** innovation distribution (`arch` library) to accommodate the fat tails.
5. **Market-risk decomposition** — the fitted conditional volatility is overlaid on the **VIX** to distinguish idiosyncratic spikes (Nike-only) from systematic spikes (whole-market).
---
 
## Limitations & next steps
 
This is a foundational implementation, in which we have not yet explored the following:
 
- **Skewed innovations** — the diagnostic identified asymmetry between the tails, but the model uses a symmetric Student's t. A **skew-t** distribution would capture this and is the natural immediate refinement.
- **Asymmetric volatility (leverage effect)** — standard GARCH treats positive and negative shocks symmetrically. **GJR-GARCH** or **EGARCH** would model the empirical fact that negative returns raise future volatility more than positive ones of equal size.
- **No out-of-sample forecasting** — the current scope is in-sample estimation and interpretation. A natural extension is rolling one-step-ahead volatility forecasts evaluated against realized volatility (e.g. via QLIKE or MSE loss).
- **Single asset** — extending to a panel of stocks would allow comparison of persistence and tail-thickness across sectors.
- **VIX comparison is qualitative** — the overlay is visual; a formal step would be regressing GARCH volatility on the VIX to quantify the share of `NKE` risk explained by systematic market fear.
