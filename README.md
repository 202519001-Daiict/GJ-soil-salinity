# 🌾 Gujarat Soil Salinity Intelligence Platform

**A GeoAI decision-support platform that monitors, forecasts, and maps soil salinity across all 33 districts of Gujarat using satellite-derived indicators, machine learning, and deep learning forecasting.**

---

## 📌 Project Summary

Soil salinity — driven by seawater intrusion, poor drainage, and groundwater over-extraction — is one of the biggest threats to agricultural land in Gujarat's coastal, canal, and inland zones. This platform turns 10 years of environmental data (2015–2025) into a district-level **Soil Salinity Prediction Index (SSPI)**, forecasts it forward to 2030, compares 4 predictive models, and serves everything through an interactive Streamlit dashboard with a live GIS map.

**In short:** raw climate + vegetation data → engineered features → SSPI score per district per year → 4 competing ML/DL models forecast future salinity → results are stored, mapped, and served in a decision-support dashboard with automated policy recommendations.

| | |
|---|---|
| **Study Area** | All 33 districts of Gujarat, India (Coastal, Canal, Inland, Hilly zones) |
| **Time Range** | Historical: 2015–2025 · Forecast: 2026–2030 |
| **Core Metric** | Soil Salinity Prediction Index (SSPI), 0–100 scale |
| **Inputs** | NDVI (vegetation), annual rainfall, Rabi-season temperature, district zone type |
| **Models Compared** | Linear Regression, Random Forest, XGBoost, Temporal Fusion Transformer (TFT) |
| **Best Model** | 🏆 TFT Transformer — **R² 0.831**, MAE 4.91, RMSE 7.25 |
| **Interface** | Streamlit dashboard with interactive Folium GIS map, trends, forecasts, and model comparison |

---

## 📸 Dashboard Preview

<img src="outputs/dashboard_screenshots/01_home_dashboard.png" width="700">

Home view — overall system accuracy, best model, and a live district-level SSPI map of Gujarat, color-coded by risk class (Low / Moderate / High / Critical).

---

## 🌍 Study Area & Zones

The platform segments Gujarat's 33 districts into four salinity-relevant zones, each with distinct risk drivers:

| Zone | Primary Salinity Driver |
|---|---|
| **Coastal** | Seawater intrusion, tidal influence |
| **Canal** | Waterlogging, secondary salinization from irrigation |
| **Inland** | Groundwater over-extraction, borewell salinity |
| **Hilly** | Naturally low salinity — used as a baseline comparison zone |

---

## ⚙️ Workflow

```
Weather Data (rainfall, temperature)
        │
        ▼
Environmental Variables (NDVI, Rainfall, Temperature)
        │
        ▼
Feature Engineering  (deficits, anomalies, lag features, Mann–Kendall trend test)
        │
        ▼
SSPI Calculation  (district-year salinity index, 0–100)
        │
        ▼
ML Models: Linear Regression → Random Forest → XGBoost
        │
        ▼
Deep Learning: Temporal Fusion Transformer (TFT) forecasting to 2030
        │
        ▼
SQLite Database  (stores history, forecasts, model comparison)
        │
        ▼
GIS Visualization  (GeoPandas + Folium choropleth maps)
        │
        ▼
Streamlit Dashboard  (interactive UI)
        │
        ▼
Decision Support System  (zone-specific mitigation recommendations)
```

The numbered scripts in [`scripts/`](scripts/) implement this pipeline end-to-end, from raw data (`01_setup_spatial.py`) through to validation against CSSRI Bharuch reference data (`12_validate.py`).

---

## 📊 Soil Salinity Prediction Index (SSPI)

A composite 0–100 score is calculated per district per year, blending NDVI deficit, rainfall deficit, and temperature anomaly relative to each district's own historical baseline, weighted by zone-specific risk factors.

| SSPI Range | Category |
|---|---|
| 0 – 24 | Low |
| 25 – 49 | Moderate |
| 50 – 74 | High |
| 75+ | Critical |

<img src="outputs/dashboard_screenshots/02_district_analysis.png" width="700">

District drill-down view — historical SSPI trend, 2030 forecast, and environmental drivers (NDVI, rainfall, temperature) for a single district.

---

## 🧠 Model Comparison

Four models were trained and benchmarked on the same district-year feature set to predict SSPI:

| Model | MAE | RMSE | R² Score |
|---|---|---|---|
| Linear Regression (Ridge) | 7.81 | 9.01 | 0.745 |
| Random Forest | 7.26 | 8.45 | 0.776 |
| XGBoost | 6.65 | 8.53 | 0.771 |
| **Temporal Fusion Transformer (TFT)** | **4.91** | **7.25** | **0.831** |

