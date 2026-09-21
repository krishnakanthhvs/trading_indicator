# Intraday Edge V2

Intraday Edge V2 (`IE v2.0`) is a TradingView overlay indicator written for Pine Script v6. It combines a Lorentzian-distance machine-learning signal model with session levels, market structure, liquidity references, and a visual entry/stop/target plan.

**Author:** Krishna Kanth  
**Source:** [Intraday Edge](Intraday%20Edge)

## Installation

1. Open the `Intraday Edge` source file in this repository and copy its contents.
2. Open TradingView's Pine Editor on a chart and paste the source into a new indicator.
3. Save the script and add it to the chart.
4. Open the indicator settings to configure signals, session times, and optional overlays.

The script imports `jdehorty/MLExtensions/2` and `jdehorty/KernelFunctions/2`. These TradingView libraries must be accessible for compilation. No local package installation or API key is required by this source.

## Feature overview

| Feature | Function | Default |
| --- | --- | --- |
| ML classification | Directional signals using configurable technical features and Lorentzian distance | Enabled |
| Signal badges | SB/B/BI and SS/S/SI strength codes | Enabled |
| Signal preview | Live badge preview near candle close | 30 seconds |
| Next-candle entry dot | Marks the next candle's opening price after a signal | Enabled |
| Kernel regression | Trend estimate and optional signal filter | Enabled |
| Trade plan | Entry, stop loss, and three risk-based targets | Enabled |
| Pivot points | Six calculation types with configurable anchors and levels | Enabled |
| Daily Open Point | Session opening-price reference | Enabled |
| Weekly Open | Weekly opening-price reference | Disabled |
| Opening range (ORB) | Session opening-range high, low, and midpoint | Disabled; 5 minutes |
| Liquidity sweeps | Confirmed swing sweeps with rejection and volume checks | Disabled |
| BOS / CHoCH | Confirmed continuation and reversal structure breaks | Enabled |
| Equal highs / lows | EQH and EQL liquidity references | Enabled |
| Order blocks | Supply/demand zones from chart or selected timeframe | Enabled |
| Candle patterns | Educational pattern letters with hover descriptions | Disabled |
| Manual support/resistance | Nearest user-supplied price levels | Enabled |
| ML bar colors | Optional prediction-based candle tint | Enabled |
| ML trade statistics | Calibration table for ML entries and exits | Enabled |
| Intraday Edge dashboard | Trend, bias, option, signal, market, strength, volume, liquidity, ORB, and risk | Enabled |
| Alerts | Eight ML entry, exit, and kernel conditions | Available |
| Disclaimer watermark | Analysis-only notice at the bottom of the chart | Enabled |

## ML model and filters

The model uses an approximate neighbor search with Lorentzian distance and sums the selected historical direction votes. Its training design uses a four-bar horizon. This is an on-chart model; the source does not connect to an external AI service.

General controls include:

- **Source:** close by default.
- **Neighbors Count:** 8 by default; adjustable from 1 to 100.
- **Max Bars Back:** 2,000 by default; adjustable from 250 to 5,000.
- **Include Full History:** off by default; expands the search to earlier loaded chart history at additional computational cost.
- **Feature Count:** 2–5 active features; 5 by default.
- **Color Compression:** adjusts visual color intensity.

Each feature slot can use RSI, WaveTrend (WT), CCI, or ADX, with configurable parameters:

| Slot | Default feature | Parameter A | Parameter B |
| --- | --- | --- | --- |
| 1 | RSI | 14 | 1 |
| 2 | WT | 10 | 11 |
| 3 | CCI | 20 | 1 |
| 4 | ADX | 20 | 2 |
| 5 | RSI | 9 | 1 |

Parameter B applies only where supported by the feature; the ADX helper uses Parameter A.

| Signal filter | Default | Setting |
| --- | --- | --- |
| Volatility | On | Library volatility filter |
| Market regime | On | Threshold −0.1 |
| ADX | Off | Threshold 20 |
| EMA | Off | Period 200; directional price filter |
| SMA | Off | Period 200; directional price filter |
| Kernel | On | Directional kernel filter |

Pivot points, opening levels, ORB, market structure, sweeps, candle patterns, order blocks, and manual levels are visual references. They do not add conditions to the ML entry model.

