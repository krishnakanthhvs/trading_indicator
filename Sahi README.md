# Intraday Edge V2 for Sahi

Use **Intraday Edge Sahi.pine**. The original **Intraday Edge** and **README.md** are unchanged.

## Install

1. Open a chart at https://web.sahi.com.
2. Open **Indicators → New script** (or **My scripts → Create new**).
3. Replace the starter EMA with the entire contents of **Intraday Edge Sahi.pine**.
4. Select **Apply** to check the chart, then **Save** to retain the indicator under your account.
5. Configure the settings for your instrument and session. The manual levels are still the original Nifty-oriented reference levels.

## Changes

- One self-contained Pine v6-compatible indicator declaration, with a distinct Sahi title.
- Replaced the two unsupported library imports with local normalized RSI, WaveTrend, CCI, ADX, volatility/regime/ADX filters, and rational-quadratic/Gaussian kernel calculations. Mathematical helpers are adapted from jdehorty's MPL-2.0 sources; attribution remains in the script.
- Passed the latest-signal tooltip directly to `table.cell`, avoiding the unsupported `table.cell_set_tooltip` setter.
- Moved all 12 input calls embedded in settings constructors into individual named assignments, as required by Sahi. Settings defaults remain the same.
- Bounded the five feature arrays and label array to 10,000 bars in both chart and multi-timeframe models. Preserved their alignment and the original absolute-bar modulo sampling. **Include Retained History** now searches at most 10,000 loaded bars. The default recent-history search remains 2,000 bars.
- Added valid minimum feature periods and bounded kernel parameters to prevent zero denominators and excessive history offsets. Defaults are unchanged.
- Replaced dashboard emoji and target check marks with ordinary text/geometric symbols for Sahi's chart fonts.
- Retained the original signal rules, confidence badges and previews, entries/exits, trade-plan drawings, pivots, session open levels, ORB, liquidity sweeps, BOS/CHoCH, equal levels, order blocks, manual levels, dashboard, multi-timeframe model, and period VWAP drawings.

## Differences and platform limits

**Statistics:** The unavailable imported `ml.backtest` is replaced by local calibration counters. These count confirmed ML exits from a tracked entry, process exits before new entries on the same candle, exclude breakeven exits from win/loss totals, and count early signal flips separately. A new entry replaces a still-open entry. Price uses the original close-only option or `(high + low + 2 × open) / 4` estimate. These figures are not guaranteed to match the imported TradingView helper. They do not count the separate visual SL/TP touches, fees, slippage, or broker fills. Signals do not depend on these statistics.

**Alerts:** Sahi's current FAQ says `alert()` and `alertcondition()` parse but do not send notifications. The eight original alert conditions remain in the source for future support; they currently do not provide Sahi notifications.

**Execution:** Sahi allows one second per bar and up to 10,000 history bars. Full retained history and multiple enabled ML timeframes can be expensive. If the editor reports a timeout, disable the Multi-TF table or unused timeframes and lower Max Bars Back to 500–1,000. The original default settings are retained; performance has not been measured in Sahi.

**Data:** Sahi and TradingView may provide different price, volume, session, and available-history data. Matching source logic does not guarantee identical signals between platforms. Higher-timeframe calculations and live previews retain the original developing-bar behavior.

## Verification status

Checked locally: exactly one indicator declaration; no library imports or unresolved library-helper calls; all input calls assigned to variables; balanced delimiters; source below Sahi's 250 KB script limit; capped feature/label arrays stay aligned and preserve absolute sample spacing across eviction; original files match their Git versions byte for byte.

**Sahi compilation and chart execution are not verified.** There is no authenticated Sahi editor/compiler available in this workspace. In particular, the advanced original drawings, user-defined types, matrices, dynamic timeframe requests and polyline APIs still need confirmation against Sahi's implementation. The source targets its documented Pine v6 compatibility; local checks cannot establish runtime compatibility or signal parity.

## References reviewed

- Walkthrough and its auto-generated English captions: https://www.sahi.com/video-guide/en/custom-indicator-full-walkthrough (linked video: https://www.youtube.com/watch?v=HFgEOcbIw0A).
- SahiScript FAQ: https://web.sahi.com/static/sahiscript-faq.html (reviewed 8 October 2026).
- jdehorty's ML library reference: https://www.tradingview.com/script/ia5ozyMF-MLExtensions/.
- Helper source: https://github.com/artificial-intelligence-edge/lorentzian-classification/tree/main/ports/pinescript/libraries. Only the required mathematical helpers are embedded; the repository's extended performance-analysis helper is not used.
