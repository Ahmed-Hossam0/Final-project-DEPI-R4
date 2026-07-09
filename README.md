# 🚦 Traffic Accident Analytics

> *Decoding the Roads — From Raw Crash Records to Public-Safety Insight*

## 📋 Overview

This is a **comprehensive data analytics graduation project** developed as part of the **Digital Egypt Pioneers Initiative (DEPI) — Data Analysis Track**. It analyzes a large, real-world traffic crash dataset covering over **209,000 accidents** to uncover the environmental, behavioral, and structural conditions most associated with accident frequency and injury severity.

The full pipeline was built entirely in **Python (pandas / NumPy)**, following the **Medallion Architecture (Bronze → Silver → Gold)**, and culminates in an interactive **Power BI dashboard** — first prototyped in **Figma** — designed to surface actionable insights for public-safety stakeholders, city planners, and the general public.

**Key Focus Areas:**

- Accident frequency by road, weather, lighting, and time conditions
- Injury severity analysis (Fatal, Major, Moderate, Minor, No Injury)
- Road surface & defect impact on crash outcomes
- Contributory cause & crash-type analysis
- **AI Insights — 24-month Accident Volume & Injury Rate Forecast (XGBoost)**

---

## 👥 Project Team

| Name |
|---|
| Ahmed Hossam Mohamed Nabil (Team Leader) |
| Fajr Abdullah |
| Youssef Amr Abdelhamid |
| Basmala Sabri |
| Eslam Jabr |
| Salma Adel |

---

## 📊 Dataset

### Traffic Accidents Dataset

| Attribute | Detail |
|---|---|
| **Dataset Name** | Traffic Accidents |
| **Source Platform** | Kaggle |
| **Publisher** | oktayrdeki |
| **Dataset URL** | [kaggle.com/datasets/oktayrdeki/traffic-accidents](https://www.kaggle.com/datasets/oktayrdeki/traffic-accidents) |
| **Data Type** | Real-world, police-reported crash records |
| **Total Records** | 209,306 rows (full dataset used) |
| **Date Range** | March 2013 – January 2025 (≈11.9 years) |
| **File Format** | CSV |
| **License** | Open / Public (Kaggle) |

The raw dataset contains **24 columns** spanning four thematic domains:

- **Crash Context** — date, traffic control device, weather, lighting, trafficway type, alignment
- **Crash Classification** — crash type, first crash type, contributory cause, road surface, road defect
- **Injury Outcomes** — total/fatal/incapacitating/non-incapacitating injuries, most severe injury
- **Time Attributes** — crash hour, day of week, month

---

## 🛠️ Tools & Technologies

| Component | Technology | Purpose |
|---|---|---|
| **Data Engineering** | Python (pandas, NumPy) | End-to-end Medallion pipeline |
| **Data Architecture** | Medallion Architecture (Bronze → Silver → Gold) | Structured, scalable data pipeline |
| **Exploratory Analysis** | Python (Plotly Express) | Data-quality checks, distributions, ranking, measures dictionary |
| **Machine Learning** | XGBoost, Prophet | Accident volume & injury rate forecasting, category prediction |
| **Visualization** | Power BI | Interactive 7-page dashboard |
| **Data Modeling** | Star Schema (1 Fact + 8 Dimensions) | Optimized relational design |
| **Dashboard Design** | Figma ("Vigilant Analytics" brand system) | UI/UX wireframing & layout planning |

---

## 🔄 Architecture & Steps

### Step 1: Data Pipeline (Medallion Architecture)

```
Bronze Layer → Silver Layer → Gold Layer
  (Raw)         (Cleaned)      (Refined)
```

#### 🥉 Bronze Layer — 🥉 Bronze Layer — [Bronze_layer.ipynb](./Medallion%20architecture/Bronze_layer/Bronze_layer.ipynb)



- Raw CSV (209,306 rows × 24 columns) read directly into a pandas DataFrame
- Profiled with `.info()` / `.describe()` — confirmed **zero missing values** across all fields
- Saved untouched as `Bronze.csv` — a permanent, unmodified archive of the source data

```python
Bronze_df = pd.read_csv("/content/sample_data/traffic_accidents.csv")
Bronze_df.info()
Bronze_df.describe()
Bronze_df.to_csv("/content/sample_data/Bronze.csv", index=False)
```

#### 🥈 Silver Layer — [[Silver Layer](./Medallion%20architecture/Silver_Layer.ipynb)](https://github.com/Ahmed-Hossam0/Final-project-DEPI-R4/blob/main/Medallion%20architecture/Silver_layer/Silver_Layer.ipynb
)


