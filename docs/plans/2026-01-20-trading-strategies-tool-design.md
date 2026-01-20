# 151 Trading Strategies Visualization Tool — Design Document

**Date:** 2026-01-20
**Status:** Approved
**Author:** Collaborative design session

---

## 1. Overview

A single-page web application that visualizes 151 trading strategies from the Kakushadze & Serur academic paper, applied to live cryptocurrency market data from decentralized exchanges.

**What it does:**
- Displays live candlestick charts with real-time updates
- Allows selecting up to 5 strategies simultaneously from 151 options
- Overlays strategy signals (buy/sell markers) on the chart
- Shows confluence when multiple strategies agree
- Provides clear entry/exit signals based on strategy agreement

**Key characteristics:**
- Single HTML file with inline JavaScript and CSS
- No build step required — load dependencies from CDN
- Dark theme, desktop-optimized (1024px+)
- Modular data layer enabling future exchange support
- URL-based state for shareable analysis setups

**Primary use case:**
Exploratory learning tool for traders who want to understand how classic trading strategies behave on live crypto markets, and discover when multiple strategies align.

**Transparency principle:**
Many strategies in the source material are designed for options or other asset classes. When a strategy is adapted or missing required data, the tool displays a subtle warning so users understand the limitation.

---

## 2. Goals and Non-Goals

### Goals

1. **Educational exploration** — Help users understand how 151 trading strategies behave in practice, not just in theory

2. **Live data visualization** — Show strategies reacting to real market conditions in real-time, not just historical backtests

3. **Confluence discovery** — Highlight when multiple independent strategies agree, as these moments are often more significant

4. **Transparency about limitations** — Clearly indicate when strategies are adapted from their original context (e.g., options strategies applied to spot)

5. **Shareability** — Enable users to share specific setups via URL for learning and discussion

6. **Modularity** — Structure the data layer so additional exchanges can be added without rewriting the application

### Non-Goals

1. **Trading execution** — This tool does not place trades; it's for analysis only

2. **Mobile support** — Optimized for desktop; no responsive layouts for phones

3. **Backtesting engine** — Shows live/recent data, not historical performance metrics

4. **User accounts or cloud storage** — No login, no server-side persistence

5. **Multiple simultaneous assets** — One chart at a time (multi-asset is a future enhancement)

6. **Financial advice** — Tool is educational; strategies are presented as-is from academic literature

---

## 3. Architecture Overview

### High-level structure

```
┌─────────────────────────────────────────────────────────────┐
│                        URL State                            │
│                  (symbol, timeframe, strategies)            │
└─────────────────────────┬───────────────────────────────────┘
                          │
┌─────────────────────────▼───────────────────────────────────┐
│                      App Controller                         │
│            (coordinates all modules, handles events)        │
└──────┬──────────────────┬───────────────────────────────────┘
       │                  │                       │
┌──────▼──────┐   ┌───────▼───────┐   ┌──────────▼──────────┐
│ Data Layer  │   │Strategy Engine│   │    UI Renderer      │
│             │   │               │   │                     │
│ - Provider  │   │ - 151 strats  │   │ - Chart (LWC)       │
│   interface │   │ - Calculate   │   │ - Controls          │
│ - Hyperliquid│  │   signals     │   │ - Signal log        │
│   impl      │   │ - Confluence  │   │ - Confluence badge  │
└──────┬──────┘   └───────┬───────┘   └──────────┬──────────┘
       │                  │                      │
       └──────────────────┴──────────────────────┘
                          │
              ┌───────────▼───────────┐
              │   Shared Candle Data  │
              │      (in memory)      │
              └───────────────────────┘
```

### Data flow

1. On load: Parse URL → Initialize provider → Fetch 500 historical candles
2. Connect WebSocket → Merge incoming candles into array
3. When candles update: Recalculate active strategies → Update signals
4. Render: Chart draws candles + markers, UI updates confluence badge
5. User interaction: Update URL → Trigger recalculation → Re-render

