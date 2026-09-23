# Seasonal ARIMA & Spectral Analysis of Melbourne Temperatures

How cold does Melbourne get from month to month, and can a simple model forecast it? I took ten years of daily minimum temperatures (1981–1990) and averaged them into 120 monthly values. Then I modeled the series two different ways: SARIMA on the time side, and spectral analysis on the frequency side.

Spoiler: both methods tell the same story. One yearly cycle explains almost everything.

## The data

[Daily Minimum Temperatures in Melbourne](https://www.kaggle.com/datasets/paulbrabban/daily-minimum-temperatures-in-melbourne) on Kaggle, originally from the [Australian Bureau of Meteorology](https://www.bom.gov.au).

The raw file has 3,650 daily readings in °C. Daily values are noisy, so I averaged them by month to bring out the seasonal pattern.

## What I did

1. Plotted the series and ran ADF tests for stationarity
2. Took a seasonal difference and read the ACF/PACF to pick model orders by hand
3. Fit my hand-built model and compared it to `auto.arima()`
4. Held out 1990 as a test year to decide which model forecasts better
5. Checked the residuals of the final model (ACF, histogram, Ljung-Box)
6. Forecast all 12 months of 1991
7. Ran periodograms on the raw series and on the model residuals

## Results

Box-Jenkins identification pointed me to **SARIMA(0,1,1)(1,1,1)[12]**. The automatic search picked **SARIMA(2,0,0)(0,1,1)[12]** instead. The two models difference the data differently, so their AIC values can't be compared fairly. I settled it with a forecast test instead: train both on 1981–1989, then predict 1990.

| Model | RMSE | MAE | MAPE |
|---|---|---|---|
| **SARIMA(2,0,0)(0,1,1)[12]** | **0.716** | **0.522** | **4.84%** |
| SARIMA(0,1,1)(1,1,1)[12] | 0.900 | 0.717 | 6.67% |

The auto model won with about a fifth less error. My extra regular difference was hurting more than helping. The series was already stationary, and the MA coefficient near −0.78 was a hint I'd over-differenced.

For the final model:

- **Residuals look like white noise.** The Ljung-Box p-value is 0.72.
- **The 1991 forecast looks realistic.** It shows about 15°C in Jan/Feb and about 7°C in Jun/Jul. Remember, it's the southern hemisphere, so January is summer.
- **The spectral check passes.** The raw periodogram has a huge spike at 1 cycle/year, which is the 12-month season. After fitting the model, that spike is gone from the residuals.

## Tools

R with `forecast`, `tseries`, `tidyverse`, and `lubridate`

## Repo contents

- `274-final.pdf` is the full report, with plots, output, and code in the appendix
- `daily-minimum-temperatures-in-me.csv` is the dataset

## What I'd do next

- Use rolling-origin cross-validation instead of relying on a single test year
- Model the daily series directly instead of monthly averages
- Add more recent Bureau of Meteorology data to see whether the seasonal pattern has shifted over time
