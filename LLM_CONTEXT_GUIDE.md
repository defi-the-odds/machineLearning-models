# LLM_CONTEXT_GUIDE

## Purpose
This document is designed for machine parsing by OpenClaw or other AI agents. It provides deterministic, context-rich guidance for generating trade ideas based on model outputs.

---

## MODEL_OUTPUT_SCHEMA

### Volatility Regressor
- output: future_vol (float)
- interpretation:
  - 0.005–0.010: Low volatility (calm, range-bound)
  - 0.010–0.020: Normal volatility (typical daily moves)
  - 0.020–0.040: Elevated volatility (active trading)
  - 0.040–0.080+: High volatility (directional moves, macro events)

### Expansion Classifier
- output: expansion_probability (float, 0.0–1.0)
- interpretation:
  - 0.0–0.25: Very unlikely to expand (short vol, expect contraction)
  - 0.25–0.50: More likely to contract (slight contraction bias)
  - 0.50–0.75: More likely to expand (expansion bias, buy vol)
  - 0.75–1.0: Very likely to expand (strong expansion signal)

### Regime Model
- output: market_regime_score (float)
- interpretation:
  - ≈85: Bull regime (strong uptrend)
  - ≈50: Sideways regime (choppy/range-bound)
  - ≈15: Bear regime (downtrend)

---

## TRADE_IDEA_RULES

- If future_vol > realized_vol_20 and RSI < 30:
    action: Long BTC, expect breakout
- If future_vol < 0.015:
    action: Increase position size, favor long/short BTC
- If future_vol > 0.04:
    action: Reduce position size, risk-off
- If expansion_probability > 0.65:
    action: Buy straddle, expect vol jump
- If expansion_probability < 0.35:
    action: Sell straddle, expect contraction
- If market_regime_score > 70 and future_vol < 0.02:
    action: High confidence in long BTC
- If market_regime_score < 30 and future_vol > 0.03:
    action: Short BTC or close longs

---

## EXAMPLES

- Scenario: Calm Market
  - Inputs: future_vol = 0.012, RSI = 45, market_regime_score = 50
  - Output: Neutral, standard trading

- Scenario: Elevated Risk
  - Inputs: future_vol = 0.032, expansion_probability = 0.62, market_regime_score = 32
  - Output: Buy straddle, expect breakout

- Scenario: Vol Spike Imminent
  - Inputs: future_vol = 0.068, expansion_probability = 0.87, market_regime_score = 15
  - Output: Buy puts, reduce leverage, expect sharp down move

---

## HIGH_CONFIDENCE_METRICS

### Definition
High-confidence metrics are combinations of model outputs and datapoints that strongly indicate a specific market condition or trading opportunity. These signals are derived from both Daily and Hourly timeframes and are suitable for various trading strategies.

### Examples

- **Daily Timeframe: Trend-Following Entry**
  - Inputs: market_regime_score > 80, future_vol < 0.02, expansion_probability < 0.4
  - Signal: Strong bull regime, low volatility, low expansion risk
  - Strategy: Enter long position; add to existing longs; hold through minor pullbacks

- **Daily Timeframe: Volatility Breakout**
  - Inputs: future_vol > 0.04, expansion_probability > 0.7, market_regime_score > 60
  - Signal: High expected volatility, high probability of expansion, bullish regime
  - Strategy: Enter new long position; add to longs on confirmation; reduce shorts

- **Hourly Timeframe: Mean Reversion**
  - Inputs: market_regime_score between 40–60, future_vol < 0.01, expansion_probability < 0.3
  - Signal: Sideways regime, low volatility, low expansion risk
  - Strategy: Reduce exposure; take profits; avoid new positions

- **Hourly Timeframe: Momentum Scalping**
  - Inputs: market_regime_score > 75, expansion_probability > 0.6, recent volume spike
  - Signal: Bull regime, high expansion probability, momentum confirmed by volume
  - Strategy: Enter short-term long trades; add to longs quickly; use trailing stops

- **Daily Timeframe: Risk-Off Hedging**
  - Inputs: future_vol > 0.06, expansion_probability > 0.85, market_regime_score < 30
  - Signal: Bear regime, extreme volatility, high expansion risk
  - Strategy: Reduce exposure; hedge existing longs; avoid new longs; consider short positions

- **Hourly Timeframe: Volatility Fade**
  - Inputs: future_vol spikes > 0.03, expansion_probability drops < 0.4 after spike
  - Signal: Volatility spike exhausted, expansion risk receding
  - Strategy: Take profits on longs; reduce exposure; avoid new trades until conditions stabilize

### Strategy Suitability
- **Trend-Following:** Use high regime scores and low volatility for entry; add to longs, hold through pullbacks
- **Breakout Trading:** Seek high future_vol and expansion_probability; enter or add to longs, reduce shorts
- **Mean Reversion:** Target mid regime scores and low volatility; reduce exposure, take profits
- **Scalping:** Use hourly signals with high regime and expansion probability; enter short-term longs, add quickly
- **Hedging/Risk-Off:** Enter when volatility and expansion risk are extreme, and regime is bearish; reduce exposure, hedge longs, consider shorts
- **Volatility Fading:** Take profits after volatility spikes and expansion probability drops; reduce exposure

---

## NOTES
- All rules and interpretations are deterministic and context-rich for machine parsing.
- Use confluence of model outputs and technical indicators for best results.
- This guide omits feature engineering details for security.
