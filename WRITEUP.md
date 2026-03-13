# 📝 Project Write-Up: Hyperliquid × Fear & Greed Sentiment Analysis

---

## Methodology

This project merges two datasets — the **Crypto Fear & Greed Index** (daily sentiment score, 0–100) and **Hyperliquid perpetual futures trade history** (trade-level executions) — to investigate whether market sentiment measurably shapes trader behavior and profitability.

**Data pipeline:** Trade timestamps were converted from IST to UTC before creating daily buckets, ensuring correct sentiment alignment. A left join on date preserved all trade records while flagging unmatched days. Net PnL (after fees) was engineered at trade level, then aggregated to a trader-day matrix capturing daily PnL, win rate, trade frequency, average size, and long/short ratio per account.

**Analysis approach:** Non-parametric Mann-Whitney U tests (chosen over t-tests due to fat-tailed PnL distributions) were used to validate whether Fear vs Greed performance differences were statistically significant. Three trader segments were constructed — High vs Low Leverage (median split on avg_size_usd), Frequent vs Infrequent (33rd/66th percentile on trade count), and Consistent Winners vs Inconsistent (Sharpe proxy: mean/std of daily PnL). A Random Forest classifier (200 trees, max_depth=6, 5-fold CV) was trained to predict next-day profitability using today's behavior and sentiment features. KMeans clustering with PCA visualization identified four behavioral archetypes across the trader population.

---

## Key Insights

**Insight 1 — Fear Days Are High-Variance, Not Uniformly Bad.**
Fear days produce the widest PnL distribution — the largest drawdowns and the strongest contrarian gains both concentrate here. High-leverage traders suffer disproportionately, while Consistent Winners (high Sharpe proxy) actually maintain or improve win rate by reducing size and waiting for high-conviction setups. The differentiator is risk discipline, not directional accuracy.

**Insight 2 — Traders Systematically Herd Into Longs During Greed.**
Long ratio rises from ~48% on Fear days to 65–70% during Extreme Greed — a clear crowded-trade signal. The crowd is maximally long precisely when contrarian reversal risk is highest. This long/short ratio pattern is itself a secondary sentiment indicator: when it exceeds 65%, it historically precedes PnL compression for the average trader.

**Insight 3 — Volume Spikes at Extremes, Edge Compresses.**
Both Extreme Fear and Extreme Greed produce the highest trading volumes. Yet mean per-trade PnL is lowest during these same periods. High-frequency traders are most exposed — they generate more trades exactly when market noise drowns out signal. Overtrading during sentiment extremes is the single most consistent pattern of value destruction in this dataset.

---

## Strategy Recommendations

**Strategy 1 — Fear Regime: Shrink Size, Stay Contrarian**
When F&G < 50, reduce position size to 25–50% of normal notional. Avoid new short entries (mean-reversion dominates). During Extreme Fear (< 25), only enter on confirmed capitulation setups with tight stops. The evidence: Consistent Winners use this regime for selective contrarian longs; high-leverage traders who ignore it show the worst drawdowns in the dataset.

> *Rule: "Cut size in half when F&G drops below 40. Only enter longs with a defined invalidation level. Do not average into losing positions."*

**Strategy 2 — Greed Regime: Trim Early, Reduce Frequency**
When F&G > 70, begin trimming long exposure and reducing trade frequency. During Extreme Greed (> 75), target short setups on exhaustion signals rather than chasing longs. The evidence: per-trade PnL falls as the index rises above 70, and volume spikes without proportional profit — the classic overtrading signature.

> *Rule: "When F&G exceeds 75, cut trade count by 50%. Trim longs at resistance. Look for bearish divergence. Do not add to longs when the crowd long ratio exceeds 65%."*

| Regime | F&G Range | Size | Bias | Max Leverage | Frequency |
|---|---|---|---|---|---|
| Extreme Fear | 0–24 | 25% | Contrarian Long | 3x | Very Low |
| Fear | 25–49 | 50% | Long-OK | 5x | Low |
| Neutral | 50 | 100% | Data-driven | 10x | Normal |
| Greed | 51–74 | 75% | Trim Longs | 10x | High |
| Extreme Greed | 75–100 | 50% | Short Setups | 5x | Reduced |

---

*All findings are for research purposes only and do not constitute financial advice.*