Twelve sequential cleaning & enrichment steps applied — **zero rows dropped**, every row of the 209,306 preserved:

| # | Issue / Task | Correction Applied |
|---|---|---|
| 1 | `crash_date` stored as text; categorical columns risk mixed dtypes | Parsed to datetime; text columns cast to string |
| 2 | Y/N shorthand in intersection flag | Replaced with YES / NO |
| 3–5 | Time-of-day information embedded in the timestamp | Extracted AM/PM period; reduced `crash_date` to date-only |
| 6 | Raw police-report column names not analysis-friendly | Renamed 10 columns to clear business labels |
| 7 | No single severity indicator existed | Engineered `Crash_severity` |
| 8 | Column order inconsistent with reporting needs | Reordered all 26 columns logically |
| 9 | `crash_hour` alone hard to group by daypart | Engineered `Time Buckets` |
| 10 | Duplicate label variant in `trafficway_type` | Standardized `UNKNOWN INTERSECTION TYPE` → `UNKNOWN` |
| 11 | `Cost_of_damage` bracket had no numeric severity proxy | Engineered `Damage_level` (1–3) |
| 12 | `crash_date` reverted to object dtype after `.dt.date` | Re-cast to datetime before export |

Output saved as `Silver.csv`.

#### 🥇 Gold Layer — [[Gold Layer](./Medallion%20architecture/Gold_Layer.ipynb)](https://github.com/Ahmed-Hossam0/Final-project-DEPI-R4/blob/main/Medallion%20architecture/Gold_layer/Gold_Layer.ipynb
)

- Enriched Silver DataFrame split into **8 dimension tables + 1 fact table**
- Each dimension assigned a sequential surrogate key, joined back onto the fact rows
- All tables exported as flat CSV files, ready for direct Power BI import

```python
fact_accidents = fact[[
    "Date_Key", "Weather_Key", "Lighting_Key", "Crash_Type_Key",
    "Road_Key", "Time_Key", "Damage_Key", "Severity_Key",
    "Number_of_vehicles_involved", "Total_injuries_num", "Fatal_injuries_num",
    "injuries_incapacitating", "injuries_non_incapacitating", "injuries_not_visibily_evident"
]]
fact_accidents.insert(0, "Accident_Key", range(1, len(fact_accidents) + 1))
```

#### 📈 EDA & AI Insights — [[`EDA.ipynb`](./EDA.ipynb)](https://github.com/Ahmed-Hossam0/Final-project-DEPI-R4/blob/main/Exploratory_data_analysis/EDA.ipynb)

- Reads all 9 Gold-layer CSVs, runs systematic null/duplicate checks (all passed — 0 nulls, 0 duplicates)
- Rebuilds the denormalized master table by merging `fact_accidents` with all 8 dimensions
- Validates the full measures dictionary, builds every page-by-page chart later implemented in Power BI
- Trains and compares **XGBoost vs. Prophet** for accident-volume and injury-rate forecasting
- Trains lightweight XGBoost classifiers for severity-by-weather and damage-by-road-surface prediction

---

### Step 2: Data Modeling (Star Schema)

The Gold layer implements one central **Fact Table** joined to eight **Dimension Tables**, each connected via a surrogate key generated during the Gold-layer build:

```
                              ┌────────────────────┐
                              │  fact_accidents     │
                    ┌─────────│  (Accident_Key +    │─────────┐
                    │         │   7 FKs + measures) │         │
                    ▼         └──────────┬──────────┘         ▼
              dim_date                   │                dim_weather
          (Year, Month,                  │               (12 conditions)
           Quarter, Day)                 │
                    ┌─────────────────────┼─────────────────────┐
                    ▼                     ▼                     ▼
             dim_lighting            dim_road               dim_time
          (lighting cond.)      (surface condition)   (hour, AM/PM, Time Buckets)
                    │                     │                     │
                    ▼                     ▼                     ▼
            dim_crash_type           dim_damage             dim_severity
          (Injury/Tow vs.       (Cost bracket +          (Crash_severity:
           No-Injury/Drive)      Damage_level 1–3)      Fatal/Major/.../No Injury)
```

