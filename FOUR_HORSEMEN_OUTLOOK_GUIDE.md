# Four Horsemen outlook metrics — interpretation & look-back windows

Predictions apply to the **next single candle** (one daily bar if models are daily, one hourly bar if hourly).

## How to read `metric` + `*_conf`

Each model outputs:

- **`tail_risk`**, **`high_vol`**, **`regime_change`**, **`regime_emergence`**: **`0` or `1`** (the predicted class for the next bar).
- **`tail_risk_conf`**, **`high_vol_conf`**, **`regime_change_conf`**, **`regime_emergence_conf`**: **`P(class = 1)`** — the model’s estimated probability that the **positive** event happens on the next bar.

| You see | Meaning |
|---------|---------|
| **`*_conf` near `1.0`** (e.g. ≥ 0.9) | **High confidence that class 1 occurs** next bar. Usually paired with **`metric = 1`**. |
| **`*_conf` near `0.0`** (e.g. ≤ 0.1) | **High confidence that class 0 occurs** next bar — i.e. **very low P(class 1)**. Usually paired with **`metric = 0`**. |
| **`*_conf` near `0.5`** | **Low confidence / coin-flip** on class 1. **`metric` may still be 0 or 1** (e.g. threshold at 0.5), but the signal is **weak** — treat as unreliable. |

So: **strong “yes”** → **`metric = 1`** and **`*_conf` high**. **Strong “no”** → **`metric = 0`** and **`*_conf` ≈ 0.0**. **Weak “yes”** → **`metric = 1`** but **`*_conf` ~ 0.5**.

| Metric column | Confidence column (`P(class 1)`) |
|---------------|----------------------------------|
| `tail_risk` | **`tail_risk_conf`** |
| `high_vol` | **`high_vol_conf`** |
| `regime_change` | **`regime_change_conf`** |
| `regime_emergence` | **`regime_emergence_conf`** |

- **`tail_risk_conf`** — *P(tail-sized move next bar)* — **not** direction.
- **`high_vol_conf`** — *P(high vol next bar vs 20/50 benchmark)*.
- **`regime_change_conf`** — *P(low→high vol transition next bar)*.
- **`regime_emergence_conf`** — *P(10-bar vol spike vs 30-bar median next bar)*.

All comparisons below use **per-bar returns** (typically close-to-close) unless noted.

---

## 1. Tail risk (`tail_risk` + `tail_risk_conf`)

| | |
|---|---|
| **Class 1** | Next candle’s **absolute return** is **larger than the 90th percentile** of absolute returns over the **prior 20 candles**. |
| **Look-back for the threshold** | **20 candles** — rolling 90th percentile of \|return\| over those 20 bars. |
| **Feature context** | **60-candle** rolling quantiles of absolute returns; **20-candle** mean absolute return. |

**`tail_risk = 1`** and **`tail_risk_conf` high (e.g. ≥ 0.9):** Strong belief in an **unusually large move** (up *or* down) next bar.

**`tail_risk = 0`** and **`tail_risk_conf` ≈ 0.0:** Strong belief in a **normal-sized** move next bar.

**`tail_risk = 1`** but **`tail_risk_conf` ~ 0.5:** Model says **tail** but **does not trust it** — weak tail signal.

---

## 2. High vol (`high_vol` + `high_vol_conf`)

| | |
|---|---|
| **Class 1** | **Next candle** in **high-vol regime**: **20-candle rolling std of returns** (end of next bar) **above the median** of that vol series over **prior 50 candles**. |
| **Look-back** | **20 bars** vol; **50 bars** for median divider. |

**`high_vol = 1`** + **`high_vol_conf` high:** Strong belief next bar is **choppy / wide** vs recent norm.

**`high_vol = 0`** + **`high_vol_conf` ≈ 0.0:** Strong belief next bar is **calmer** vs that benchmark.

---

