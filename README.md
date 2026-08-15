[README.md](https://github.com/user-attachments/files/31109266/README.md)
# Monthly Domestic Air Travel Passenger Time Series Analysis

Time series analysis and forecasting of U.S. monthly domestic air travel passenger volumes (January 2003 – September 2023) using SARIMA and SARIMAX modeling, with a focus on isolating the COVID-19 shock as an exogenous disruption rather than unexplained noise.

## Overview

Domestic air travel passenger counts are a widely used indicator of economic activity, informing airline capacity planning and macroeconomic forecasting alike. This project models and forecasts monthly domestic passenger volumes using two approaches:

- **SARIMA**, fit via the classical Box-Jenkins methodology (ACF/PACF-driven parameter identification)
- **SARIMAX**, fit via `auto.arima` with a binary exogenous indicator variable flagging the COVID-19 disruption period (March 2020 – December 2021)

The dataset captures a sharp negative shock in 2020 due to the pandemic, which distorts standard autocorrelation-based parameter selection and inflates forecast uncertainty if left unaddressed. This project directly investigates whether modeling that shock as an exogenous regressor — rather than folding it into the stochastic noise term — improves both model fit and forecast precision.

## Data

- **Source:** Bureau of Transportation Statistics (T-100 Segment data)
- **Frequency:** Monthly, January 2003 – September 2023 (249 observations)
- **Range:** ~3.5M passengers (COVID trough) to ~90M passengers (peak season, pre/post-COVID)

## Methodology

**SARIMA(p,d,q)(P,D,Q)₁₂** — Box-Jenkins approach:
1. Log transformation to stabilize non-constant seasonal variance
2. Non-seasonal and seasonal differencing (d=1, D=1) to induce stationarity, seasonal period s=12
3. Parameter selection via ACF/PACF inspection of the differenced series
4. Diagnostic checks (standardized residuals, residual ACF, Q-Q plot, Ljung-Box test)
5. Model comparison via AIC/AICc/BIC and parameter significance

**SARIMAX(p,d,q)(P,D,Q)₁₂ with exogenous COVID indicator**:
- Fixes the same differencing structure (d=1, D=1, s=12) identified above
- Introduces a binary regressor u_t = 1 for March 2020 – December 2021, 0 otherwise
- Parameter selection via `auto.arima` (stepwise=FALSE, approximation=FALSE) rather than manual ACF/PACF analysis
- Same diagnostic and model-comparison procedure as the SARIMA model

## Results

| Model | Parameters | AIC | AICc | BIC |
|---|---|---|---|---|
| SARIMA | (0,1,2)(0,1,1)₁₂ | −0.3820 | −0.3816 | −0.3233 |
| SARIMAX | (0,1,3)(0,1,2)₁₂ | **−0.4362** | **−0.4346** | **−0.3334** |

Both models pass diagnostic checks (Ljung-Box p > 0.05 at lag H=20; residual ACF within confidence bounds), though both show elevated standardized residuals during 2020 reflecting the magnitude of the pandemic shock.

**Key finding:** The SARIMAX specification achieves a lower AIC, AICc, and BIC than the best Box-Jenkins SARIMA model, and its COVID indicator variable is highly significant (p ≈ 0). Structurally, this is because the exogenous regressor absorbs the 2020 shock as an explained, one-time disruption rather than requiring the stochastic MA/AR terms to account for it as unexplained variance. This has a direct consequence for forecasting: the SARIMAX 12-month forecast produces a **narrower 95% prediction interval** than the SARIMA forecast, since less residual uncertainty is attributed to ordinary process noise. In effect, explicitly modeling a known structural break as an exogenous covariate is more efficient than allowing a purely autoregressive model to internalize a large, transient shock through its noise structure.

## Repository Structure

```
.
├── air_traffic.csv          # Raw monthly passenger data (BTS T-100 Segment)
├── PSTAT174_FinalProj.Rmd   # Full R Markdown analysis and code
├── PSTAT174_FinalProj.pdf   # Rendered writeup with figures and diagnostics
└── README.md
```

## Tools

R, with `astsa` (SARIMA/SARIMAX fitting and diagnostics via `sarima`/`sarima.for`) and `forecast` (`auto.arima`) as the core modeling packages.

## Author

Alexandru Tudor
