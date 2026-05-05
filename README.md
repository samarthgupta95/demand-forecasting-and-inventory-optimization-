# Walmart Demand Forecasting & Inventory Optimization

---

## 📌 Project Overview

Walmart operates 45 stores across multiple regions, each running dozens of product departments simultaneously. Every week, store and supply chain managers must decide how much inventory to hold, when to reorder, and how much buffer to keep — decisions made repeatedly, at scale, with limited visibility into what demand will actually look like. This project builds a full operations analytics system — from raw weekly sales data to actionable inventory decisions — using time series forecasting and classical inventory optimisation theory.

---

## 🎯 Business Problem

The challenge is not a lack of data. Walmart captures weekly sales figures across every store and department, alongside external signals like temperature, fuel prices, consumer price indices, unemployment rates, and promotional markdowns. The data exists. What does not exist is a consistent, reliable way to translate that data into forward-looking demand expectations that managers can actually plan around.

The result is a recurring operational problem with costs on both sides. When demand is underestimated, shelves run empty before the next order arrives and sales are lost permanently. When demand is overestimated, excess inventory accumulates, storage costs rise, and markdowns are used to clear stock that should never have been ordered in the first place. Holiday periods and promotional events amplify both risks, as demand patterns during these windows behave very differently from the rest of the year.

**Questions from stakeholders:**

1. How does weekly sales demand behave across stores and departments — and are there patterns that repeat predictably over time?
2. Which store-department combinations experience the most erratic demand week over week?
3. How much does a promotional markdown actually move the needle on sales — and does its impact vary by store or department?
4. Are external conditions like fuel prices, inflation, and unemployment meaningfully correlated with how customers spend in-store?
5. For our highest-revenue departments, how confident can we be in a weekly sales estimate going into the next quarter?
6. At what point should a store trigger a reorder — and how much buffer stock is justified given how unpredictable a department's demand has historically been?

---

## 📂 Dataset & Scope