**Key principle:** Unidirectional data flow. Candle data is the single source of truth. Strategies read from it, UI renders from strategy outputs.

---

## 4. Data Layer

### DataProvider Interface

```javascript
const DataProvider = {
  name: string,                    // 'hyperliquid'

  // Get available trading pairs
  async getSymbols(): Symbol[],

  // Fetch historical candles
  async fetchCandles(symbol, interval, limit): Candle[],

  // Subscribe to live updates
  subscribeCandles(symbol, interval, onCandle): void,

  // Clean up
  unsubscribe(): void
}
```

### Candle format (normalized)

```javascript
{
  time: number,      // Unix timestamp (seconds)
  open: number,
  high: number,
  low: number,
  close: number,
  volume: number
}
```

### Hyperliquid implementation specifics

- **REST endpoint:** `https://api.hyperliquid.xyz/info` for historical candles
- **WebSocket:** `wss://api.hyperliquid.xyz/ws` for live updates
- **Subscription:** `{ "method": "subscribe", "subscription": { "type": "candle", "coin": "BTC", "interval": "1h" }}`
- **Intervals supported:** 1m, 3m, 5m, 15m, 30m, 1h, 2h, 4h, 8h, 12h, 1d, 3d, 1w, 1M

### Candle merging logic

- Incoming WebSocket candle matches last candle's timestamp → Update in place
- Incoming candle has new timestamp → Append to array
- Maintain maximum 500 candles in memory (drop oldest when exceeding)

---

## 5. Strategy Engine

### Strategy interface

```javascript
const Strategy = {
  id: string,              // 'sma-cross'
  name: string,            // 'Single Moving Average'
  category: string,        // 'Trend Following'
  description: string,     // Brief explanation
  sourceSection: string,   // '3.11' (reference to 151 paper)

  // Adaptation warning (null if native to spot/crypto)
  warning: string | null,  // 'Adapted from options strategy'

  // Configurable parameters with defaults
  params: {
    period: { default: 20, min: 2, max: 500, label: 'Period' }
  },

  // Calculate signal for each candle
  calculate(candles, params): Signal[]
}
```

### Signal format

```javascript
{
  time: number,            // Candle timestamp
  type: 'buy' | 'sell' | 'neutral',
  strength: number,        // 0-1, for future weighting
  reason: string           // 'Price crossed above SMA(20)'
}
```

### Strategy categories (8 groups)

| Category | Example Strategies | Approx Count |
|----------|-------------------|--------------|
| Trend Following | SMA, EMA, MACD, Three MA | ~15 |
| Mean Reversion | RSI, Bollinger Bands, Mean Reversion Clusters | ~12 |
| Momentum | Price Momentum, Residual Momentum | ~10 |
| Volatility | ATR Breakout, Volatility Bands | ~8 |
| Support/Resistance | Pivot Points, Donchian Channel | ~8 |
| Options-Adapted | Covered Call Signal, Straddle Breakout | ~60 |
| Machine Learning | KNN, Neural Network Simplified | ~5 |
| Multi-Factor | Alpha Combos, Factor Rotation | ~10 |

### Calculation approach

- Strategies receive full candle array, return signal for each candle
- Signals are cached; recalculated only when candles update or params change
- Each strategy is self-contained — no dependencies between strategies

---

## 6. UI Components

### Overall layout

```
┌─────────────────────────────────────────────────────────────────┐
│ [Symbol ▼]  [Timeframe ▼]                           🟢 Live     │ Row 1
├─────────────────────────────────────────────────────────────────┤
│ [Strategy 1 ▼]⚙ [Strategy 2 ▼]⚙ [+ Add]   Threshold:[==3==]   │ Row 2
│                                              │ 3/5 Bullish │    │
├─────────────────────────────────────────────────────────────────┤
│ ┌─ Settings: SMA Cross ─────────────────────────────────────┐   │ Row 2.5
│ │  Period: [20]  Signal on: [Cross ▼]            [Apply]    │   │ (expanded)
│ └───────────────────────────────────────────────────────────┘   │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│                                                                 │
│                         CHART                                   │
│                    (Lightweight Charts)                         │
│                                                                 │
│    ▲ SMA     ▲ RSI                     ▼ MACD                   │ Markers
│                                                                 │
├─────────────────────────────────────────────────────────────────┤
│ ▶ Signals (12)                                   [Export CSV]   │ Collapsed
└─────────────────────────────────────────────────────────────────┘
```

