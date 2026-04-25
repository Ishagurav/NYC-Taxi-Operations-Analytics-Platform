# NYC-Taxi-Operations-Analytics-Platform
📌 Overview
End-to-end data engineering pipeline built on Databricks Community Edition processing 3M+ real-world NYC TLC Yellow Taxi records (January 2026) using Medallion Architecture across Bronze, Silver and Gold Delta Lake layers — delivered via Databricks Lakeview AI/BI Dashboard.


## 📂 Project Structure

```
nyc-taxi-operations-analytics/
│
├── notebooks/
│   ├── 01_bronze_layer.ipynb        # Raw ingestion into Delta table
│   ├── 02_silver_layer.ipynb        # Cleaning, validation, enrichment
│   ├── 03_gold_layer.ipynb          # Business aggregation tables
│   └── 04_lakeview_dashboard.ipynb  # Visualization queries
│
├── data/
│   └── README.md                    # Dataset download instructions
│                                      (actual files not included — see below)
│
├── screenshots/
│   ├── bronze_layer.png             # Bronze table output
│   ├── silver_layer.png             # Silver table output
│   ├── gold_layer.png               # Gold tables output
│   └── dashboard.png                # Lakeview dashboard screenshot
│
└── README.md
```

---

## 📥 Dataset — How to Download

> ⚠️ Dataset is not included in this repository due to large file size (~200MB).
> Please download it directly from the official NYC TLC source below.

### Step 1 — Go to official NYC TLC website
```
https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page
```

### Step 2 — Download this specific file
```
Year  : 2026
Month : January
File  : Yellow Taxi Trip Records (PARQUET)
```

### Step 3 — Upload to Databricks
1. Login to Databricks
2. Go to **Catalog → nyc_taxi_catalog → bronze → bronze_raw**
3. Click **"Upload to this Volume"**
4. Upload the downloaded `.parquet` file

### File Details
| Property | Value |
|---|---|
| File Name | yellow_tripdata_2026-01.parquet |
| Format | Parquet |
| Size | ~200MB |
| Rows | ~3 Million |
| Source | NYC Taxi & Limousine Commission (TLC) |

---

## 🗄️ Databricks Setup — Catalog Structure

```
nyc_taxi_catalog                    ← Catalog
├── bronze                          ← Schema
│   ├── Volume: bronze_raw          ← Upload parquet file here
│   └── Table:  bronze_taxi_data    ← Created by notebook 01
│
├── silver                          ← Schema
│   └── Table:  silver_taxi_data    ← Created by notebook 02
│
└── gold                            ← Schema
    ├── Table: gold_hourly_demand   ← Created by notebook 03
    ├── Table: gold_vendor_revenue
    ├── Table: gold_payment_analysis
    ├── Table: gold_trip_efficiency
    └── Table: gold_daily_analysis
```

---

## 🥉 Bronze Layer — `01_bronze_layer.ipynb`

**Purpose:** Store raw data exactly as received — no transformations

| Step | Action |
|---|---|
| Read | Raw Parquet from Unity Catalog Volume |
| Add | `ingestion_timestamp` — when data was loaded |
| Add | `source_file` — which file it came from |
| Save | Delta table → `nyc_taxi_catalog.bronze.bronze_taxi_data` |

---

## 🥈 Silver Layer — `02_silver_layer.ipynb`

**Purpose:** Clean, validate and enrich Bronze data

### Data Quality Checks Applied
| Check | Rule |
|---|---|
| Type casting | `timestamp_ntz` → `timestamp` |
| Null removal | Drop nulls in passenger_count, fare_amount, trip_distance |
| Fare validation | `0 < fare_amount <= 500` |
| Distance validation | `0 < trip_distance <= 100` |
| Passenger validation | `0 < passenger_count <= 6` |
| Duration validation | `1 min <= trip_duration <= 1440 mins` |
| Outlier removal | Invalid fares and distances removed |

### Derived Columns Added
| Column | Description |
|---|---|
| `trip_duration_mins` | Dropoff time minus pickup time in minutes |
| `cost_per_mile` | fare_amount divided by trip_distance |
| `pickup_hour` | Hour extracted from pickup datetime |
| `pickup_day` | Day of week from pickup datetime |
| `trip_category` | Short / Medium / Long based on distance |
| `payment_label` | Credit Card / Cash / No Charge / Dispute |
| `is_high_tip` | Yes if tip > 20% of fare |

> ✅ ~15% invalid records removed after all quality checks

---

## 🥇 Gold Layer — `03_gold_layer.ipynb`

**Purpose:** Business-ready aggregated tables for analytics

| Table | Business Question Answered |
|---|---|
| `gold_hourly_demand` | Which hours are busiest? When to deploy more drivers? |
| `gold_vendor_revenue` | Which vendor earns more revenue? |
| `gold_payment_analysis` | How are customers paying? Are card tips higher? |
| `gold_trip_efficiency` | Which trip type is most profitable per mile? |
| `gold_daily_analysis` | Which day of week has highest demand? |

---

## 📊 Lakeview Dashboard — `04_lakeview_dashboard.ipynb`

**6 Visualizations + 3 KPI Counters built on Gold tables**

| Chart | Type | Gold Table |
|---|---|---|
| Busiest Hours | Bar Chart | gold_hourly_demand |
| Revenue Trend | Line Chart | gold_hourly_demand |
| Payment Breakdown | Pie Chart | gold_payment_analysis |
| Vendor Performance | Bar Chart | gold_vendor_revenue |
| Trip Efficiency | Bar Chart | gold_trip_efficiency |
| Busiest Days | Bar Chart | gold_daily_analysis |
| Total Trips | KPI Counter | gold_hourly_demand |
| Total Revenue | KPI Counter | gold_hourly_demand |
| Avg Fare | KPI Counter | gold_hourly_demand |

---

## 💡 Key Business Insights

- 🕕 **6PM–8PM** is the peak revenue window across all weekdays
- 💳 **Credit Card** is the dominant payment method (~70% of trips)
- 🚗 **Medium trips (2–10 miles)** generate the highest total revenue
- 📅 **Weekdays** consistently outperform weekends in trip volume
- 🏆 **Vendor 2** outperforms Vendor 1 in total revenue

---

## 🚀 How to Run

### Prerequisites
- Databricks Community Edition account
- Dataset downloaded from NYC TLC website (see above)

### Steps
```
1. Create Unity Catalog structure:
   Catalog  → nyc_taxi_catalog
   Schemas  → bronze, silver, gold
   Volumes  → bronze_raw (under bronze schema)

2. Upload yellow_tripdata_2025-01.parquet 
   to bronze_raw volume

3. Run notebooks in order:
   01_bronze_layer.ipynb
   02_silver_layer.ipynb
   03_gold_layer.ipynb
   04_lakeview_dashboard.ipynb

4. Open Databricks Dashboards to  make the view
   Lakeview AI/BI Dashboard



## 📄 License
This project is for educational and portfolio purposes.
Dataset sourced from NYC Taxi & Limousine Commission (TLC) — publicly available.