The TFT — a deep learning architecture built for multi-horizon time-series forecasting — outperformed the classical baselines by learning temporal dependencies (lag features, trend slopes) across all 33 districts jointly, rather than treating each district-year as an independent row.

<img src="outputs/dashboard_screenshots/04_model_comparison.png" width="700">

---

## 🗺️ GIS Mapping & Trends

<img src="outputs/dashboard_screenshots/03_gis_trends_map.png" width="700">

Interactive Folium map layers for current (2025) vs. forecasted (2030) SSPI, plus historical trend charts (SSPI, NDVI, rainfall, temperature) per district and a side-by-side SSPI comparison across all 33 districts.

---

## 🎯 Smart Policy Recommendation Engine

Based on a district's SSPI class and zone type, the dashboard auto-generates mitigation guidance, e.g.:

- **Coastal / Critical:** Coastal protection measures, salt-tolerant crop varieties
- **Canal / High:** Drainage management, reduce irrigation frequency
- **Inland / Moderate:** Monitor groundwater depth/EC quarterly, regulate borewell extraction
- **Any zone / Low:** Routine monitoring, gypsum application if trending upward

---

## 📁 Repository Structure

```
├── app.py                          # Streamlit dashboard entry point — run this
├── data/
│   └── processed/
│       ├── Gujarat_districts.gpkg  # District boundaries (33 districts)
│       ├── master_raw.csv          # Merged raw environmental data
│       ├── features_complete.csv   # Engineered features (lags, deficits, trends)
│       ├── master_with_sspi.csv    # Final data with SSPI scores
│       └── salinity_db.sqlite      # SQLite DB backing the dashboard
├── outputs/
│   ├── dashboard_screenshots/      # Dashboard preview images
│   └── demo/                       # Demo.mp4 — screen recording of the live app
├── scripts/                        # Full pipeline, step 01 → 12, in order
│   ├── 01_setup_spatial.py         # Load & dissolve district boundaries, assign zones
│   ├── 02_weather_download.py      # Download historical rainfall/temperature
│   ├── 03_data_merge.py            # Merge weather + NDVI + spatial data
│   ├── 04_sspi_calc.py             # Compute district-year SSPI
│   ├── 05_features_trend.py        # Lag features + Mann–Kendall trend test
│   ├── 06_ml_prep.py               # Train/test split, scaling
│   ├── 07_linear_regression.py     # Baseline Ridge regression
│   ├── 08_rf_xgb.py                # Random Forest + XGBoost
│   ├── 08b_tft.py                  # Temporal Fusion Transformer (deep learning)
│   ├── 09_predict_future.py        # Forecast SSPI to 2030
│   ├── 10_maps_charts.py           # Generate static forecast maps/charts
│   ├── 10b_map_2025_current.py     # Generate current-year salinity map
│   ├── 11_store_db.py              # Persist everything to SQLite
│   └── 12_validate.py              # Validate SSPI against CSSRI Bharuch ground data
├── requirements.txt
├── LICENSE
└── README.md
```

> Trained model artifacts (`*.pkl`, `*.ckpt`) are not included in the repo (see `.gitignore`) — they're regenerated by running `scripts/07_linear_regression.py`, `scripts/08_rf_xgb.py`, and `scripts/08b_tft.py` in order after the feature pipeline.

---

## 🛠️ Technology Stack

| Category | Tools |
|---|---|
| Data Processing | Pandas, NumPy |
| Geospatial | GeoPandas, Folium, Streamlit-Folium |
| Machine Learning | Scikit-Learn, XGBoost |
| Deep Learning | PyTorch, PyTorch Forecasting, Lightning |
| Statistics | pymannkendall (trend testing) |
| Visualization | Matplotlib, Plotly, Streamlit |
| Database | SQLite |

---

## ⚙️ Installation & Setup

```bash
# Clone the repository
git clone <this-repo-url>
cd gujarat-soil-salinity-intelligence-platform

# Install dependencies
pip install -r requirements.txt

# Launch the dashboard
streamlit run app.py
```

---

## 🔮 Future Enhancements

- Satellite-based salinity estimation (integrate NDSI-style spectral indices directly, rather than relying only on NDVI/climate proxies)
- Real-time weather API integration
- Explainable AI (XAI) for model transparency
- Cloud deployment for public access
- Multi-state expansion beyond Gujarat

---

## 🎥 Demo

A full screen-recording walkthrough of the live dashboard is available at [`outputs/demo/Demo.mp4`](outputs/demo/Demo.mp4).

---

## 👥 Credits

Built as part of a coursework GeoAI/agricultural-intelligence project. Pipeline design, modeling, and dashboard development by the project team.

Data sources: MODIS GEE (NDVI), NASA POWER (climate), CSSRI Bharuch (salinity validation reference).
