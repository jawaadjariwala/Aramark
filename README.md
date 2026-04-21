# Aramark Procurement Analytics

A cloud-native big data analytics project on Aramark's 2025 US procurement dataset (~43 million rows). Built for CS 562 — Big Data.

The project goes beyond descriptive dashboarding to quantify supplier concentration risk, test whether external weather patterns visibly move procurement behavior, and surface hidden customer archetypes through machine learning.

---

## Headline Findings

1. **Extreme supplier concentration.** MASTER DISTRIBUTOR handles **54.3%** of all US Aramark spend and is the #1 distributor in every single US state. The top 10 distributor groups together account for ~87% of total spend.

2. **Every US state is concentrated under DOJ antitrust standards.** 42 of 52 states exceed HHI 2,500 (HIGH risk tier); 10 are MODERATE; zero are competitive.

3. **Manufacturer layer is diversified — risk is operational, not sourcing.** Upstream manufacturer HHI stays below 500 in nearly every state. If the main distributor channel were disrupted, the products themselves would still be available in the market.

4. **Weather measurably shifts the procurement mix.** FOOD's share of (state, month) spend swings ~15 percentage points across the temperature range; `r = −0.33` with summer heat intensity (the strongest single weather correlation in the dataset).

5. **Concentration risk × weather shock interact.** HIGH-risk states show ~2 percentage points wider spend deviation during extreme-weather months than MODERATE-risk states — direct empirical link between Phase 3 theory and Phase 5 operational data.

6. **Customer base organizes into six interpretable procurement archetypes.** Specialty distribution channels (DIRECT BEVERAGE, ENGINEERING MRO) emerge as dominant for specific customer segments — patterns invisible in national aggregates.

---

## Tech Stack

| Layer | Tool |
|---|---|
| Source data | Google Cloud Storage |
| Warehouse | Google BigQuery (16 stored views + 1 weather table) |
| Notebooks | Google Colab |
| Analysis | Python (pandas, numpy, scikit-learn) |
| Visualization | matplotlib, seaborn |
| External data | NOAA CIRS climdiv (statewide monthly climate) |
| Presentation | PowerPoint (generated via python-pptx) |
| Documentation | Markdown → PDF (markdown-pdf library) |

---

## Architecture

```
GCS (raw 9.4 GB CSV)
   │
   ▼  one-time load (~3 min)
BigQuery — aramark_spend.raw_spend (43M rows, typed)
   │
   ▼  16 stored views (agg_*)
┌─────────────┬─────────────┬─────────────┐
│  Phase 3     │  Phase 5    │  Phase 7    │
│ concentration│  weather    │  ML         │
│ & risk       │  correlation│  segments   │
└─────────────┴─────────────┴─────────────┘
         │          │           │
         ▼          ▼           ▼
      agg_state_risk_scorecard (shared integration point)
         │
         ▼
    Phase 5 and Phase 7 both join on the scorecard
```

Every analysis notebook queries the same shared BigQuery views, so the numbers are consistent across phases. Views cost zero storage and cannot drift from source data.

---

## Key BigQuery Views

| View | Purpose |
|---|---|
| `raw_spend` | typed 43M-row source table |
| `agg_distributor_group` | national distributor rollup |
| `agg_category_level_1` / `agg_category_l1_l2` | product hierarchy rollups |
| `agg_state` / `agg_year_month` | single-dim geo and time aggregations |
| `agg_category_state_month` | state × month × category cube |
| `agg_distributor_state` / `agg_distributor_room_band` | cross-dimensional rollups |
| `agg_category_state` / `agg_category_month` / `agg_category_entity` | category cross-cuts |
| `agg_distributor_category` | Phase 3 category-level HHI input |
| `agg_state_risk_scorecard` | **integration point — used by Phases 5 & 7** |
| `weather_state_monthly` | NOAA climdiv weather table (49 states × 12 months) |

---

## Reproducing the Analysis

### Prerequisites
- Google Cloud account with BigQuery access to the `big-data-algorithms-493312` project
- Python 3.10+
- Packages: `pandas`, `numpy`, `scikit-learn`, `matplotlib`, `seaborn`, `google-cloud-bigquery`, `gcsfs`, `noaa-sdk`

### Order of execution
Run the notebooks in numerical order. Each notebook is self-contained but assumes the previous phase's BigQuery artifacts already exist:

1. `01_Data_Exploration.ipynb` — runs against the GCS CSV directly
2. `02_bigquery_load_and_aggregations.ipynb` — loads the BigQuery table and creates 14 views
3. `03_concentration_and_risk.ipynb` — adds 2 more views including the risk scorecard
4. `04_external_data_ingestion.ipynb` — loads the NOAA weather table
5. `05_weather_correlation_analysis.ipynb` — creates `agg_category_state_month` and runs correlation
6. `07_ml_segmentation.ipynb` — pulls sample from `raw_spend`, runs clustering


## Data Caveats

Worth stating explicitly on any downstream business memo that uses this analysis:

1. **Only 12 months of data (2025).** Seasonal patterns cannot be separated from one-off events. No forecasting was attempted.
2. **`spend_random_factor` is randomized** for confidentiality. Ratios, shares, and rankings are trustworthy; absolute dollar totals are not.
3. **Weather coverage: 49 states.** Hawaii, DC, and Puerto Rico are excluded from NOAA climdiv; all weather-related claims in Phase 5 and Phase 7 exclude those locations.
4. **`manufacturer_id` is ~34% missing.** Manufacturer HHI reflects the known portion of the supplier mix only.
5. **All Phase 5 relationships are correlations in one year**, not causal proofs.
6. **BigQuery `RAND()` sampling is non-reproducible.** Phase 7 re-runs produce slightly different cluster IDs but the structural archetypes (six differentiated segments, specialty channels, FOOD backbone) are stable.

---

## Business Recommendations (Summary)

The full recommendations are in Section 9 of `REPORT.pdf`. Seven concrete actions each with measurable KPIs:

1. Pilot supplier diversification in the worst-concentrated states (SD, ME, IA, KS, DE)
2. Treat alternative-channel readiness as a tracked operational KPI per region
3. Focus risk investment on logistics redundancy, not manufacturer re-sourcing
4. Integrate NOAA weather forecasts into monthly replenishment cadence
5. Activate contingency procedures proactively during forecasted extreme-weather months
6. Study specialty-channel clusters (0 and 3) and design migration offers for mass-market FOOD clusters
7. Publish data caveats alongside every downstream business memo

---

## Contributors

| Phase | Deliverable | Lead |
|---|---|---|
| 0 | `01_Data_Exploration.ipynb` | Jawaad |
| 1–2 | `02_bigquery_load_and_aggregations.ipynb` | Jawaad |
| 3 | `03_concentration_and_risk.ipynb` | Jawaad |
| 4 | `04_external_data_ingestion.ipynb` | Tharun and Jawaad|
| 5 | `05_weather_correlation_analysis.ipynb` | Tharun and Jawaad |
| 7 | `07_ml_segmentation.ipynb` | Praneel and Jawaad|
| — | Report, slides, README | Jawaad |

Commits are attributed per-author; `git blame` gives per-line attribution.

---

## Course

**CS 562 — Big Data**
Project submission, 2026.

---

## License

This repository is a course project using an Aramark-provided confidential dataset with randomized spend values. Analysis code and documentation are the team's own work. The underlying data itself is not redistributable.
