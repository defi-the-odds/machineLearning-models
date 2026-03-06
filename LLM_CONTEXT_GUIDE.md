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

## NOTES
- All rules and interpretations are deterministic and context-rich for machine parsing.
- Use confluence of model outputs and technical indicators for best results.
- This guide omits feature engineering details for security.
