# Volatility Prediction Models Guide

This document explains the two companion machine learning models used to predict Bitcoin daily volatility and expansion probability. Both models are trained on historical BTC data and used to generate forward-looking predictions stored in the database.

---

## Model 1: Future Volatility Regressor (XGBRegressor)

### How the Model is Trained


**Training Overview:**
The model is trained on historical BTC data, using engineered targets and a variety of technical and volatility-related signals. Details of feature engineering and exact inputs are intentionally omitted.

**Model Performance Metrics:**
- RMSE (Root Mean Squared Error): measures average prediction error in volatility units
- R² (coefficient of determination): explains variance in typical market conditions
- Directional Accuracy: % of times the model correctly predicts if volatility will rise or fall
- Vol Expansion Accuracy: % of times the model correctly predicts if vol expands relative to a baseline

---

### What the Model Predicts

The regressor outputs a **single continuous value** for each row: the expected future volatility in the original (non-log) scale.

**Output Range & Interpretation:**

| Prediction Value | Interpretation | Market Context |
|---|---|---|
| **0.005–0.010** | Low volatility | Calm, range-bound market; tight spreads; low overnight gaps |
| **0.010–0.020** | Normal volatility | Typical daily moves; standard intraday ranges; stable conditions |
| **0.020–0.040** | Elevated volatility | Active trading; wider moves; increased news sensitivity |
| **0.040–0.080+** | High volatility | Directional moves; large gaps; macro events; panic/euphoria |

**Example Predictions on Today's BTC Daily Chart:**

Suppose today (March 5, 2026) at close we observe:
- `realized_vol_5 = 0.018` (5-day realized volatility)
- `realized_vol_10 = 0.015` (10-day realized volatility)
- Market regime is stable, no major news

**Scenario 1: Calm Market**
- Model predicts: `future_vol = 0.012`
- Interpretation: Expect the next 3–5 days to have ~1.2% daily moves
- Confidence: High (if historical patterns persist)

**Scenario 2: Elevated Risk**
- Current indicators show: `vol_lag_3 = 0.035`, `vol_ratio = 1.5` (short-term vol >> long-term)
- Model predicts: `future_vol = 0.032`
- Interpretation: Expect sharp moves (3.2% daily); likely whipsaws; breakout or breakdown brewing

**Scenario 3: Spike/Crash Event**
- `realized_vol_5 = 0.055`, `return_skew_20 = -2.1` (downside skew), Fed decision imminent
- Model predicts: `future_vol = 0.068`
- Interpretation: Extreme volatility expected; 6.8% swings possible; option premiums justify hedging

---

### Best Practices: How to Infer Trade Ideas

1. **Volatility Expansion Trade (Long BTC on Breakout)**
   - When `future_vol` prediction > current realized volatility and RSI < 30 (oversold):
     - Go long BTC expecting a volatility-driven breakout
     - Confluence: High volume on the entry candle confirms momentum
   - Example: If realized vol = 0.018 and model says `future_vol = 0.035`, and RSI dips below 30, long BTC with stop below recent low

2. **Directional Trade with Vol Confirmation**
   - Use the regressor output to set risk/position size:
     - High vol prediction (>0.04) → reduce position size or close existing positions
     - Low vol prediction (<0.015) → increase position size for long/short BTC
   - Example: If model predicts low vol (0.010) tomorrow, increase long BTC position; if high vol (0.050) expected, reduce long BTC exposure

3. **Stop-Loss & Target Sizing**
   - Set stops at 2–3× the predicted daily move
   - Example: If `future_vol = 0.025` (~2.5% expected), set stop at ±5–7.5% and target at 3–4%
   - Confluence: Use support/resistance levels for better stop placement

4. **Volatility Mean Reversion**
   - When `future_vol` is 2–3 SD above the 20-day mean, expect reversion; short BTC if overbought
   - Confluence: RSI > 70 signals overbought, MACD bearish divergence confirms
   - Example: If typical vol is 0.015 and prediction spikes to 0.055, short BTC expecting pullback

5. **Regime Filtering**
   - Check the `market_regime_score` input alongside the vol prediction:
     - Uptrend regime + low vol → high confidence in long BTC
     - Downtrend regime + high vol → short BTC or close longs
   - Confluence: ADX > 25 confirms trend strength
   - Example: Uptrend regime with low vol prediction → long BTC; downtrend with high vol → short BTC

6. **Combining with Other Indicators**
   - Cross-check predictions with:
     - RSI extreme levels (overbought/oversold often precede vol spikes)
     - MACD divergences (signal hidden vol)
     - Recent volume profile (volume surge → vol spike forthcoming)
   - Example: If model predicts vol = 0.025 AND RSI = 75 AND volume spike seen, short BTC expecting reversal

---

## Model 2: Volatility Expansion Classifier (XGBClassifier)