## 3. Regime change (`regime_change` + `regime_change_conf`)

| | |
|---|---|
| **Class 1** | **Next bar** is **high vol** (20/50 rule) while **current bar** is **not** — **low→high** transition. |
| **Look-back** | Same as high vol: **20 / 50**. |

**`regime_change = 1`** + **`regime_change_conf` high:** Strong belief in **volatility arriving** on the next bar.

**`regime_change = 0`** + **`regime_change_conf` ≈ 0.0:** Strong belief **that transition does not** happen next bar.

---

## 4. Regime emergence (`regime_emergence` + `regime_emergence_conf`)

| | |
|---|---|
| **Class 1** | **Next bar**: **10-candle rolling std of returns** **>** **median** of that vol over **prior 30 candles**. |
| **Look-back** | **10-bar** short vol; **30-bar** median benchmark. |

**`regime_emergence = 1`** + **`regime_emergence_conf` high:** Strong belief **short-horizon vol spikes** next bar.

**`regime_emergence = 0`** + **`regime_emergence_conf` ≈ 0.0:** Strong belief **that spike does not** occur next bar.

---

## Likely combinations (coherent stories)

Check **both** `metric` and **`*_conf`** — ignore legs where **`*_conf` ~ 0.5**.

| Combination | Plain-language story |
|-------------|----------------------|
| **high_vol=1, regime_change=0, tail_risk=0** (with meaningful conf on each) | Volatile without being the **first** high-vol day; size may be “normal” for vol. |
| **regime_change=1, high_vol=1** + **high `*_conf`** | **First high-vol day** and classified high vol. |
| **regime_emergence=1, high_vol=1** + **high `*_conf`** | Short-horizon vol jump **and** 20/50 high vol. |
| **tail_risk=1, high_vol=1** + **high `*_conf`** | **Event-style** next bar. |
| **All four = 0** + **`*_conf` ≈ 0** on each | **Quiet** skew next bar. |
| **regime_emergence=1, tail_risk=0** | Vol heating in **10-bar** sense without necessarily a **90th %-ile** \|return\|. |

---

## Strong signals (actionable themes)

Use **`*_conf` ≥ 0.9** (or similar) for **class-1** themes, and **`*_conf` ≤ 0.1** with **`metric = 0`** for **class-0** / calm themes.

| Theme | Pattern | Expected next candle(s) |
|-------|---------|-------------------------|
| **Volatility event** | **tail_risk=1**, **`tail_risk_conf` high** + **high_vol=1**, **`high_vol_conf` high** | Large range + elevated vol. |
| **Regime break** | **regime_change=1**, **`regime_change_conf` high** + **high_vol=1** | First **20/50** high-vol day. |
| **Vol acceleration** | **regime_emergence=1**, **`regime_emergence_conf` high** + **regime_change=1** | Short vol spike + low→high transition. |
| **Full stress stack** | All four **= 1**, all four **`*_conf` high** | Maximum **event + regime** alignment. |
| **Calm baseline** | All four **= 0**, all four **`*_conf` ≈ 0.0** | Strong belief in **quiet** next bar. |

**Caution:** Definitions overlap; **backtest** hit rates, not intuition alone.

---

## Trader actions (ideas — not advice)

Examples use **spot/perp/futures** language: **Long** = buy to open; **Short** = sell to open. **Buy-stop** / **sell-stop** = stop orders that trigger when price crosses a level (breakout-style entries or protective exits).

