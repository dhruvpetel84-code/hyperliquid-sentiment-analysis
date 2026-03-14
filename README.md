# 📊 Hyperliquid × Fear & Greed Index: Sentiment-Driven Trading Analysis

> **A data science project analyzing how crypto market sentiment shapes trader behavior and profitability on Hyperliquid — a decentralized perpetual futures DEX.**

---

## Table of Contents
- [Project Summary](#project-summary)
- [Tool Stack](#tool-stack)
- [Setup & Installation](#setup--installation)
- [How to Run](#how-to-run)
- [Output Charts & Tables](#output-charts--tables)
- [Methodology](#methodology)
- [Key Insights](#key-insights)
- [Strategy Recommendations](#strategy-recommendations)
- [Why These Visualizations?](#why-these-visualizations)

---

## Project Summary

This project merges two datasets:

| Dataset | Source | Granularity | Key Fields |
|---|---|---|---|
| **Fear & Greed Index** | Alternative.me | Daily | value (0–100), classification |
| **Hyperliquid Trades** | Hyperliquid DEX | Trade-level | account, coin, size_usd, closed_pnl, side, timestamp |

**Core Questions Answered:**
1. Does trader performance (PnL, win rate, drawdown) differ significantly between Fear and Greed days?
2. Do traders change their behavior (frequency, size, long/short bias) based on sentiment?
3. Which trader segments thrive or suffer most under different sentiment regimes?
4. Can we predict next-day profitability using sentiment + behavioral features?
5. What behavioral archetypes exist among Hyperliquid traders?

---

## Tool Stack

| Tool | Version | Why Used |

| **Python**  | Core language |
| **pandas** | Data wrangling, groupby aggregations, merging |
| **numpy**  | Vectorized math, percentile calculations |
| **matplotlib**  | Fine-grained chart control (box plots, bar charts, radar) |
| **seaborn**  | Statistical heatmaps, styled scatter plots |
| **scikit-learn** | KMeans clustering, Random Forest, PCA, StandardScaler |
| **scipy** | Mann-Whitney U test, Kruskal-Wallis (non-parametric stats) |
| **streamlit**  | Interactive dashboard for result exploration |
| **jupyter** | Notebook environment for reproducible analysis |

**Why these tools specifically:**
- `scipy` non-parametric tests were chosen over t-tests because PnL distributions in trading are **heavily non-normal** (fat tails, skew, outliers).
- `scikit-learn` KMeans + PCA for clustering because the dataset is tabular with mixed-scale features — tree-based or neural approaches would overfit given the sample size.

---



## Setup & Installation



### Step 1 — Clone or download the project
```bash
git clone https://github.com/dhruvpetel84-code/hyperliquid-sentiment-analysis.git
cd hyperliquid-sentiment-analysis
```

### Step 2 — Create a virtual environment
```bash
# Create
python -m venv venv

# Activate — Windows CMD
venv\Scripts\activate


```

### Step 3 — Install dependencies
```bash
pip install -r requirements.txt
```

### Step 4 — Place data files
```
Place both CSV files inside a data/ folder:
  data/fear_greed_index.csv
  data/historical_data.csv

OR place them in the same directory as the notebooks
and update the file paths in Cell A1 accordingly.
```

---

## How to Run

### Option A — Jupyter Notebook (full analysis)
```bash
jupyter notebook hyperliquid_sentiment_analysis.ipynb
```
Run all cells top to bottom (`Kernel → Restart & Run All`).
Charts are saved automatically to the working directory.



## Output Charts & Tables

### Part A — Data Preparation
| Output | Description |
|---|---|
| Console print | Row counts, column names, missing value audit |
| Console print | Duplicate count using composite key (Transaction Hash + Trade ID) |
| Console print | Match rate of trades to sentiment dates |
| `daily` DataFrame | Trader-day level aggregated metrics |
| `trader` DataFrame | Lifetime stats per account |

### Part B — Analysis Charts

| Chart | Type | What It Shows |
|---|---|---|
| `chart1` | Grouped bar | Mean PnL, Win Rate, Drawdown proxy — Fear vs Neutral vs Greed |
| `chart2` | Grouped bar | Trade frequency, size, long ratio, volume by sentiment |
| `chart3` | Dual-axis time series | Long/short bias vs F&G index value over time |
| `chart4` | Grouped bar | High vs Low leverage trader performance by sentiment |
| `chart5` | Grouped bar | Frequent vs Infrequent traders by sentiment |
| `chart6` | Grouped bar | Consistent Winners vs Inconsistent traders by sentiment |
| `chart7` | Box plot | Full PnL distribution per sentiment (shows tails + outliers) |
| `chart8` | Bar | Long bias % across all 5 sentiment classes |
| `chart9` | Grouped bar | Mean + Median trading volume by sentiment class |

### Part C — Strategy
| Chart | Type | What It Shows |
|---|---|---|
| `chart10` | Color-coded table | Position size, bias, leverage, frequency rules per regime |

### Bonus — Model & Clustering
| Chart | Type | What It Shows |
|---|---|---|
| `chart11` | Feature importance + confusion matrix | RF model: what predicts next-day profitability |
| `chart12` | Elbow line | Optimal k for KMeans behavioral clustering |
| `chart13` | PCA scatter | 4 behavioral archetypes visualized in 2D |
| `chart_interest_elbow` | Elbow line | Optimal k for interest-based clustering |
| `chart_interest_clusters_pca` | PCA scatter + centroids | Trader groups by coin preference + behavior |
| `chart_coin_preference_heatmap` | Heatmap | % trades per coin per cluster |
| `chart_radar_profiles` | Spider/radar | Multi-dimensional behavioral profile per cluster |
| `chart_cluster_pnl_by_sentiment` | Grouped bar | Each cluster's PnL and win rate across sentiment regimes |

---

## Methodology

### Data Pipeline

```
Raw Trades (IST timestamps)
        │
        ▼
  Convert IST → UTC  (subtract 5h30m)
        │
        ▼
  Create daily date bucket (UTC midnight)
        │
        ▼
  Left-join Fear & Greed Index on date
        │
        ▼
  Engineer features:
    net_pnl = closed_pnl − fee
    is_win, is_close, is_long
        │
        ▼
  Aggregate → daily DataFrame (trader × day)
  Aggregate → trader DataFrame (lifetime stats)
        │
        ▼
  ┌─────────────┬──────────────┬──────────────┐
  │  Sentiment  │ Segmentation │  ML + Cluster│
  │  Analysis   │  (3 tiers)   │              │
  └─────────────┴──────────────┴──────────────┘
```

### Timestamp Handling
Trade timestamps are in IST (UTC+5:30). The Fear & Greed index uses UTC dates.
We subtract 5h30m before creating the daily bucket to prevent cross-day contamination — especially critical for trades executed near midnight IST.

### Statistical Approach
Non-parametric tests (Mann-Whitney U, Kruskal-Wallis) were chosen over t-tests because:
- Trading PnL distributions are right-skewed with fat tails
- Large outlier trades violate the normality assumption required by parametric tests
- Mann-Whitney U tests for **stochastic dominance** — whether one group tends toward higher values regardless of distribution shape

### Segmentation Logic
| Segment | Split Method | Rationale |
|---|---|---|
| High/Low Leverage | Median split on avg_size_usd | Notional size proxies effective leverage when margin data unavailable |
| Frequency tiers | 33rd / 66th percentile | Avoids arbitrary thresholds; always produces balanced groups |
| Consistent Winner | Sharpe proxy (mean/std daily PnL) above median | Risk-adjusted measure; not fooled by one lucky large trade |

### Predictive Model
- **Algorithm:** Random Forest (200 trees, max_depth=6)
- **Target:** Binary — will tomorrow be a profitable day? (1=yes, 0=no)
- **Features:** n_trades, total_pnl, win_rate, avg_size_usd, long_ratio, volume, F&G value, sentiment encoding
- **Validation:** 5-fold stratified cross-validation (stratified to handle class imbalance)
- **Metric:** AUC-ROC — insensitive to class imbalance, measures ranking quality
- **depth=6 cap:** Prevents overfitting on noisy financial data; forces the model to find general patterns

---

## Key Insights

### 🔍 Insight 1 — Fear Days Are High-Variance, Not Uniformly Bad
Fear days produce the **widest PnL distribution** — the deepest losses and the best contrarian gains both concentrate here. High-leverage traders experience disproportionately large drawdowns during Fear. Consistent Winners, however, maintain or improve win rate by keeping position sizes small and waiting for high-conviction setups.

> **Implication:** Fear is not uniformly bad — it is a dangerous time to be over-leveraged, and a genuine opportunity for disciplined, small-size traders.

### 🔍 Insight 2 — Traders Herd Into Longs During Greed
Long ratio rises systematically with greed — reaching 65–70% of all trades during Extreme Greed. This is a classic **crowded trade** signal. The crowd is simultaneously trend-following at the moment when contrarian risk is highest.

> **Implication:** The long/short ratio itself becomes a sentiment indicator. A long ratio above 65% is a warning sign, not confirmation to buy.

### 🔍 Insight 3 — Volume Spikes at Extremes, Edge Compresses
Trading volume peaks sharply during both Extreme Fear and Extreme Greed. Yet per-trade mean PnL is **lowest** during these high-activity periods. High-frequency traders are most affected — they execute more trades exactly when the signal-to-noise ratio is worst.

> **Implication:** Activity is not edge. The most liquid, most volatile moments are when the average trade is least profitable.

---

## Strategy Recommendations

### Strategy 1 — Fear Regime Protocol (F&G < 50)

| Sub-regime | Position Size | Direction Bias | Action |
|---|---|---|---|
| Extreme Fear (0–24) | 25% of normal | Contrarian long only | Wait for capitulation candle; tight stop |
| Fear (25–49) | 50% of normal | Long-OK; avoid new shorts | Mean-reversion preferred over momentum |

**Rationale:** High-leverage traders blow up most during Fear. The winners here run small, patient positions. Risk control is the edge — not direction-picking.

### Strategy 2 — Greed Regime Protocol (F&G > 50)

| Sub-regime | Position Size | Direction Bias | Action |
|---|---|---|---|
| Neutral (50) | 100% normal | Data-driven | Full strategy; no sentiment overlay needed |
| Greed (51–74) | 75–100% | Trim longs progressively | Momentum works, but tighten stops; watch long ratio |
| Extreme Greed (75–100) | 50% | Prepare short setups | Reduce frequency; contrarian short on exhaustion signals |

**Rationale:** Extreme Greed is the most tempting time to overtrade — and the most damaging one. Volume spikes but per-trade edge compresses. Consistent Winners actually *reduce* activity here.

### Quick Reference Table

```
F&G 0–24   → 25% size │ contrarian longs  │ low leverage  │ very low frequency
F&G 25–49  → 50% size │ long-OK           │ low leverage  │ low-medium frequency
F&G 50     → 100% size│ data-driven       │ medium        │ normal frequency
F&G 51–74  → 75% size │ trim longs        │ medium-high   │ high frequency (momentum)
F&G 75–100 → 50% size │ short setups      │ low-medium    │ reduce frequency
```

---

## Why These Visualizations?

| Chart Type | Why Chosen |
|---|---|
| **Box plot** (Chart 7) | Shows full PnL distribution including tails and outliers. Bar charts of means would completely hide the most important information in trading data — the extremes. |
| **Grouped bar charts** (Charts 1–6) | Direct side-by-side comparison across sentiment groups for multiple metrics. Simple, readable, unambiguous. |
| **Dual-axis time series** (Chart 3) | Long ratio and F&G index live on different scales. Dual axis allows visual correlation without normalizing away the actual values. |
| **Heatmap** (Coin preference) | A sparse matrix of traders × coins is unreadable as a table. Color encoding makes cluster differences instantly visible across many coins. |
| **Radar/Spider chart** | Ideal for multi-dimensional behavioral profiles. Shows the *shape* of a cluster's behavior holistically — one glance reveals if a cluster is high-frequency, high-size, low-win-rate simultaneously. |
| **PCA scatter** | Dimensionality reduction makes cluster separation visually verifiable. Without PCA, comparing 20-dimensional clusters is impossible to eyeball. Centroids marked as ★ anchor each group. |
| **Feature importance bar** | Horizontal bar is the clearest format for ranked importance. Immediately communicates which features the model depends on — crucial for trusting or questioning the model. |

---