### How the Model is Trained


**Training Overview:**
The classifier is trained on historical BTC data, using engineered targets and a broad set of technical and volatility-related signals. Exact features and target engineering are intentionally omitted.

**Model Performance Metrics:**
- **Accuracy**: % of correct classifications (expansion vs. no expansion)
- **ROC-AUC**: area under the receiver-operator curve; measures ranking quality of probabilities (0.5 = random, 1.0 = perfect)
- **Confusion Matrix**: shows false positives (predicted expansion, didn't happen) vs. false negatives (missed expansion)

---

### What the Model Predicts

The classifier outputs a **probability** (0.0 to 1.0) that volatility will **expand** (exceed the 20-day realized volatility).

**Output Range & Interpretation:**

| Probability | Interpretation | Trade Signal |
|---|---|---|
| **0.0–0.25** | Very unlikely to expand | Short vol; sell straddles; expect vol contraction |
| **0.25–0.50** | Below 50-50; more likely to contract | Slight vol contraction bias; favorable for short vol |
| **0.50–0.75** | Above 50-50; more likely to expand | Vol expansion bias; favorable for long vol; buy straddles |
| **0.75–1.0** | Very likely to expand | Strong expansion signal; vol spike imminent; high risk |

**Example Predictions on Today's BTC Daily Chart:**

Suppose today's indicators:
- 20-day realized volatility: 0.018
- 5-day realized volatility: 0.016 (stable)
- RSI: 45 (neutral)
- Market regime: neutral

**Scenario 1: Quiet Market, No Expansion Expected**
- Model predicts: `expansion_probability = 0.18`
- Interpretation: 18% chance volatility exceeds 20-day baseline in next 3–5 days
- Trading action: Vol is likely to contract; consider selling call spreads / call ratio spreads; expect range-bound consolidation

**Scenario 2: Mixed Signals, Slight Expansion Bias**
- Current indicators show:
  - Volume surging
  - Return skew = -1.2 (slight downside bias)
  - ADX = 32 (trend strengthening)
- Model predicts: `expansion_probability = 0.62`
- Interpretation: 62% chance vol expands; slight bullish edge for long vol trades
- Trading action: Buy a straddle or call spread; expect breakout; tight stops below support

**Scenario 3: Extreme Alert — Vol Spike Imminent**
- Risk-off backdrop:
  - Fed taper expected tomorrow
  - `realized_vol_5 = 0.045` (already elevated)
  - `vol_of_vol = 0.032` (high vol of vol suggests regime shift)
  - Return skew = -3.5 (extreme downside tail risk)
- Model predicts: `expansion_probability = 0.87`
- Interpretation: 87% chance volatility explodes beyond 20-day baseline; crash risk
- Trading action: Buy protective puts; reduce leverage; short-term straddle; expect sharp down move within 48 hours

---

### Best Practices: How to Infer Trade Ideas

1. **Vol Expansion Long (Buy Volatility)**
   - When `expansion_probability > 0.65`:
     - Buy ATM straddle (long call + long put)
     - Buy call spread (upside vol) or put spread (downside vol protection)
     - Increase position size; expect larger moves
   - Example: If probability = 0.78 and realized vol = 0.015, buy a straddle expecting vol to jump to 0.025+; breakeven at current spot ± 1–2%

2. **Vol Contraction Short (Sell Volatility)**
   - When `expansion_probability < 0.35`:
     - Sell ATM straddle or iron condor (sell calls + sell puts)
     - Collect premium from vol contraction
     - Set tight stops if vol suddenly expands (rare but painful)
   - Example: If probability = 0.22 and implied vol is elevated, sell a 30 DTE straddle collecting 0.5% credit; expect vol to drop

3. **Regime Shift Detection**
   - Rising `expansion_probability` over several days (0.4 → 0.6 → 0.75) signals a **volatility regime shift**
   - Precedes breakouts or crashes
   - Action: Begin positioning for directional move; long calls if uptrend regime, long puts if downtrend regime
   - Example: Over 3 days, expansion prob rises steadily; on day 3 at 0.79, BTC breaks out up 15%; you caught the vol expansion regime shift

4. **Hedging Decision**
   - Use `expansion_probability` to decide whether to hedge:
     - If prob < 0.40: lightweight hedges (e.g. 5% portfolio in puts) sufficient; don't overpay for protection
     - If prob > 0.70: aggressive hedges (e.g. 15–20% in puts); vol spike likely
   - Saves money on unnecessary hedge premiums; ensures protection when needed
   - Example: If prob = 0.75, buy puts for 20% of portfolio; if prob = 0.20, skip hedges, allocate capital to growth

5. **Profit-Taking Signals**
   - When vol expansion actually occurs (realized vol exceeds 20-day baseline) **and** `expansion_probability` has dropped to <0.50:
     - Vol spike is exhausting; reversal likely
     - Exit long vol positions; take profits on straddles
   - Example: Vol spiked to 0.042, you held a straddle. Model now says expansion_probability = 0.32 (realized vol already expanded, now expects contraction). Exit straddle for profit

6. **Position Sizing Adjustment**
   - Base leverage/position size on expansion probability:
     - Prob = 0.20 → max leverage (higher Sharpe, expect calm)
     - Prob = 0.50 → moderate leverage (neutral, standard risk)
     - Prob = 0.80 → reduce leverage or go cash (high risk of sharp move)
   - Example: If prob = 0.85 and you're 5x leveraged long, reduce to 2x; if prob = 0.15 and you're 1x, increase to 3x

7. **Combining Both Models (Regressor + Classifier)**
   - **High Magnitude + High Expansion Probability**: Strongest vol expansion signal
     - Regressor says: `future_vol = 0.055`
     - Classifier says: `expansion_probability = 0.82`
     - Action: Aggressive long vol; buy expensive straddles; expect large moves up AND down
   
   - **High Magnitude + Low Expansion Probability**: Contradiction (rare); directional move more likely
     - Regressor says: `future_vol = 0.048`
     - Classifier says: `expansion_probability = 0.28`
     - Action: Vol will rise, but relative to current baseline it's already elevated. Look for directional breakout instead of expecting further vol expansion. Sell vol slightly OTM in direction of trend.
   
   - **Low Magnitude + Low Expansion Probability**: Best for range trading
     - Regressor says: `future_vol = 0.009`
     - Classifier says: `expansion_probability = 0.15`
     - Action: Range-bound consolidation; sell premium at extremes; trade intraday setups within range

---

## Model 3: Market Regime Model (GMMHMM)

### How the Model is Trained

- The model is a Gaussian Mixture Model Hidden Markov Model (GMMHMM) trained on historical BTC price data, using features engineered to capture returns, volatility, and regime shifts.
- **Feature Engineering:**
  - Log returns: `returns = log(close / close.shift(1))`
  - Price range: `(high - low) / close`
  - Adjusted range: For hourly data, range is normalized by hour-of-day average; for daily, raw range is used.
  - Rolling z-scores: 30-day (daily) or 720-period (hourly) z-scores of ADX and ATR to address non-stationarity.
  - Hourly data includes cyclical time features (sin/cos of hour) to capture session effects.
- **Training:**
  - The model is fit to the selected features after dropping NaNs and requiring a minimum data length (150 for daily, 1000 for hourly).
  - The GMMHMM is configured with 3 hidden states and 3 Gaussian mixtures per state, capturing bull, bear, and sideways regimes.
  - After training, states are mapped to 'bull', 'sideways', and 'bear' by sorting the mean returns of each state.
  - The model is saved and reused unless retraining is forced.

### How to Read the Outputs

- For each row, the model outputs the probability of being in each regime (bull, sideways, bear).
- A **market_regime_score** is computed as a weighted sum: `score = 15 × P(bear) + 50 × P(sideways) + 85 × P(bull)`
  - Higher scores indicate a stronger bull regime; lower scores indicate a bear regime; mid-range scores indicate sideways.
- The score is smoothed (EMA) to reduce noise, especially for hourly data.

**Example Outputs:**
- `market_regime_score ≈ 85`: Strong bull regime (high probability of uptrend)
- `market_regime_score ≈ 50`: Sideways regime (choppy, range-bound)
- `market_regime_score ≈ 15`: Bear regime (high probability of downtrend)

**Best Practices for Trade Ideas:**
- **High Score (>70):** Favor long BTC trades, especially if confirmed by trend/momentum indicators (e.g., EMA cross, ADX rising).
- **Low Score (<30):** Favor short BTC trades, or reduce/close longs. Confirm with bearish signals (e.g., RSI < 40, MACD bearish).
- **Mid Score (30–70):** Market is likely range-bound; reduce position size, avoid aggressive trend trades, or use mean-reversion strategies.
- Always use the regime score in confluence with other indicators (volume, RSI, MACD) for higher conviction.
- For hourly data, be aware of session effects and use the smoothed score to avoid reacting to noise.

---

## Advanced Regime Detection: GMMHMM Model

### How Regime Detection Challenges Are Addressed

1. **Non-stationarity (Rolling Z-Scores):**
   - Financial time series often change their mean and variance over time, making regime detection difficult.
   - The model uses rolling z-scores (30 days for daily, 720 periods for hourly) of ADX and ATR to normalize these features, allowing the model to adapt to changing market conditions and detect regime shifts more reliably.

2. **Volatility Clustering (ATR/Range):**
   - Volatility tends to cluster in financial markets (periods of high volatility are followed by more high volatility).
   - The model includes ATR and price range features to capture these clusters, helping distinguish between trending and choppy regimes.

3. **Noise (EMA Smoothing):**
   - Market data is noisy, especially at higher frequencies.
   - The regime score is smoothed using an Exponential Moving Average (EMA), with a longer memory for hourly data, to filter out short-term noise and highlight persistent regime changes.

4. **Fat Tails (GMMHMM):**
   - Returns in crypto markets often have fat tails (extreme moves are more common than in a normal distribution).
   - The GMMHMM (Gaussian Mixture Model Hidden Markov Model) can model each regime as a mixture of several Gaussian distributions, capturing both normal and extreme events. This is superior to a standard HMM, which assumes each regime is a single Gaussian and thus cannot represent fat tails or multi-modal behavior.

5. **Hour-of-Day Patterns (Cyclical Encoding & Normalization):**
   - Hourly BTC data exhibits strong session effects (certain hours are more volatile or trend-prone).
   - The model encodes hour-of-day using sine and cosine transforms, and normalizes volatility by typical values for each hour, allowing it to learn and adjust for these cyclical patterns.

### Why GMMHMM is Superior to Standard HMM
- **Standard HMM:** Assumes each hidden state (regime) is described by a single Gaussian distribution, which cannot capture fat tails, multi-modal returns, or regime-specific volatility clusters.
- **GMMHMM:** Each regime is modeled as a mixture of multiple Gaussians, allowing for:
  - Better fit to real-world financial data with fat tails and outliers
  - More accurate regime assignment during extreme market events
  - Improved detection of transitions between regimes, especially in noisy or volatile periods

### Understanding the 3 Regimes and the Market Regime Score
- The model identifies three regimes, mapped by the average returns of each state:
  - **Bull Regime:** High average returns, typically strong uptrends, but can also mean strong trending (not always up)
  - **Sideways Regime:** Near-zero average returns, choppy or range-bound markets
  - **Bear Regime:** Low or negative average returns, typically downtrends, but can also mean strong trending (not always down)
- The **market_regime_score** is a weighted sum:
  - `score = 15 × P(bear) + 50 × P(sideways) + 85 × P(bull)`
  - **Score ≈ 85:** High probability of a strong trending regime (often up, but could be a strong downtrend if the bull state is mapped to negative returns in current market)
  - **Score ≈ 50:** High probability of a sideways regime (range-bound, low conviction)
  - **Score ≈ 15:** High probability of a strong trending regime (often down, but could be a strong uptrend if the bear state is mapped to positive returns in current market)
- **Important:** The regime labels (bull/bear) are mapped by average returns, but in some market conditions, a 'bull' state could mean a strong downtrend if the data is dominated by negative returns. Always check the actual return profile of each regime in your current dataset.

**Best Practices:**
- Use the regime score as a measure of trend strength, not just direction. High or low scores mean strong trends (up or down); mid scores mean choppy/range-bound.
- Always confirm regime interpretation with recent price action and other indicators (e.g., if score is high but price is falling, the 'bull' regime may be mapped to a strong downtrend in current conditions).
- For hourly data, pay extra attention to session effects and use the smoothed score to avoid reacting to short-term noise.

---

## Summary: Quick Decision Tree

```
Check today's predictions:
├─ expansion_probability > 0.70 AND future_vol > 0.040
│  └─ → RISK-OFF: Vol explosion. Reduce leverage, buy puts, sell calls.
├─ expansion_probability > 0.60 AND 0.025 < future_vol < 0.040
│  └─ → Long Vol: Buy straddles, vol ratio spreads. Position for breakout.
├─ 0.40 < expansion_probability < 0.60 AND 0.015 < future_vol < 0.025
│  └─ → Neutral Vol: Standard trading. Use options for directional bets.
├─ expansion_probability < 0.35 AND future_vol < 0.015
│  └─ → Short Vol: Sell straddles, iron condors. High leverage OK.
└─ expansion_probability < 0.20 AND future_vol < 0.010
   └─ → Max Risk-On: Range-bound calm. Leverage up for directional trades.
```

---

## Technical Notes

- **Log-Transform**: The regressor target is log-transformed to handle the wide range of volatility values (0.005 to 0.15+). The model outputs are automatically inverse-transformed (expm1) for readability.
- **Rolling Features Lag**: Features like SMA-200, EMA-200 require ~200 days of history. Backfill skips the first 50 rows to avoid NaN-induced data loss.
- **Time-Series Split**: Models are trained/tested on time-ordered data to avoid lookahead bias.
- **Feature Importance**: SHAP analysis shows realized volatility (vol_5, vol_10) and regime signals dominate predictions; price alone is weak.
- **Retraining Frequency**: Models should be retrained monthly or after major market regimes (e.g., post-Fed decision, post-major news) to adapt to changing market structure.
