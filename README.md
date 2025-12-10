# US Flu Forecasting (HHS Regions)

Simple project to forecast weekly influenza-like illness (ILI) activity in all US HHS regions using CDC FluView data.

## 1. Goal
Predict **ILI_PERCENT** 1–4 weeks ahead to support basic public health planning.

## 2. Data
Source: **CDC FluView** (ILI + lab data)  
Weekly data, ~1997–2025, HHS Regions 1–10.

Main fields: ILI_PERCENT, PERCENT_POSITIVE, TOTAL_SPECIMENS, YEAR, WEEK, DATE.  
Final cleaned panel:

```
data/processed/flu_panel_clean.csv
```

## 3. Models
- **Naive**
- **Seasonal Naive (lag-52)**
- **Holt–Winters**
- **SARIMA / SARIMAX** per region
- **Random Forest** (global panel model)

Features: region ID, week-of-year, seasonal sine/cosine, lab positivity, lagged ILI values.

## 4. Evaluation
Time-based split:
- Train: until ~2016  
- Val: 2016–2020  
- Test: 2020+

Metrics: RMSE, MAE, MAPE.  
Results reported overall + per region.

## 5. Repository Structure
```
.
├─ data/
│  ├─ raw/
│  └─ processed/flu_panel_clean.csv
├─ notebook/notebooks/flu-forecasting.ipynb
├─ reports/
│  ├─ figures/
│  ├─ metrics/
│  └─ group_report.pdf
├─ requirements.txt
└─ README.md
```

## 6. How to Run
Clone:
```bash
git clone https://github.com/ElMehdiBayoud/flu-forecasting-us.git
```

Install:
```bash
pip install -r requirements.txt
```

Run notebook:
```
notebook/notebooks/flu-forecasting.ipynb
```