## Kernel regression and ML exits

The indicator uses Rational Quadratic and Gaussian kernel estimates for trend direction and crossover comparisons. It can display the kernel estimate, filter entries with it, and optionally smooth direction changes through crossover logic.

Defaults are a lookback window of **8**, relative weighting of **8**, regression level of **25**, and lag of **2**. Enhanced smoothing is off.

- **Show Default Exits** displays exit crosses and is off by default.
- The strict exit logic uses the model's four-bar holding rules and signal conditions.
- **Use Dynamic Exits** optionally uses kernel direction changes for exits.
- Dynamic exits apply only when the EMA filter, SMA filter, and enhanced kernel smoothing are all disabled; otherwise the script uses strict exits.

These ML exit events are separate from the visual trade plan's stop-loss and target touches.

## Signal badges and timing

Signal strength is calculated as `abs(prediction) / neighborsCount`:

| Buy | Sell | Meaning | Vote strength |
| --- | --- | --- | --- |
| SB | SS | Strong | At least 80% |
| B | S | Standard | At least 45%, below 80% |
| BI | SI | Low confidence / ignore category | Below 45% |

Strength measures model vote agreement, **not a probability of profit**. The standard badge tooltip's “50-50” wording is a category label, not a measured win rate.

- **30-Second Preview:** shows strength badges during the final part of a live time-based candle. Lead time is adjustable from 5 to 240 seconds. A preview can change or disappear before the candle closes and requires a fresh market update to appear.
- **Confirmed Candle Close:** waits for candle confirmation before drawing strength badges.
- Badge spacing is adjustable in ATR units; the default history limit is 75 badges.
- Low-confidence badges can be hidden without removing their underlying signals, alerts, entry dots, or trade plans.
- Turning off strength codes uses plain directional markers. In the current implementation, these plain markers do not use the badge timing gate.
- An optional orange dot marks the next candle's open after any ML entry signal. Its color and size are configurable; the default size is Tiny.

Optional bar coloring offers Default and Solid schemes, confidence-gradient control, and color compression. These settings change presentation rather than signal logic.

## Signal trade plan

The latest confirmed ML entry signal creates a visual plan on the next candle:

| Component | Long plan | Short plan |
| --- | --- | --- |
| Entry | Next candle's open | Next candle's open |
| Stop loss | Signal candle low minus ATR buffer | Signal candle high plus ATR buffer |
| Risk (R) | Entry minus stop | Stop minus entry |
| Target | Entry plus selected R multiple | Entry minus selected R multiple |

The default stop buffer is **0.25 × ATR(14)** using the signal candle's ATR, and targets are **1R, 2R, and 3R**. Each multiple is adjustable.

- Entry, SL, TP1, TP2, and TP3 have price labels and configurable line style.
- Lines project 24 chart bars by default, adjustable from 5 to 200. This controls drawing length, not a timed trade exit.
- Reached targets can receive a check mark and a grey label.
- By default, the plan is removed after all targets are reached or the stop is touched.
- With stop-removal enabled, a candle touching both stop and target is handled as a stop first because the intrabar sequence is unknown.
- A new signal replaces the previous plan, including signals whose low-confidence badges are hidden.
- A gap producing risk no greater than one minimum price tick prevents creation of the new plan.

The plan draws reference prices; it does not submit orders or simulate brokerage fills.

## Pivot points

Settings are organized as **7.1 • PIVOT POINTS**, **7.2 • PIVOT POINTS — LABELS**, and **7.3 • PIVOT POINTS — LEVELS**. Other sections already have unique numbers.

Supported types are **Traditional, Fibonacci, Woodie, Classic, DM, and Camarilla**. The default is Traditional with an Auto anchor and one historical pivot set.

- Anchors: Auto, Daily, Weekly, Monthly, Quarterly, Yearly, Biyearly, Triyearly, Quinquennially, and Decennially.
- Auto uses daily pivots on intraday charts up to 15 minutes, weekly pivots on higher intraday charts, monthly pivots on daily charts, and yearly pivots otherwise.
- Daily-based values are enabled by default; intraday-based values are also supported.
- P and supported S1–S5/R1–R5 levels have individual visibility and color controls. Available levels depend on the calculation type.
- Label names, prices, left/right placement, line width, and historical retention are configurable.