### Component breakdown

| Component | Library/Approach |
|-----------|------------------|
| Symbol dropdown | Choices.js (searchable) |
| Timeframe dropdown | Native `<select>` (14 options) |
| Strategy dropdown | Choices.js (searchable + grouped) |
| Settings accordion | Vanilla JS toggle |
| Threshold slider | Native `<input type="range">` |
| Confluence badge | Styled `<div>` |
| Chart | Lightweight Charts |
| Signal markers | LWC markers API |
| Signal log | Vanilla JS list |
| Toasts | Vanilla JS (absolute positioned) |
| Status indicator | Styled `<span>` |

### Marker colors (by strategy slot)

- Strategy 1: `#2962FF` (blue)
- Strategy 2: `#FF6D00` (orange)
- Strategy 3: `#00C853` (green)
- Strategy 4: `#AA00FF` (purple)
- Strategy 5: `#FFD600` (yellow)

---

## 7. Confluence System

### Voting mechanism

Each active strategy votes on each candle:
- **Bullish (+1):** Strategy's most recent signal is "buy"
- **Bearish (-1):** Strategy's most recent signal is "sell"
- **Neutral (0):** No signal or signal expired

### Confluence calculation

```javascript
function calculateConfluence(strategies, currentTime) {
  let bullish = 0, bearish = 0, neutral = 0;

  for (const strategy of activeStrategies) {
    const signal = getMostRecentSignal(strategy, currentTime);
    if (signal.type === 'buy') bullish++;
    else if (signal.type === 'sell') bearish++;
    else neutral++;
  }

  return { bullish, bearish, neutral, total: strategies.length };
}
```

### Threshold slider

- Range: 1 to 5 (matches max active strategies)
- Default: 3
- Label updates: "Signal when ≥ N strategies agree"
- When threshold met → Entry signal marker with glow effect

### Confluence badge display

| State | Display |
|-------|---------|
| 3 bullish, 1 bearish, 1 neutral | `3/5 Bullish` (green tint) |
| 2 bullish, 2 bearish, 1 neutral | `Mixed` (gray) |
| 0 bullish, 4 bearish, 1 neutral | `4/5 Bearish` (red tint) |
| All neutral | `Neutral` (gray) |

### Background zones

- When ≥ 4 strategies agree: subtle background wash on chart
- Green wash (`rgba(0, 200, 83, 0.05)`) for bullish confluence
- Red wash (`rgba(255, 82, 82, 0.05)`) for bearish confluence
- Helps users spot "interesting periods" when scrolling history

### Signal log entry for confluence

```
14:32  CONFLUENCE  ▲ Strong Buy (4/5)  $95,420
```

---

## 8. State Management

### URL hash format

```
#sym=BTC&tf=4h&s1=sma:20&s2=rsi:14,70,30&s3=macd&th=3
```

### Parameter encoding

| Key | Description | Example |
|-----|-------------|---------|
| `sym` | Symbol | `BTC`, `ETH`, `SOL` |
| `tf` | Timeframe | `1m`, `1h`, `4h`, `1d` |
| `s1`-`s5` | Strategy + params | `sma:20` or `rsi:14,70,30` |
| `th` | Confluence threshold | `1` to `5` |

### Strategy param encoding

```
{strategyId}:{param1},{param2},{param3}
```

- `sma:50` → SMA with period 50
- `rsi:14,70,30` → RSI with period 14, overbought 70, oversold 30
- `macd` → MACD with default params (no colon needed)

### State flow