| Table | Type | Key Column(s) | Description |
|---|---|---|---|
| `fact_accidents` | Fact | `Accident_Key` + 7 FKs | One row per crash; holds all foreign keys plus numeric injury/vehicle measures |
| `dim_date` | Dimension | `Date_Key` | Date, Year, Month, Month_Name, Day, Quarter, Day_Of_Week |
| `dim_weather` | Dimension | `Weather_Key` | Distinct weather conditions at time of crash |
| `dim_lighting` | Dimension | `Lighting_Key` | Distinct lighting conditions at time of crash |
| `dim_crash_type` | Dimension | `Crash_Type_Key` | Injury/Tow vs. No-Injury/Drive-Away classification |
| `dim_road` | Dimension | `Road_Key` | Distinct roadway surface conditions |
| `dim_time` | Dimension | `Time_Key` | Crash hour, AM/PM flag, engineered Time Buckets |
| `dim_damage` | Dimension | `Damage_Key` | Cost-of-damage bracket + engineered Damage_level (1–3) |
| `dim_severity` | Dimension | `Severity_Key` | Engineered Crash_severity category |

---

### Step 3: Feature Engineering

Three composite columns were engineered in the Silver layer to capture analytical categories not observable from any single raw field:

| Feature | Description | Logic |
|---|---|---|
| **Crash_severity** | Priority-cascade injury severity label | Fatal → Major → Moderate → Minor → No Injury (`np.select`, first true condition wins) |
| **Time Buckets** | Daypart grouping from `crash_hour` | Late Night (00–05) │ Morning (06–11) │ Afternoon (12–17) │ Evening (18–21) │ Night (22–23) |
| **Damage_level** | Numeric 1–3 proxy for `Cost_of_damage` | 1 = ≤$500 │ 2 = $501–$1,500 │ 3 = >$1,500 |
| **AM/PM** | Half-day period extracted from timestamp | Derived directly from `crash_date` before reducing it to date-only |

```python
Silver_df["Crash_severity"] = np.select(
    [
        Silver_df["Fatal_injuries_num"] > 0,
        Silver_df["injuries_incapacitating"] > 0,
        Silver_df["injuries_non_incapacitating"] > 0,
        Silver_df["Total_injuries_num"] > 0
    ],
    ["Fatal", "Major", "Moderate", "Minor"],
    default="No Injury"
)
```

---

### Step 4: Machine Learning (AI Insights)

Four modeling experiments were run on the denormalized master table (built from `fact_accidents` + all 8 dimensions):

| # | Experiment | Target | Type | Outcome |
|---|---|---|---|---|
| 1 | Accident Volume Forecast | Monthly accident count (24 months ahead) | Time-series regression | **XGBoost selected** — R² = 0.261, MAPE = 11.27% |
| 2 | Injury Rate Forecast | Monthly injuries ÷ accidents (24 months ahead) | Time-series regression | XGBoost with lag/rolling features |
| 3 | Crash Severity by Weather | Most likely severity per weather condition | Classification | Majority-class result ("No Injury" for all — honestly reported, see below) |
| 4 | Damage Level by Road Surface | Most likely damage tier per surface condition | Classification | Majority-class result ("Over $1,500" for all — honestly reported, see below) |

#### Model Comparison — XGBoost vs. Prophet (Accident Volume Forecast)

Both models were trained on identical monthly-aggregated data and evaluated on the same held-out **2024 hold-out year** (strict time-based split, no shuffling):

| Model | R² | MAE | RMSE | MAPE | Selected |
|---|---|---|---|---|---|
| **XGBoost** | **0.261** | **160.3** | **299.3** | **11.27%** | ✅ **Best** |
| Prophet | -0.020 | 185.8 | 351.4 | 13.36% | |

**Why XGBoost?**
- Explains a meaningful share of month-to-month variance (R² = 0.261); Prophet's negative R² performs worse than predicting the historical mean
- Lower error across every metric (MAE, RMSE, MAPE)
- Engineered lag (1/2/3/12-month) and rolling-average (3/12-month) features let it react to recent momentum, where Prophet relies only on a smooth yearly-seasonality curve