Pivot calculation requires enough price history for the selected anchor. Daily-based and intraday-based calculations can differ when their source OHLC data differs.

## Session profiles and opening levels

The following are the script's configured session presets, interpreted in the **symbol's exchange timezone**:

| Profile | Configured hours |
| --- | --- |
| Indices / Equity | 09:15–15:30 |
| Commodities | 09:00–23:30 |
| Commodities Winter | 09:00–23:55 |
| MCX International Agri | 09:00–21:00 |
| MCX Domestic Agri | 09:00–17:00 |
| Crypto | 24 hours |
| Custom | User-defined start/end hours and minutes |

**Auto** selects Crypto for crypto symbols, Commodities for the `MCX` exchange prefix, and Indices / Equity otherwise. It does not automatically select the winter or agriculture presets. Select the appropriate preset or custom hours for the instrument.

The **Daily Open Point** captures the first available chart bar's open when a selected session begins. Its line is bullish-colored above the open, bearish-colored below it, and uses At Open Color at equality.

The optional **Weekly Open** uses the weekly opening price. Its colors use configurable bullish/bearish point thresholds, both 10 points by default; prices inside that band use the neutral color. Those thresholds do not control Daily Open Point coloring.

Both features provide label, value, color, and width controls. Session-only drawings are cleared after the configured close when the script evaluates the closed-session state.

Visibility rules:

- Session opening levels and ORB are intended for intraday charts.
- Crypto session levels, including ORB, are hidden on charts of 30 minutes and above.
- MCX/commodity Daily Open Point is hidden on charts of 30 minutes and above; ORB remains available on higher intraday charts.
- Nifty 50, Bank Nifty, and Sensex index charts also hide Daily Open Point at 30 minutes and above. This does not disable the separately controlled Weekly Open.

## Opening range (ORB)

ORB is disabled by default. When enabled, it collects the high and low during a configured window after the selected session opening, then locks and draws the completed range.

- Duration choices: **1, 3, 5, 10, 15, 30, or 60 minutes**; default 5 minutes.
- Skip 0–20 initial chart candles; default 0. This shifts the collection window by the corresponding chart-timeframe duration.
- Display high, low, and optional midpoint `(high + low) / 2`.
- Extend lines through the session or end them at the latest bar.
- Customize labels, prices, colors, width, and Solid/Dashed/Dotted style.
- Reset for each selected session and remove the drawings after session close.

The range is built from chart bars, not lower-timeframe reconstruction. Use bars that fit and align with the selected opening window for an exact range; larger or misaligned bars can include prices outside that window.

## Liquidity sweeps

The optional sweep module detects a move beyond a confirmed swing followed by rejection back through that level on a confirmed candle.

- Swing length: **7** candles on each side by default.
- Minimum penetration: **0.15 × ATR(14)** beyond the swing.
- Volume: at least **1.20 × the 20-bar average**.
- Rejection: bearish candles close in the lower 40% of their range; bullish candles close in the upper 40%, by default.
- Bullish and bearish sweeps have separate visibility controls and a configurable marker color.
- Each confirmed swing can produce only one qualifying sweep signal.

Swing confirmation introduces a delay. Volume-dependent conditions may not qualify on symbols without usable volume data.

## Smart Money Concepts

### Break of Structure and Change of Character

The script tracks confirmed pivot highs/lows and consumes each level once it produces a qualifying break:

- **BOS:** a continuation break, or an initial qualifying structure break.
- **CHoCH:** a break against the previously tracked structure direction.
- A single break is not labeled as both BOS and CHoCH.

Default confirmation settings:

| Setting | Default |
| --- | --- |
| Confirmed swing length | 10 candles on each side |
| Break buffer | 0.30 ATR |
| Minimum candle displacement | 0.25 ATR |
| Minimum volume multiplier | 1.20 |
| Minimum body-to-range percentage | 55% |
| Volume average length | 20 |

Structure lines connect the swing level to the confirmed break candle, with optional centered labels. BOS/CHoCH visibility, bullish/bearish colors, style, width, and retained history are adjustable; the default is 25 structure lines.

