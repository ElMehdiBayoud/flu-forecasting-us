# Forecasting Seasonal Influenza Activity in US HHS Regions

GROUP PROJECT

We build a **real-life-like forecasting pipeline** using public health surveillance data from the **CDC FluView** system, focusing on **weekly influenza-like illness (ILI) activity** across US HHS regions.

---

## 1. Problem & Motivation

Seasonal influenza places recurring pressure on healthcare systems.  
Public health authorities need **short-term forecasts of regional flu activity** to:

- Allocate hospital staff and ICU capacity.
- Plan communication campaigns and vaccination pushes.
- Coordinate resources between regions.

We use historical **ILI surveillance** and **lab-confirmed influenza** data to forecast:

> **Weekly ILI percentage (`ILI_PERCENT`) 1–4 weeks ahead for all US HHS regions.**

---

## 2. Data

**Source:** CDC FluView – ILI and NREVSS (lab) data  
Exported via the FluView web portal and merged into `flu_merged.csv`.

- **Frequency:** Weekly  
- **Horizon:** ~1997–2025  
- **Spatial unit:** HHS Regions (Region 1–10)  
- **Key variables:**
  - `ILI_PERCENT`: % of outpatient visits due to influenza-like illness  
  - `PERCENT_POSITIVE`: % of lab tests positive for influenza  
  - `TOTAL_SPECIMENS`, `TOTAL_A`, `TOTAL_B`  
  - Age-group ILI counts, number of providers, total patients  
  - Time variables: `YEAR`, `WEEK`, `DATE`, `WEEKOFYEAR`

Raw data are stored under `data/raw/` (not tracked on GitHub for size/privacy reasons).  
After cleaning and feature engineering we create a panel dataset saved as:

- `data/processed/flu_panel_clean.csv`

---

## 3. Forecasting Task

**Target variable:**

- `ILI_PERCENT` (per `REGION`, per `DATE`)

**Unit of analysis:**

- One observation = (Region, Week).

**Forecast horizons:**

- 1 week ahead (very short-term)
- 4 weeks ahead (short-term planning)

**Problem type:**

- Time series forecasting with a **panel structure** (multiple regions).
- Univariate target with **exogenous features** (lab positivity, seasonal indicators, region ID, lag features).

---

## 4. Methods

We follow the course structure and implement:

### 4.1 Baselines

- **Naive:** predict last observed ILI value per region.
- **Seasonal Naive:** use ILI from the same week one year earlier (lag-52).

### 4.2 Classical Models

- **Exponential Smoothing / Holt–Winters** (on selected regions).
- **SARIMA / SARIMAX** with weekly seasonality (period = 52),
  fitted separately per region.

### 4.3 Advanced Model (Panel ML)

A **global machine learning model** trained across all regions:

- Features:
  - Region ID (categorical → numeric)
  - Week-of-year, sine/cosine seasonal encodings
  - Lab positivity (`PERCENT_POSITIVE`, `TOTAL_SPECIMENS`)
  - Lagged ILI values: 1, 2, 3, 4, 8, 12, 26, 52 weeks
  - Lagged lab positivity: 1, 2, 3, 4, 52 weeks
- Models:
  - **RandomForestRegressor**
  - Optionally **LightGBM** for faster and more flexible boosting

This satisfies the “advanced method” requirement (ensemble/ML model on time-series features).

---

## 5. Evaluation Setup

We use a **time-based split**:

- Train: up to ~2016-06-30  
- Validation: 2016-07-01 – 2020-06-30  
- Test: from 2020-07-01 onwards

For SARIMA we fit per-region using train+validation and forecast on the test period.  
For the global RF/LGBM model, we train on panel data from all regions (train+validation) and test on all regions jointly.

**Metrics:**

- RMSE
- MAE
- MAPE (reported with caution during low-activity weeks)

Metrics are reported:

- **Globally:** averaged over all regions.
- **Per region:** to see which regions are easier/harder to predict.

---

## 6. Repository Structure

```text
.
├─ data/
│  ├─ raw/              # raw FluView exports (not tracked)
│  └─ processed/
│     └─ flu_panel_clean.csv
├─ notebooks/
│  ├─ 01_eda_flu.ipynb
│  ├─ 02_feature_engineering_splits.ipynb
│  ├─ 03_baselines_and_sarima.ipynb
│  ├─ 04_global_ml_panel_model.ipynb
│  └─ 05_evaluation_and_plots.ipynb
├─ src/
│  ├─ data_prep.py
│  ├─ features.py
│  ├─ models.py
│  ├─ metrics.py
│  └─ plotting.py
├─ reports/
│  ├─ figures/
│  └─ group_report.pdf
├─ README.md
├─ requirements.txt
└─ .gitignore

```

## 7. Getting Started

1.  **Clone the repository:**
    ```bash
    git clone https://github.com/ElMehdiBayoud/flu-forecasting.git
cd flu-forecasting
    git clone [https://github.com/ElMehdiBayoud/drug-launch-analytics.git](https://github.com/ElMehdiBayoud/flu-forecasting-us)
    ```
2.  **Install dependencies:**
    ```bash
    pip install -r requirements.txt
    ```
3.  **Data Access:**
    * This project uses public **CDC FluView** data.
4.  **Run the Notebook:**
    * Open `notebooks/flu-forecasting.ipynb` and run all cells.