```
Page Load
    │
    ▼
Parse URL hash ──(empty)──► Apply defaults (BTC, 1h, no strategies, th=3)
    │
    (has params)
    │
    ▼
Validate params ──(invalid)──► Fall back to defaults, show toast
    │
    (valid)
    │
    ▼
Initialize app with parsed state
    │
    ▼
User changes setting ──► Update URL hash (replaceState, no history spam)
```

### Defaults when URL is empty

```javascript
{
  symbol: 'BTC',
  timeframe: '1h',
  strategies: [],      // None selected
  threshold: 3
}
```

**Sharing:** User copies URL from address bar → Recipient opens → Sees exact same setup.

---

## 9. Error Handling

### Connection states

| State | Indicator | Behavior |
|-------|-----------|----------|
| Connecting | 🟡 `Connecting...` | Initial load, show spinner on chart |
| Connected | 🟢 `Live` | Normal operation |
| Reconnecting | 🟡 `Reconnecting...` | Auto-retry with backoff, toast shown |
| Disconnected | 🔴 `Disconnected` | Retries exhausted, manual retry button |

### Retry logic

```javascript
const RETRY_DELAYS = [1000, 2000, 4000, 8000]; // Exponential backoff

async function connectWithRetry() {
  for (let attempt = 0; attempt < RETRY_DELAYS.length; attempt++) {
    try {
      await connect();
      return; // Success
    } catch (err) {
      if (attempt < RETRY_DELAYS.length - 1) {
        showToast(`Connection failed, retrying in ${RETRY_DELAYS[attempt]/1000}s...`);
        await sleep(RETRY_DELAYS[attempt]);
      }
    }
  }
  showToast('Unable to connect. Check your network.', 'error');
  setStatus('disconnected');
}
```

### Toast notifications

| Event | Toast Message | Duration |
|-------|---------------|----------|
| WebSocket connected | `✓ Connected to Hyperliquid` | 3s |
| Connection lost | `⚠️ Connection lost, reconnecting...` | Until resolved |
| Reconnected | `✓ Reconnected` | 3s |
| Data gap detected | `⚠️ Gap detected: ${n} candles missing` | 5s |
| Invalid URL params | `⚠️ Invalid settings in URL, using defaults` | 5s |
| REST fetch failed | `⚠️ Failed to load history, retrying...` | Until resolved |

### Graceful degradation

- WebSocket fails but REST works → Show historical data, badge shows 🟡 `History only`
- Symbol not found → Toast + fall back to BTC
- Strategy calculation error → Skip that strategy, log to console, show toast

---

## 10. File Structure

### Repository layout

```
/
├── index.html              # Main application (single file)
├── README.md               # Project overview and usage
├── LICENSE                 # MIT or similar
│
├── assets/
│   └── favicon.ico         # Browser tab icon
│
└── docs/
    └── plans/
        ├── 2026-01-20-trading-strategies-tool-design.md
        └── 2026-01-20-trading-strategies-tool-implementation.md
```

### index.html internal structure

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>151 Trading Strategies</title>
    <link rel="icon" href="assets/favicon.ico">

    <!-- CDN Dependencies -->
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/choices.js/public/assets/styles/choices.min.css">
    <script src="https://cdn.jsdelivr.net/npm/choices.js/public/assets/scripts/choices.min.js"></script>
    <script src="https://unpkg.com/lightweight-charts/dist/lightweight-charts.standalone.production.js"></script>

    <!-- Inline Styles -->
    <style>
        /* ~300 lines: dark theme, layout, components */
    </style>
</head>
<body>
    <!-- HTML Structure -->

    <script>
    // ~2000-3000 lines organized as:
    // 1. Config & Constants
    // 2. Data Provider (interface + Hyperliquid impl)
    // 3. Strategy Definitions (all 151)
    // 4. Strategy Engine
    // 5. Confluence Calculator
    // 6. URL State Manager
    // 7. UI Controller
    // 8. Chart Manager
    // 9. Toast/Status Manager
    // 10. App Initialization
    </script>