| Condition | Rationale | Example actions |
|-----------|-----------|------------------|
| **`tail_risk = 1`** + **`tail_risk_conf` > 0.9** | High **P(large \|move\|)** next bar; direction unknown. | Place **buy-stop** above the recent high / range and **sell-stop** below the recent low; use **OCO** or size so only one side can hurt you. Whichever fires, you are **long** or **short** in the direction of the break. |
| **`tail_risk = 0`** + **`tail_risk_conf` ≈ 0.0** | High **P(no tail)** — range-like next bar. | Prefer **limit** entries near range edges: **long** near support, **short** near resistance; avoid wide **buy-stop/sell-stop** brackets. Tighter stops; scale **long/short** size to smaller expected range. |
| **`tail_risk = 1`** but **`tail_risk_conf` ~ 0.5** | Weak conviction on tail. | Narrower stops on any open **long**/**short**; delay new **buy-stop/sell-stop** brackets until **`*_conf`** clears. |
| **`high_vol = 1`** + **`high_vol_conf` > 0.9** | High **P(wide, choppy bar)**. | **Widen** stop distance on **long** and **short** (or reduce size). Avoid stacking multiple **buy-stop**/**sell-stop** triggers too tight—whipsaw risk. |
| **`high_vol = 0`** + **`high_vol_conf` ≈ 0.0** | High **P(calm)**. | **Trend-following** **long** or **short** with normal stops; **buy-stop** above pullback highs (long bias) or **sell-stop** below rally lows (short bias) for continuation entries. |
| **`regime_change = 1`** + **`regime_change_conf` > 0.9** | High **P(break from quiet into volatile)**. | **Buy-stop** above consolidation for **long** breakout; **sell-stop** below for **short** breakdown. Cancel the unfilled side once one triggers. |
| **`regime_emergence = 1`** + **`regime_emergence_conf` > 0.9** | High **P(sudden expansion in short-horizon vol)**. | Same as regime change: favor **stop**-triggered **long**/**short** (breakout), not fading the first spike. Wider initial stops after fill. |
| **Full stack** + **all `*_conf` > 0.9** | Strongest **event** alignment. | Full **buy-stop** + **sell-stop** bracket; or stay **flat** until after the candle if you do not want gap/slip risk. If one side fills, manage the **long** or **short** with a hard stop. |
| **All zeros + all `*_conf` ≈ 0** | Strong **calm** alignment. | Prefer **long** in uptrend (pullback limits, **buy-stop** on reclaim) or **short** in downtrend (**sell-stop** on breakdown). Mean-reversion: **short** resistance, **long** support—with stops just beyond the level. |

**`*_conf` ~ 0.5** on any leg → treat that leg as **no edge** for sizing.

### Preferential strategies (by regime)

| Regime (from metrics + `*_conf`) | Preferential approach | Order types to emphasize |
|----------------------------------|----------------------|---------------------------|
| **Tail risk high** (`tail_risk=1`, `tail_risk_conf` high) | **Breakout / bracket** — capture direction only after price proves it | **Buy-stop** + **sell-stop** (OCO-style); then **long** or **short** with stop |
| **Tail risk low** (`tail_risk=0`, `tail_risk_conf` ≈ 0) | **Range / fade** — smaller expected move | **Limit** **long** at lows, **limit** **short** at highs; tight protective **sell-stop** on longs, **buy-stop** on shorts |
| **High vol predicted** | **Smaller size, wider stops** — survive noise | Stops further from entry on **long**/**short**; fewer concurrent stop triggers |
| **Calm predicted** (high_vol=0, conf ≈ 0) | **Trend continuation** | **Buy-stop** for **long** add-ons in uptrend; **sell-stop** for **short** add-ons in downtrend |
| **Regime change / emergence high** | **First move wins** — trade the break | **Buy-stop** (bull break) or **sell-stop** (bear break); avoid counter-trend **long**/**short** into the open |
| **Full calm stack** (all 0, conf ≈ 0) | **Directional drift or mean-reversion** (match your higher-timeframe bias) | **Long** with **buy-stop** on strength; **short** with **sell-stop** on weakness; or range limits as above |

**Not investment advice.** Size every **long**/**short** so a single stop-out is acceptable.

---

## Disclaimer

**Not investment advice.** Statistical targets from training; markets can diverge. Use execution, liquidity, and your own risk limits.