```python
xgb_model = xgb.XGBRegressor(
    n_estimators=300, max_depth=3, learning_rate=0.05,
    subsample=0.8, colsample_bytree=0.8, random_state=42
)
```

#### Accident Volume Forecast (24 Months)

| Period | Forecasted Accidents | Notable Detail |
|---|---|---|
| Year 1 (Feb 2025 – Jan 2026) | 17,599 | Peaks in Jun–Jul (~1,600–1,624/mo); lowest in Feb (1,176) |
| Year 2 (Feb 2026 – Jan 2027) | 17,580 | Peaks in Jun–Sep (~1,545–1,577/mo); lowest in Feb (1,311) |
| **Total (24 Months)** | **35,179** | Stable year-over-year — no strong upward/downward trend |

#### Injury Rate Forecast

| Period | Injury Rate Range | Pattern |
|---|---|---|
| 2025 | 0.36 (Jan/Feb) → 0.49 (Jul) | Rises through spring, peaks in summer, declines toward year-end |
| 2026 | 0.35 (Feb) → 0.49 (Jul) | Repeats the same shape one year later — stable seasonal cycle |

#### Transparency Note — Category Classifiers

Both the severity-by-weather and damage-by-road-surface classifiers were trained on a **single categorical predictor**. With only one input feature and a heavily imbalanced target (74% of accidents are "No Injury"; 70% fall in the "Over $1,500" damage tier), each model converges to predicting the single majority class for every input category. This is reported here as an **honest, expected outcome of majority-class dominance** rather than a modeling error — the volume columns (`Predicted_Crash_Count`) remain the real analytical payload for the dashboard, not the single predicted label. A more discriminative version would require additional predictors (e.g. lighting, time bucket, vehicle count).

---

## 📈 Power BI Dashboard (7 Pages)

The dashboard connects to the Gold-layer CSV tables plus the consolidated ML output workbook, following a deliberate data-storytelling structure: landing page → executive overview → drill-downs → forward-looking AI Insights.

### Landing Page — "Public Safety Portal"
Full-bleed aerial highway photograph, centered title card, and navigation buttons routing to each analytical page. No KPIs or charts — pure entry point.

### Page 1: Dashboard Overview
High-level snapshot — Total Accidents, Severity Index, Injury Totals; yearly accident trend line, severity donut, road-surface treemap.

### Page 2: Road Condition Analysis
Road surface & defect impact — grouped bar of accidents vs. severe injuries by surface, crash-severity stacked bar, road-defect bar chart.

### Page 3: Weather Condition Analysis
Weather-related accident & fatal-injury totals; bar charts by weather condition; weather-vs-severity stacked bar.

### Page 4: Severity Analysis
Fatal, incapacitating & severe-injury counts; fatal-injuries-over-years line chart; fatal injuries by weather/lighting; injuries-by-severity donut.

### Page 5: Crash Cause Analysis
Total accidents & crash-hour peaks; dual-line chart of accidents/injuries by hour; ranked bars by weather, trafficway type, lighting condition.

### Page 6: AI Insights
Forward-looking capstone — Total Predicted Accidents (2026–2027) and Injury Prediction Accuracy KPI cards; predicted-volume stacked bars by weather/road surface; 24-month accident-count and injury-rate forecast lines.

---

## 📐 Key KPIs

### Accident Volume
| KPI | Value | Derivation |
|---|---|---|
| Total Accidents | 209,306 | `COUNTROWS(fact_accidents)` |
| Total Vehicles Involved | 431,861 | `SUM(Number_of_vehicles_involved)` |
| Avg. Vehicles per Accident | 2.06 | Total Vehicles / Total Accidents |
| Date Range Covered | Mar 2013 – Jan 2025 (~11.9 yrs) | `MAX(Date) − MIN(Date)` in `dim_date` |

