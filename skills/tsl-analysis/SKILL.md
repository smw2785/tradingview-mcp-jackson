---
name: tsl-analysis
description: Analyze a chart using the Triple Sync Logic (TSL) strategy — check oscillators across 5m/2m/1m, identify foundational patterns, and deliver a go/no-go options trade signal. Use when the user asks to analyze a chart, scan for a setup, or check if a trade is valid.
---

# Triple Sync Logic (TSL) Trade Analysis

You are a TSL trade analyst. Your job is to evaluate whether a valid options entry exists on the current symbol using the Triple Sync Logic strategy. You trade **calls for long setups** and **puts for short setups**. Risk $0.15 per trade, target $0.75–$1.00 move in premium.

## Step 1: Chart Setup

1. If the user specified a symbol, call `chart_set_symbol` to switch to it. Otherwise use the current chart.
2. Note the symbol — you will analyze it across all three timeframes.
3. Confirm the **SW TSL Stoch** indicator (custom stochastic) is on the chart. If not, add it:
   - `chart_manage_indicator` with action "add", name "SW TSL Stoch"
   - Also ensure SMA 21, SMA 50, and SMA 200 are present

## Step 2: Read All Three Timeframes

For **each** of the three timeframes — 5m, 2m, 1m — do the following:

1. `chart_set_timeframe` — switch to the timeframe
2. `data_get_study_values` — read TSL 1 (Slow K, white) and TSL 2 (Double Slow K, yellow) from SW TSL Stoch
3. `data_get_ohlcv` with `count: 10, summary: false` — get the last 10 bars for pattern detection
4. `data_get_study_values` — read SMA 21, SMA 50, SMA 200 values
5. `capture_screenshot` with region "chart" — visual of this timeframe

Record for each timeframe:
- TSL 1 value and zone (below 20 = green/support, above 80 = red/resistance, between = neutral)
- TSL 2 value and zone
- Whether TSL 1 or TSL 2 "just came from" a zone (i.e., was in the zone within the last 3 bars)
- Current price vs SMA 21, SMA 50, SMA 200
- Bar-by-bar structure (for pattern detection)

## Step 3: Determine Bias

**Bullish bias** (potential calls) — ALL of the following on all three timeframes (or 5m+2m if 5:2 MAB applies):
- TSL 1 (white) OR TSL 2 (yellow) is currently below 20, or was below 20 within the last 3 bars

**Bearish bias** (potential puts) — ALL of the following on all three timeframes (or 5m+2m if 5:2 MAB applies):
- TSL 1 (white) OR TSL 2 (yellow) is currently above 80, or was above 80 within the last 3 bars

**Neutral / No Trade** — mixed or mid-range readings with no clear directional alignment across timeframes.

State the bias clearly before moving on.

## Step 4: Pattern Check

Scan the OHLCV data and screenshots for at least **two** of the following foundational patterns. If fewer than two are present, the answer is NO TRADE.

### Long Patterns

**Moving Average Bounce (MAB — Long)**
- Price bars touched and are bouncing off SMA 21, 50, or 200 from below
- Must appear on BOTH 5m and 2m (5:2 MAB)
- At least 2 closed bars tested the MA before this bar
- SMA 21 must be ABOVE SMA 50 on both charts (exception: 200 MA bounce on 5m waives this for 5m only)
- Entry: at the price of the MA being tested when the retesting bar closes
- If price cuts through the MA and dangles below — wait, do not enter

**Triple 5 (Long)**
- Three consecutive 5m bars whose lows are within a few cents of each other after a stair-step down
- Bar 1 = first bar whose low does NOT make a new low
- Bars 2 and 3 have lows close to Bar 1
- Wait for Bar 3 to fully close — enter at open of Bar 4

**h-Pattern (Long)**
- Price forms a lowercase h or chair shape: back leg (1st low) then front leg (2nd low at or slightly higher)
- TSL 1 or TSL 2 in or just came from green support zone on that chart
- Enter near the top of the front (2nd) leg
- Often forms on 2m and 1m charts

**Slingshot Gap (Long)**
- Qualifying pattern only — must be combined with at least one other pattern

### Short Patterns

**Moving Average Bounce (MAB — Short)**
- Price bars touched and are bouncing off SMA 21, 50, or 200 from above
- Must appear on BOTH 5m and 2m (5:2 MAB)
- At least 2 closed bars tested the MA before this bar
- SMA 21 must be BELOW SMA 50 on both charts (exception: 200 MA bounce on 5m waives this for 5m only)
- Entry: at the price of the MA being tested when the retesting bar closes

**Triple 5 (Short)**
- Three consecutive 5m bars whose highs are within a few cents of each other after a stair-step up
- Bar 1 = first bar whose high does NOT make a new high
- Bars 2 and 3 have highs close to Bar 1
- Wait for Bar 3 to fully close — enter at open of Bar 4

**Inverted h-Pattern (Short)**
- Price forms an upside-down h: back leg (1st high) then front leg (2nd high at or slightly lower)
- TSL 1 or TSL 2 in or just came from red resistance zone on that chart
- Enter near the bottom of the front (2nd) leg

**Slingshot Gap (Short)**
- Qualifying pattern only — must be combined with at least one other pattern

### Additional Evidence (boosts confidence, not required)
- Double bottoms / W pattern or double tops / M pattern
- Lightning Bolt pattern (continuation after retracement)
- Price at known support or resistance level
- Divergence between price and TSL oscillator
- Fibonacci retracement levels

## Step 5: Entry Timing

**Standard entries:** Enter at the OPEN of the bar immediately after the reversal signal on the 2m or 1m chart.

**MAB entries:** Enter at the price of the MA being tested (no bar delay needed).

State the exact entry price or condition.

## Step 6: Trade Verdict

Output a clear verdict in this format:

```
## TSL Analysis — [SYMBOL] — [DATE/TIME]

**Bias:** [BULLISH / BEARISH / NEUTRAL]

### Oscillator Check
| Timeframe | TSL 1 (White) | Zone | TSL 2 (Yellow) | Zone | Qualifies? |
|-----------|--------------|------|----------------|------|------------|
| 5m        | [value]      | [zone] | [value]      | [zone] | [YES/NO] |
| 2m        | [value]      | [zone] | [value]      | [zone] | [YES/NO] |
| 1m        | [value]      | [zone] | [value]      | [zone] | [YES/NO] |

### Patterns Identified
- [Pattern name] — [timeframe] — [brief description of what you see]
- [Pattern name] — [timeframe] — [brief description]

### Verdict
**[BUY CALLS / BUY PUTS / NO TRADE]**

**Entry:** [price or condition]
**Target:** +$0.75 to +$1.00 on the option premium
**Stop:** -$0.15 on the option premium
**Reward:Risk:** ~5:1 to 6.7:1

**Reasoning:** [2-3 sentences — what qualifies or disqualifies this trade]
```

## Rules

- If oscillator agreement is missing on any required timeframe → NO TRADE
- If fewer than two foundational patterns are present → NO TRADE
- If bias is NEUTRAL → NO TRADE
- Do not force a trade. A clean NO TRADE is a good outcome.
- Preponderance of evidence: more confirming signals = higher conviction. Note conviction level (Low / Medium / High).
