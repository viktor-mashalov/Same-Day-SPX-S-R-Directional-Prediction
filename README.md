# SPX Dynamic Support-Resistance Model  
**Rule-based intraday levels driven by volatility, gap size, and VIX**

---

## 1 | What the Script Produces
For every trading day it **publishes one price level**—either a *support* (if the index gaps up) or a *resistance* (if it gaps down).  
* **Accuracy** tells us how often that level “held” by the close.  
* **Fill accuracy** measures only the subset of days when the level was actually touched intraday.  
* A simple ±0.40 % range around the open acts as a **baseline** for comparison.

> **Latest back-test (2014-01-01 → 2026-05-18)**  
> ```
> Model accuracy                     : 81.24 %
> Accuracy when |level| ≤ 0.50 %     : 81.24 %
> Fill accuracy (touched intraday)   : 49.58 %
> Fill rate (level was touched)      : 37.21 %
> -------------------------------------------------
> Baseline accuracy (|ret_open|≤0.4%) : 49.13 %
> Edge over baseline                 : 32.11 %
> -------------------------------------------------
> Direction correct on support days  : 60.04 %
> Direction correct on resistance    : 54.49 %
> ```
> *An 81 % “hold” rate and a 32 percentage-point edge over a naive ±0.4 % window suggest the levels are materially tighter and more reliable than a static buffer.*

---

## 2 | Core Ingredients

| Data Series                                      | Role in the Model |
|--------------------------------------------------|-------------------|
| **SPX OHLCV** (Yahoo Finance symbol `^GSPC`)     | Intraday high/low test the level; open/close drive gaps and returns. |
| **VIX** (`^VIX`)                                 | 30-day implied volatility proxy. |
| **3-month VIX** (`^VXV`)                         | Adds term-structure context (optional in this version). |
| **14-day ATR** *(Average True Range)*            | Realised volatility anchor. |

---

## 3 | Step-by-Step Logic

1. **Compute Overnight Gap**  
   `gap = open / prev_close − 1`  
   A positive gap signals bullish sentiment → we set a *support* underneath.  
   A negative gap sets a *resistance* overhead.

2. **Estimate Volatility Two Ways**  
   * **Implied:** `vix / √252` (one-day σ)  
   * **Realised:** `atr14 / prev_close`  
   The model uses the higher of the two.

3. **Dynamic z-Score**  
   Starts at **1.645** (≈95 % confidence) and scales up to **2×** for big gaps:  
   `z_score = 1.645 × (1 + |gap| / 0.005)`, capped at 2.

4. **Raw Percentage Offset**  
   `raw_pct = z_score × vol_est`

5. **Blend With Gap Size**  
   `combined_pct = 0.7 × raw_pct + 0.3 × |gap|`  
   (weights `w_vol` = 0.7, `w_gap` = 0.3)

6. **Cap at 0.40 % Absolute**  
   Practical intraday width ceiling: `max_pct = 0.004`.

7. **Assign Direction**  
   *Support → negative percent (below open)*  
   *Resistance → positive percent (above open)*

8. **Level Price**  
   `level_price = open × (1 + level_pct)`

9. **Validation Metrics**  
   * **Held?** Price closed on the protective side of the level.  
   * **Filled?** High/low actually hit the level intraday.  
   * **Success?** Filled **and** held by the close.

---

## 4 | Interpreting the Metrics

| Metric              | Meaning | Current Value |
|---------------------|---------|---------------|
| **Model accuracy**  | % of days the level held (filled or not). | **81 %** |
| **Fill rate**       | % of days the level was touched. | 37 % |
| **Fill accuracy**   | If touched, % it held. | 50 % |
| **Edge**            | Model accuracy – baseline accuracy. | +32 pp |
| **Directional stats** | Probability SPX keeps rising on a *support* day or falling on a *resistance* day. | 60 % / 54 % |

*High overall accuracy + sizable edge indicate the level is meaningfully tighter than a simple volatility band; however, once the market actually trades at the level (fill rate 37 %), the “bounce/fade” success drops to ~50 %.*

---

## 5 | Possible Uses (Illustrative Only)

* **Intraday risk framing** – know where a pull-back or pop might stall.  
* **Stop placement** – a tighter, probabilistically-derived region vs. fixed ATR stops.  
* **Option strikes** – choosing day-trade iron condors or butterflies around the level.

---

## 6 | Caveats & Next Steps

* **Transaction costs / slippage** not modelled.  
* **Single-asset test** – SPX only; warrants cross-asset validation.  
* **Look-ahead bias** – None by construction (all inputs lag 1 day), but further peer review encouraged.  
* **Parameter tuning** – Weights and caps chosen heuristically; grid search or Bayesian optimisation could improve robustness.

---

## 7 | Running the Notebook Yourself

1. `pip install yfinance pandas numpy`  
2. Copy the Python script from this repo.  
3. Adjust `start`, `end`, or even swap in another index symbol.  
4. Run, inspect the printed metrics, and plot the `df` DataFrame to visualise level placements.

---

## 8 | Disclaimer

This repository is **for educational illustration only**. It does not constitute trading advice or a solicitation to buy or sell any security. Past performance is not indicative of future results; execute at your own risk and always perform your own due diligence.

*© 2025 Viktor Mashalov*  