### Injury & Severity
| KPI | Value | Derivation |
|---|---|---|
| Total Injuries | 80,105 | `SUM(Total_injuries_num)` |
| Avg. Injuries per Accident | 0.38 | Total Injuries / Total Accidents |
| Total Fatal Injuries | 389 | `SUM(Fatal_injuries_num)` |
| Incapacitating Injuries | 7,975 | `SUM(injuries_incapacitating)` |
| Non-Incapacitating Injuries | 46,307 | `SUM(injuries_non_incapacitating)` |
| Severe Injuries | 8,364 | Fatal + Incapacitating |
| Severity Rate | 10.44% | Severe Injuries / Total Injuries |
| Severity Index | 0.6857 | Weighted injury score / Total Accidents |
| Accidents With ≥1 Fatality | 351 (0.17%) | `COUNT` where Fatal Injuries > 0 |

### Road, Weather & Damage
| KPI | Value | Derivation |
|---|---|---|
| Accidents on Clear Weather | 164,700 (78.7%) | `COUNT` where Weather = Clear |
| Accidents on Wet/Icy/Snow Roads | 40,414 (19.3%) | `COUNT` where Road Surface ∈ {Wet, Ice, Snow/Slush} |
| Accidents in Daylight | 134,109 (64.1%) | `COUNT` where Lighting = Daylight |
| Accidents in Darkness (any) | 60,814 (29.1%) | `COUNT` where Lighting ∈ {Darkness, Darkness-Lighted Road} |
| High-Damage Accidents (>$1,500) | 147,313 (70.4%) | `COUNT` where Damage_level = 3 |
| Injury/Tow Crash Type Share | 91,930 (43.9%) | `COUNT` where Crash_Type = Injury and/or Tow |

### AI Insights
| KPI | Value |
|---|---|
| Total Predicted Accidents (2026–2027) | 35,179 |
| Injury Prediction Accuracy | 89% |

---

## 🔬 SMART Research Questions

| # | Question | KPI / Measure |
|---|---|---|
| 1 | Does the Severity Index vary meaningfully by lighting condition, and are darkness-related crashes measurably more severe than daylight crashes? | Severity Index by Lighting |
| 2 | What % of accidents occur on wet, icy, or snow/slush road surfaces, and do these conditions produce a higher Severity Rate than dry roads? | Severity Rate, Road Surface % |
| 3 | Do specific days of week (Friday, Thursday, Saturday) and months (October, September, December) show accident volumes significantly above the yearly average? | Total Accidents by Day / Month |
| 4 | Is there a measurable relationship between the estimated cost-of-damage bracket and injury severity? | Damage_level, Severe Injuries |
| 5 | Which primary contributory causes account for the largest share of accidents, and do top causes differ between injury/tow and no-injury/drive-away crashes? | Total Accidents by Contributory Cause, Crash Type |

---

## 📁 Project File Structure

```
Traffic_Accident_Analytics/
├── Medallion_Architecture/
│   ├── Bronze_layer.ipynb          # Raw ingestion → Bronze.csv (no transformation)
│   ├── Silver_Layer.ipynb          # 12-step cleaning + feature engineering → Silver.csv
│   └── Gold_Layer.ipynb            # Star schema build → fact_accidents.csv + 8 dim_*.csv
├── EDA.ipynb                       # Data quality checks, measures dictionary,
│                                    # page-by-page charts, XGBoost/Prophet forecasting,
│                                    # category prediction, AI Insights consolidation
├── Traffic_Accident_Analytics_Documentation.pdf   # Full graduation project documentation
└── README.md
```

---

## 🚀 How to Run

### Prerequisites

- Python 3.x environment (Jupyter / Google Colab)
- Python libraries: `pandas`, `numpy`, `plotly`, `xgboost`, `prophet`, `scikit-learn`

### Execution Steps

#### Step 1: Run Bronze Layer
```bash
Medallion_Architecture/Bronze_layer.ipynb
```
Reads the raw Kaggle CSV, profiles it, and saves the untouched archive as `Bronze.csv`.

#### Step 2: Run Silver Layer
```bash
Medallion_Architecture/Silver_Layer.ipynb
```
Applies 12 cleaning/standardization steps and engineers `Crash_severity`, `Time Buckets`, and `Damage_level`. Outputs `Silver.csv`.

#### Step 3: Run Gold Layer
```bash
Medallion_Architecture/Gold_Layer.ipynb
```
Builds the star schema (8 dimensions + 1 fact) and exports all tables as CSV.