### Equal highs and lows

EQH/EQL annotations identify nearby confirmed swing highs or lows. The default tolerance is **0.10 ATR**, with 10 historical annotations retained. Equal-high and equal-low colors are configurable.

### Order-block zones

A bullish structure break can create a demand zone from the most recent bearish candle; a bearish break can create a supply zone from the most recent bullish candle. Zones use that candle's high/low range.

- Source choices: Chart, 3 minutes, 5 minutes, 15 minutes, 1 hour, 4 hours, 1 day, 1 week, or 1 month.
- Chart is the default. Select a source timeframe equal to or higher than the chart timeframe for the intended use.
- Supply and demand zones have independent visibility and color controls.
- Zones extend across later sessions until price overlaps their range on a later chart bar, when they are removed as mitigated.
- Eight zones are retained by default; the limit is adjustable from 1 to 40.

These are simplified chart-based zones, not exchange order-book data. Higher-timeframe requests use `lookahead_off`; this alone should not be interpreted as a blanket guarantee that developing higher-timeframe values cannot change.

## Candle-pattern letters

This optional module draws one priority-selected annotation per confirmed candle. Hovering a letter shows its full pattern name.

| Letter | Pattern |
| --- | --- |
| E | Bullish or bearish Engulfing |
| C | Dark Cloud Cover |
| I | Inverted Hammer below the 21-period EMA |
| S | Shooting Star above the 21-period EMA |
| M | Bullish or bearish Marubozu |

Each pattern family can be toggled. Defaults include a 2:1 minimum upper-wick/body ratio for hammer/star detection, a 90% minimum body percentage for Marubozu, 0.10 ATR label offset, and 50 retained letters.

When multiple patterns qualify, priority is Dark Cloud Cover, Engulfing, Inverted Hammer, Shooting Star, then Marubozu. Pattern letters do not modify ML signals or trade plans.

## Manual support and resistance

Enter price levels separated by commas or newlines. The source includes a preloaded Nifty-oriented list; replace it with levels appropriate to the chart. These values are static inputs, not automatically maintained market levels.

- Shows the nearest two levels per side by default; configurable from 1 to 5.
- Levels below price are labeled **MS** (manual support); levels at or above price are labeled **MR** (manual resistance).
- A level changes role as price moves across it.
- Repeated prices are not displayed twice on the same side, and invalid text entries are ignored.
- Drawings are refreshed on the latest bar and use the same start/end times as the latest pivot set, including its next-period rollover. With Auto pivots this means daily spans through 15-minute charts, weekly spans on higher intraday charts, monthly spans on daily charts, and yearly spans on weekly/monthly charts.
- Explicit pivot timeframe selections, including multi-year anchors, also control the MS/MR span. This works when pivot lines are hidden. The former session-close movement option is removed because pivot timing now controls the extent.
- Colors, line style, and width are configurable.

Enter `24000`, not `24,000`: commas separate distinct levels.

## Alerts

The source exposes these eight alert conditions:

| Alert | Trigger |
| --- | --- |
| Open Long ▲ | ML long entry |
| Close Long ▲ | ML long exit |
| Open Short ▼ | ML short entry |
| Close Short ▼ | ML short exit |
| Open Position ▲▼ | Either ML entry |
| Close Position ▲▼ | Either ML exit |
| Kernel Bullish Color Change | Bullish kernel change condition |
| Kernel Bearish Color Change | Bearish kernel change condition |

Alert messages include ticker, price, and chart interval and retain the source's `LDC` prefix. Create the desired alert from the indicator's available conditions in TradingView.

The badge timing setting does **not** gate the alert conditions. Use a bar-close alert frequency when confirmed-close notifications are required. Hiding exit crosses or low-confidence badges does not disable the associated alert conditions.

There are no dedicated alert conditions for target/stop touches, ORB breaks, BOS/CHoCH, sweeps, order blocks, candle patterns, or manual levels in this version.

## Intraday Edge dashboard

The main table follows the supplied reference layout: **📈 Trade Stats**, Winrate, Trades, WL Ratio, Early Signal Flips, a blank separator, then **INTRADAY EDGE** with these rows in order:

**Trend, Bias, Option, Signal, Market, Strength, Volume, Liquidity, ORB, Risk.**