</body>
</html>
```

### Why single file works here

- No module bundler needed
- Easy to deploy (just serve one file)
- Strategies are data-like (config objects), not complex logic
- Total size estimate: ~150KB uncompressed, ~40KB gzipped

---

## 11. Dependencies

### CDN-loaded libraries

| Library | Version | Purpose | Size |
|---------|---------|---------|------|
| Lightweight Charts | 4.x | Candlestick charting | ~40KB gzip |
| Choices.js | 10.x | Searchable dropdowns | ~20KB gzip |

### Why these libraries

- **Lightweight Charts:** Purpose-built for financial charts by TradingView. Handles candlesticks, markers, crosshair, zoom/pan out of the box. Well-documented, actively maintained.

- **Choices.js:** Clean searchable/groupable dropdowns. No jQuery dependency. Handles 151 strategies in grouped dropdown gracefully.

### No additional libraries needed for

- WebSocket — Native browser API
- Fetch — Native browser API
- URL parsing — Native `URLSearchParams`
- DOM manipulation — Vanilla JS
- Toasts — Custom (10 lines of CSS + JS)
- Settings accordion — Custom (trivial toggle)

### CDN URLs (pinned versions for stability)

```html
<!-- Choices.js -->
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/choices.js@10.2.0/public/assets/styles/choices.min.css">
<script src="https://cdn.jsdelivr.net/npm/choices.js@10.2.0/public/assets/scripts/choices.min.js"></script>

<!-- Lightweight Charts -->
<script src="https://unpkg.com/lightweight-charts@4.1.0/dist/lightweight-charts.standalone.production.js"></script>
```

### Total page weight estimate

- HTML/CSS/JS: ~150KB
- Libraries: ~60KB gzip
- **Total: ~100KB gzip** (fast initial load)

---

## 12. Future Enhancements

### Noted for future, explicitly not in v1

| Enhancement | Description | Why Deferred |
|-------------|-------------|--------------|
| Multi-asset view | 2-3 charts side by side for correlation analysis | Adds significant complexity; single asset covers primary use case |
| Additional exchanges | Binance, Coinbase, Kraken data providers | Interface is ready; implement when needed |
| Lazy-load history | Fetch more candles when scrolling left | 500 candles sufficient for most analysis |
| Mobile responsive | Phone/tablet layouts | Desktop tool; charts need screen space |
| Backtesting metrics | Win rate, profit factor, drawdown stats | Shifts from exploration to evaluation; different tool |
| Alerts/notifications | Browser notifications when confluence reached | Requires permissions, adds complexity |
| Custom strategies | User-defined strategy formulas | Editor UI is substantial; 151 strategies is plenty |
| Weighted voting | User-assigned strategy weights | Obscures learning; simple voting is more transparent |
| localStorage backup | Persist last session if URL empty | URL-based sharing is primary; keep it simple |
| Strategy performance comparison | Side-by-side signal accuracy | Needs historical ground truth; out of scope |

### Extension points built into v1

1. **DataProvider interface** — Add new exchange by implementing 4 methods
2. **Strategy registry** — Add new strategies by adding objects to the array
3. **Category system** — New categories just need a string identifier
4. **Signal format** — `strength` field reserved for future weighted voting

---

## Appendix A: Reference Material

**Source:** "151 Trading Strategies" by Zura Kakushadze and Juan Andrés Serur (2018)

The paper provides detailed descriptions and mathematical formulas for 151+ trading strategies across multiple asset classes. This tool adapts these strategies for live cryptocurrency market visualization.

**Key sections referenced:**
- Section 3: Stocks (momentum, mean-reversion, moving averages)
- Section 7: Volatility strategies
- Section 18: Cryptocurrencies (ANN, sentiment analysis)

**Data source:** Hyperliquid decentralized exchange API
- REST: `https://api.hyperliquid.xyz/info`
- WebSocket: `wss://api.hyperliquid.xyz/ws`