#### Step 4: Run EDA & AI Insights
```bash
EDA.ipynb
```
Validates data quality, rebuilds the master table, produces every dashboard-page chart, trains the XGBoost/Prophet forecasts, and exports the consolidated ML workbook for Power BI.

#### Step 5: Open Power BI Dashboard
```
# Connect Power BI to the Gold-layer CSV tables + the consolidated ML output workbook
# Open the .pbix file and refresh the data source
```

---

## 📊 Key Findings & Insights

### Accident Patterns
- **Afternoon** (12:00–17:59) accounts for the largest single share of accidents at 40.6%, and the PM half of the day (65.4%) outweighs the AM half (34.6%) by nearly two to one
- **Friday, Thursday, and Saturday** are the highest-volume days; **October, September, and December** are the highest-volume months
- The 24-month forecast reproduces the same summer-peak / late-winter-trough seasonal pattern seen historically, with the peak shifting slightly toward mid-year

### Severity
- 74.0% of accidents are "No Injury," 15.1% "Moderate," 7.7% "Minor," 3.1% "Major," and just 0.17% "Fatal" — a realistic long-tail distribution that the engineered Severity Index is designed to weight correctly
- Only 351 accidents (0.17%) involved at least one fatality, out of 389 total fatal injuries recorded

### Road & Weather
- 78.7% of accidents occur in clear weather and 64.1% in daylight — a reminder that raw accident volume is dominated by "normal" conditions, while severity concentration should be read separately by condition
- 70.4% of accidents fall in the highest damage bracket (>$1,500)

### Machine Learning
- **XGBoost outperforms Prophet** for accident-volume forecasting on every metric (R² = 0.261 vs. -0.020; MAPE = 11.27% vs. 13.36%), thanks to its engineered lag and rolling-average features
- The injury-rate forecast moves independently of the accident-count forecast, rising fastest into early summer — suggesting summer accidents tend to be more injury-prone per event, not just more frequent
- The severity-by-weather and damage-by-road-surface classifiers honestly converge to majority-class predictions given a single categorical input and heavy class imbalance — reported transparently rather than dressed up as a nuanced result

---

## 🎓 Learning Outcomes

| Skill | Application |
|---|---|
| **Data Pipeline Design** | Medallion Architecture (Bronze → Silver → Gold) in pure Python |
| **Data Modeling** | Star schema with 1 fact table and 8 dimension tables |
| **Data Engineering** | pandas/NumPy cleaning, type correction, value standardization |
| **Feature Engineering** | Priority-cascade severity labeling, daypart bucketing, numeric damage proxy |
| **Machine Learning** | XGBoost vs. Prophet time-series comparison, recursive multi-step forecasting, classification with transparent limitation reporting |
| **Statistical Analysis** | Data-quality/null/duplicate checks, KPI & DAX measure derivation, ranking & outlier analysis |
| **Data Visualization** | 7-page interactive Power BI dashboard with slicers, heatmaps, and KPI cards |
| **Dashboard Design** | Figma "Vigilant Analytics" brand system — color palette, iconography, storytelling flow |

---

## 📄 Data Source & Attribution

**Dataset:** [Traffic Accidents](https://www.kaggle.com/datasets/oktayrdeki/traffic-accidents) by oktayrdeki on Kaggle (Open / Public license)

**Tools:** Python (pandas, NumPy, Plotly Express, XGBoost, Prophet) · Power BI · Figma

**Initiative:** Digital Egypt Pioneers Initiative (DEPI) — Data Analysis Track · 2025 / 2026

---

## ✨ Summary

This **Traffic Accident Analytics** project delivers:

🎯 **Complete 3-Layer Data Pipeline** — Bronze, Silver, Gold via dedicated Python notebooks
📊 **7-Page Interactive Power BI Dashboard** — Public Safety Portal, Overview, Road, Weather, Severity, Crash Cause, AI Insights
🤖 **XGBoost Forecasting Engine** — 24-month accident volume (35,179 predicted) & injury rate forecast, benchmarked against Prophet
🔬 **Transparent ML Methodology** — Time-based train/test split, honest majority-class reporting for single-feature classifiers
🚀 **Full Star-Schema Data Model** — 1 fact table + 8 dimensions across 209,306 real-world crash records