Both sections are enabled by default. The table uses grey cells, dark headers, light borders, centered text, and colored status values. The dashboard settings offer four corner positions and three text sizes. Show Trade Stats and Show Intraday Edge Table independently control the two sections. Entry, SL, and target prices are not included in the table; the chart trade-plan feature remains separately configurable.

The screenshot establishes appearance but does not provide the old formulas. The reconstructed display uses these definitions:

- **Trend:** kernel direction, with the existing smoothing setting.
- **Bias:** direction of the current ML vote total.
- **Option / Signal:** LONG or SHORT and the signal category on a confirmed entry candle; otherwise WAIT / NO SIGNAL.
- **Market:** the regime filter evaluated independently of its entry-filter toggle.
- **Strength:** one point each for directional ML votes, aligned kernel, aligned session open, aligned market structure, aligned ORB breakout, and volume meeting the SMC multiplier. Missing confirmations score zero, for a total of 0–6.
- **Volume:** HIGH at the SMC volume multiplier, LOW below 0.8 times average, otherwise NORMAL; N/A when unavailable.
- **Liquidity:** the existing enabled liquidity-sweep module's current bullish/bearish event, otherwise NONE.
- **ORB:** ABOVE ORB, BELOW ORB, or INSIDE ORB when the range is available; otherwise N/A.
- **Risk:** LOW for strength 5–6, MEDIUM for 4, HIGH for 0–3. This is a display category, not a measured loss probability.

Values update with the chart and can change intrabar. These summary calculations do not change entry signals, alerts, or trade plans. Numeric values are live calculations rather than the fixed numbers in the reference image.

## Statistics and backtest output

Enable **Show Trade Stats** to display an ML calibration table containing win rate, total trades with wins/losses, win/loss count ratio, and early signal flips. An early flip is a model direction change before the four-bar holding period completes.

**Use Worst Case Estimates** requests close-based estimates from the library's backtest helper. The panel describes ML entries/exits only; it does not evaluate the visual trade plan's next-open entries, targets, stops, or optional reference overlays.

A hidden **Backtest Stream** plot is available for adapter use:

| Value | Event |
| --- | --- |
| `1` | Long entry |
| `2` | Long exit |
| `-1` | Short entry |
| `-2` | Short exit |
| `na` | No event |

When conditions overlap, the stream uses the first matching event in the order above. This is an `indicator()`, not a `strategy()`, and the statistics table is a calibration aid rather than a complete execution backtest.

## Configuration and troubleshooting

1. Select the session profile matching the symbol; Auto's non-MCX/non-crypto fallback uses the configured equity hours.
2. Replace the preloaded manual levels for your instrument.
3. Choose preview badges or confirmed-close badges, and configure alert frequency separately.
4. Enable the visual references you need and configure the trade plan's ATR buffer and R multiples.
5. Inspect behavior on the intended chart timeframe before relying on its output.

| Symptom | Check |
| --- | --- |
| Script time-limit warning | Disable Include Full History; reduce Max Bars Back to 500–1,000; leave trade stats off when unused |
| Missing ORB | Check intraday timeframe, session hours, skipped candles, completed collection window, and crypto visibility restrictions |
| Missing Daily Open Point | Check active session and the 30-minute MCX/crypto visibility restrictions |
| No sweep or structure marker | Check confirmed swing availability, volume, displacement, and rejection thresholds |
| Missing trade plan | Wait for the next candle after a signal; check whether the plan hit its stop/targets or was invalidated by a gap |
| Pivot history error | Load more history or select a shorter pivot anchor |
| Old drawings disappear | Check retention settings; the script declares limits of 500 lines, 500 labels, and 50 boxes shared across modules |

Preview signals can change before close, confirmed pivots need future bars to become known, and session cleanup depends on script updates. Historical chart output does not reproduce every live preview or intrabar sequence.

## Attribution and license

The source credits **Krishna Kanth** and declares the **Mozilla Public License 2.0** in its header. Imported libraries are authored under the TradingView account **jdehorty** and retain their respective attribution and licensing terms.

The configurable chart watermark reads: **FOR ANALYSIS PURPOSES ONLY - NOT A TRADING RECOMMENDATION**.
