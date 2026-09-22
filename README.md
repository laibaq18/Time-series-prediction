<!-- In this project, we explore the application of the Dual-Stage Attention-Based Recurrent Neural Network (DA-RNN) [1] to predict precipitation levels using the ERA5 dataset  [2]. The DA-RNN architecture is specifically designed to address two major limitations of classical time series models: (i) their inability to select the most relevant input features, and (ii) their weakness in capturing long-range temporal dependencies, which are critical for modeling complex weather patterns.

The DA-RNN model has already demonstrated state-of-the-art performance on financial (NASDAQ 100 Stock dataset) and environmental datasets (SML 2010 dataset) by capturing long-term dependencies and filtering relevant features through input and temporal attention mechanisms. Thus, this makes DA-RNN suitable for multi-variate, exogenous time series data such as ERA5, where the goal is to predict a target variable (precipitation) based on multiple atmospheric drivers.

For methodology & results, please refer to the Report. -->

<div align="center">

# 🌧️ Precipitation Forecasting with DA-RNN
### Dual-Stage Attention RNN vs. LSTM vs. Linear Regression on ERA5 Reanalysis Data

![Python](https://img.shields.io/badge/-Python-3776AB?style=flat&logo=python&logoColor=white)
![Jupyter](https://img.shields.io/badge/-Jupyter-F37626?style=flat&logo=jupyter&logoColor=white)
![Time Series](https://img.shields.io/badge/-Time%20Series-4B8BBE?style=flat)
![Status](https://img.shields.io/badge/status-active-brightgreen)

*Forecasting hourly precipitation in Munich from 2 years of atmospheric reanalysis data and testing whether attention actually earns its keep.*

</div>

---

## 🎯 Overview

Precipitation is hard to forecast: it's driven by many interacting atmospheric variables and has long-range temporal dependencies that classical models (ARMA, ARIMA, NARX) can't capture well. 
In this project, we explore the application of the Dual-Stage Attention-Based Recurrent Neural Network (DA-RNN) [1] to predict precipitation levels using the ERA5 dataset  [2]. The DA-RNN architecture is specifically designed to address two major limitations of classical time series models: (i) their inability to select the most relevant input features, and (ii) their weakness in capturing long-range temporal dependencies, which are critical for modeling complex weather patterns.

The DA-RNN model has already demonstrated state-of-the-art performance on financial (NASDAQ 100 Stock dataset) and environmental datasets (SML 2010 dataset) by capturing long-term dependencies and filtering relevant features through input and temporal attention mechanisms. Thus, this makes DA-RNN suitable for multi-variate, exogenous time series data such as ERA5, where the goal is to predict a target variable (precipitation) based on multiple atmospheric drivers.

So this project implements the **Dual-Stage Attention-Based RNN (DA-RNN)** - model with two attention stages: Input & Temporal attention ...and benchmarks it against an **LSTM** and a **Linear Regression** baseline to check whether the extra complexity is actually worth it.

> 📄 Full write-up: [`Project-Report.pdf`](assets/Project-Report.pdf) (literature review, methodology, references)

---

## Models

<details>
<summary><strong>DA-RNN — Dual-Stage Attention RNN</strong></summary>

```
Driving variables [T=10h, 10 features + 4 cyclical time features]
  → Encoder (RNN) with Input Attention
       (learns which of the 10 driving variables matter most, per timestep)
  → Encoder hidden states [T, hidden_dim]
  → Decoder with Temporal Attention
       (weights encoder states + historical target values y_history)
  → Next-hour precipitation forecast
```
Reference: Qin et al., *"A Dual-Stage Attention-Based Recurrent Neural Network for Time Series Prediction,"* 2017 - [arXiv:1704.02971](https://arxiv.org/pdf/1704.02971).
</details>

<details>
<summary><strong>LSTM (baseline)</strong></summary>

```
Input window [T=10h, n_features] → single-layer LSTM (64 hidden units)
  → final hidden state → Fully Connected layer → next-hour forecast
```
No explicit attention and no separate historical-target input - the point of comparison for what DA-RNN's attention actually buys you.
</details>

<details>
<summary><strong>Linear Regression (baseline)</strong></summary>

```
Input window [T=10h, n_features] → flattened to one vector → linear layer → next-hour forecast
```
</details>

---

## Dataset

**Source:** [ERA5 reanalysis](https://cds.climate.copernicus.eu/datasets/reanalysis-era5-single-levels?tab=overview) hourly data, pulled via the Copernicus Climate Data Store API, **Jan 2023 – Dec 2024**, fixed to a single point (**Munich, Germany**) to turn ERA5's gridded output into a clean multivariate time series.

**10 driving variables:** evaporation (`e`), surface evaporation stress (`es`), runoff from rainfall (`avg_rorwe`), soil evaporation (`avg_esrwe`), wind components (`u10`, `v10`), 2m temperature (`t2m`), mean sea-level pressure (`msl`), surface pressure (`sp`), total cloud cover (`tcc`) — plus **4 engineered cyclical time features** (`hour_sin/cos`, `doy_sin/cos`) to encode daily/seasonal periodicity.

**Target:** total precipitation (`tp`), predicted one hour ahead from a **10-hour sliding window**.

**Split:** 14,035 train / 1,754 validation / 1,754 test hourly samples.

---

## Training Setup

| | |
|---|---|
| Optimizer | Adam |
| Learning rate | 0.001 |
| Batch size | 32 |
| Epochs | 100 |
| Window size (T) | 10 hours → predict next hour |

---

## 📊 Results

Test-set performance (lower is better):

| Model | MAE | RMSE |
|---|:---:|:---:|
| Linear Regression | 0.000208 | 0.000248 |
| LSTM | 0.000097 | 0.000179 |
| **DA-RNN** | **0.000033** | **0.000082** |
| Persistence baseline *(repeat last value)* | 0.005128 | 0.016370 |

**Key takeaways:**
- DA-RNN's dual attention mechanism cuts RMSE by **~54% vs. LSTM** and **~67% vs. linear regression** thus the attention is earning its complexity, not just adding parameters.
- All three trained models beat the naive persistence baseline by **~200×**, confirming they're learning real atmospheric signal rather than just autocorrelation.
- In recursive 24-hour multi-step mode, both DA-RNN and LSTM correctly forecast zero precipitation for Jan 1, 2025 - which matches the true ERA5 record for that day, a good sanity check that the multi-step forecasts stay physically plausible instead of degenerating.

---

## References
[1] Qin, Y., Song, D., Chen, H., Cheng, W., Jiang, G., & Cottrell, G. (2017). A Dual-Stage Attention-Based Recurrent Neural Network for Time Series Prediction. arXiv preprint arXiv:1704.02971. https://arxiv.org/pdf/1704.02971

[2] ERA5 Climate Reanalysis Dataset. Copernicus Climate Data Store. https://cds.climate.copernicus.eu/datasets/reanalysis-era5-single-levels?tab=overview