- **Source:** [Walmart Store Sales Forecasting — Kaggle](https://www.kaggle.com/datasets/aslanahmedov/walmart-sales-forecast)
- **Raw dataset:** train.csv (421,570 rows × 5 cols) merged with features.csv (8,190 rows × 12 cols)
- **Cleaned dataset:** 420,285 rows × 14 columns
- **Coverage:** 45 stores · 81 departments · February 2010 – October 2012
- **Domain:** Retail operations / supply chain

---

## 🧠 Approach

### 1. Plan Stage — Data Preparation
- Merged train.csv and features.csv on Store + Date — zero unjoined records
- Filled MarkDown1–5 nulls with 0 (absence of record = no active promotion)
- Dropped 1,285 negative Weekly_Sales rows — returns, not true demand
- Resolved duplicate IsHoliday columns from merge
- Final clean dataset: 420,285 rows × 14 columns — saved as `walmart_clean.csv`

### 2. Analyse Stage — EDA & Volatility
- Weekly sales baseline $40M–$51M with two holiday spikes: $289M (Nov/Dec 2010), $288M (Nov/Dec 2011)
- Store revenue tiers: Stores 20, 4, 14 each exceed $280M; Stores 5, 33, 44 fall below $50M
- Top departments: Dept 92 ($484M), Dept 95 ($449M), Dept 38 ($393M)
- Holiday weeks average $17,100 vs $15,900 non-holiday — 7.5% demand lift
- MarkDown5 (0.051) and MarkDown1 (0.047) carry strongest positive correlation with sales; Temperature and Fuel Price effectively uncorrelated
- STL decomposition confirmed near-flat trend, consistent annual seasonal spikes, tight residuals
- CV computed across all store-department combinations — most volatile: Store 35 Dept 59 (CV 3.35), Store 31 Dept 99 (2.94); most stable concentrated in Depts 8, 13, 40

### 3. Construct Stage — Forecasting & Inventory Modelling
- ADF test p-value 0.1102 — non-stationary; handled via d=1 within SARIMA
- 80/20 train-test split: 114 training weeks, 29 test weeks on Store 1, Dept 1
- SARIMA(1,1,1)(1,0,1)[52]: MAE $34,263 | RMSE $34,580 | MAPE 190.04% — failure driven by holiday spike weeks
- Prophet with yearly seasonality + custom Walmart holiday regressor: MAE $2,558 | RMSE $4,871 | MAPE 14.30% — carried forward as primary model
- EOQ model built on Prophet annual demand estimate ($1,082,179): EOQ 6,579 units · Safety Stock 16,210 units · Reorder Point 38,724 units

### 4. Execute Stage
- Strategic recommendations and inventory decisions detailed below

---

## 🔍 Key Findings

| Metric | Value | Context |
|--------|-------|---------|
| Weekly Sales Baseline | $40M–$51M | Across all 45 stores |
| Holiday Sales Spike | $289M peak | Nov/Dec 2010 — 7.5% avg lift over non-holiday |
| Top Store (Store 20) | $301M total | 6× higher than bottom-tier stores |
| Top Department (Dept 92) | $484M total | Depts 92, 95, 38 dominate revenue |
| Most Volatile Combo | Store 35, Dept 59 | CV 3.35 — requires significantly higher safety stock |
| SARIMA MAPE | 190.04% | Collapses on holiday weeks — structurally unfit |
| Prophet MAPE | 14.30% | Holiday regressor resolves spike forecasting |
| EOQ | 6,579 units | Minimises total ordering + holding cost |
| Safety Stock | 16,210 units | At 95% service level, 1-week lead time |
| Reorder Point | 38,724 units | Trigger level for replenishment order |
| MarkDown Correlation | 0.051 (MD5) | Strongest external signal — still weak overall |
| Best Forecast Model | Prophet | Outperforms SARIMA across MAE, RMSE, and MAPE |

---

## 🔥 Strategic Recommendation Snapshot

**01 — Treat Holiday Weeks as a Separate Planning Regime**
The 7.5% average holiday lift masks individual store spikes that SARIMA completely failed to capture (MAPE 190%). Holiday inventory decisions require a dedicated replenishment protocol — not a scaled version of a standard week. Prophet's holiday regressor confirms the demand structure is predictable once correctly modelled.

**02 — Differentiate Inventory Strategy by Store Revenue Tier**
The 6× revenue gap between top-tier stores (Stores 20, 4, 14 at $280M+) and bottom-tier stores (Stores 5, 33, 44 below $50M) means a uniform EOQ policy systematically over-stocks low-volume stores and under-stocks high-volume ones. Tier-specific reorder parameters are warranted.

**03 — Prioritise Safety Stock Investment in High-Volatility Departments**
Store-department combinations with CV above 2.0 — including Store 35 Dept 59 (3.35), Store 31 Dept 99 (2.94), and Store 13 Dept 99 (2.83) — require materially higher safety stock buffers than the portfolio-wide EOQ model assumes. A volatility-adjusted Z-score approach should replace the single 95% service level assumption across all departments.

**04 — Anchor Markdown Planning Around Departments 92, 95, and 38**
These three departments account for $1.33B in total sales across the dataset period. Markdown timing and depth decisions for these departments carry outsized financial consequences and should be modelled explicitly rather than treated as uniform promotional activity.

*Full strategic breakdown with department-level numbers and recommended steps available in the project notebook.*

---

## 🗃️ Project Assets

- 📓 **Notebook:** Full PACE-structured analysis with code, outputs, observations, and stage summaries — `notebook/demand_forecasting_inventory_optimization.ipynb`
- 📊 **Dashboard:** Two-page Power BI operations dashboard (Executive Overview + Demand Risk & Forecast Insights) — `dashboard/walmart_dashboard.pbix` · `dashboard/walmart_dashboard.pdf`
- 🤵 **Executive Presentation:** Business storytelling deck — findings, forecast model selection, inventory decisions, and strategic recommendations — `presentation/walmart_executive_presentation.pptx`

---

## ⚙️ Tools & Technologies

- **Python** (Pandas, NumPy, Matplotlib, Seaborn) — data cleaning, EDA, visualisation, PACE notebook
- **Statsmodels** — ADF stationarity test, STL decomposition, SARIMA modelling
- **Prophet** — time series forecasting with custom holiday regressors
- **Scikit-learn** — MAE, RMSE evaluation metrics
- **Power BI** — two-page operations dashboard
- **Jupyter Notebook** — full PACE-structured analysis

---

## 🚀 Final Takeaway

Demand is not random — it has structure, and that structure is forecastable. The right model, applied with the right assumptions, converts historical sales patterns into inventory decisions that are quantifiable, defensible, and operationally actionable.

---

*Samarth Gupta | Data Analyst | Business Analytics & Machine Learning*
