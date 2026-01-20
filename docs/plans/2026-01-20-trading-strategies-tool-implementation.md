# 151 Trading Strategies Tool — Implementation Plan

**Date:** 2026-01-20
**Prerequisites:** Read the [Design Document](./2026-01-20-trading-strategies-tool-design.md) first.
**Estimated tasks:** 24 bite-sized tasks across 8 phases

---

## Before You Start

### Required Reading

1. **Design Doc** — `docs/plans/2026-01-20-trading-strategies-tool-design.md` (understand what we're building)
2. **151 Strategies Reference** — `references/151.txt` (skim sections 3.11-3.15 for moving average strategies)
3. **Lightweight Charts Docs** — https://tradingview.github.io/lightweight-charts/
4. **Choices.js Docs** — https://choices-js.github.io/Choices/
5. **Hyperliquid API Docs** — https://hyperliquid.gitbook.io/hyperliquid-docs/for-developers/api

### Development Principles

| Principle | What It Means Here |
|-----------|-------------------|
| **TDD** | Write a test (or manual test plan) before implementing. Verify it fails. Implement. Verify it passes. |
| **DRY** | Don't copy-paste strategy logic. Use the Strategy interface. |
| **YAGNI** | Don't build features not in the design doc. No "nice-to-haves." |
| **Frequent Commits** | Commit after each task. Small, focused commits with clear messages. |

### Local Development Setup

```bash
# No build step needed. Just serve the HTML file.
cd /home/user/151-trading-strategies

# Option 1: Python
python3 -m http.server 8080

# Option 2: Node
npx serve .

# Then open http://localhost:8080
```

### Testing Approach

Since this is a single HTML file with no build system, we use:

1. **Manual browser testing** — Open DevTools, check console for errors
2. **Console-based unit tests** — Small test functions that log PASS/FAIL
3. **Visual verification** — Does the chart render? Do markers appear?

Each task includes specific test criteria.

---

## Phase 1: Project Scaffolding

### Task 1.1: Create Directory Structure

**Goal:** Set up the repository structure defined in the design doc.

**Files to create:**
- `index.html` (empty placeholder)
- `README.md` (basic project description)
- `assets/` directory (empty for now)

**Steps:**

```bash
cd /home/user/151-trading-strategies

# Create assets directory
mkdir -p assets

# Create placeholder index.html
touch index.html
```

**index.html content:**
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>151 Trading Strategies</title>
</head>
<body>
    <h1>151 Trading Strategies</h1>
    <p>Coming soon...</p>
</body>
</html>
```

**README.md content:**
```markdown
# 151 Trading Strategies Visualization Tool

An exploratory tool for visualizing 151 trading strategies on live cryptocurrency data.

## Usage

Open `index.html` in a browser, or serve locally:

```bash
python3 -m http.server 8080
```

Then visit http://localhost:8080

## Documentation

- [Design Document](docs/plans/2026-01-20-trading-strategies-tool-design.md)
- [Implementation Plan](docs/plans/2026-01-20-trading-strategies-tool-implementation.md)
```

**Test criteria:**
- [ ] `index.html` opens in browser without errors
- [ ] Shows "151 Trading Strategies" heading

**Commit message:** `scaffold: create project structure with index.html and README`

---

### Task 1.2: Add CDN Dependencies

**Goal:** Load Lightweight Charts and Choices.js from CDN.

**File to modify:** `index.html`

**Changes:**
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>151 Trading Strategies</title>

    <!-- Choices.js CSS -->
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/choices.js@10.2.0/public/assets/styles/choices.min.css">

    <!-- Choices.js JS -->
    <script src="https://cdn.jsdelivr.net/npm/choices.js@10.2.0/public/assets/scripts/choices.min.js"></script>

    <!-- Lightweight Charts -->
    <script src="https://unpkg.com/lightweight-charts@4.1.0/dist/lightweight-charts.standalone.production.js"></script>
</head>
<body>
    <h1>151 Trading Strategies</h1>

    <script>
        // Verify libraries loaded
        console.log('Choices.js loaded:', typeof Choices !== 'undefined');
        console.log('Lightweight Charts loaded:', typeof LightweightCharts !== 'undefined');
    </script>
</body>
</html>
```

**Test criteria:**
- [ ] Open browser DevTools console
- [ ] See `Choices.js loaded: true`
- [ ] See `Lightweight Charts loaded: true`
- [ ] No 404 errors in Network tab

**Commit message:** `deps: add Lightweight Charts and Choices.js from CDN`

---

### Task 1.3: Add Base Dark Theme CSS

**Goal:** Establish the dark theme foundation.

**File to modify:** `index.html`

**Add inside `<head>` after CDN links:**
```html
<style>
    /* ========== CSS Reset & Base ========== */
    *, *::before, *::after {
        box-sizing: border-box;
        margin: 0;
        padding: 0;
    }

    /* ========== Dark Theme Variables ========== */
    :root {
        --bg-primary: #1a1a2e;
        --bg-secondary: #16213e;
        --bg-tertiary: #0f0f1a;
        --text-primary: #eaeaea;
        --text-secondary: #a0a0a0;
        --accent-blue: #2962ff;
        --accent-green: #00c853;
        --accent-red: #ff5252;
        --accent-orange: #ff6d00;
        --accent-purple: #aa00ff;
        --accent-yellow: #ffd600;
        --border-color: #2a2a4a;
    }

    /* ========== Base Styles ========== */
    html, body {
        height: 100%;
        font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
        font-size: 14px;
        background-color: var(--bg-primary);
        color: var(--text-primary);
    }

    body {
        display: flex;
        flex-direction: column;
        min-height: 100vh;
    }

    /* ========== Utility Classes ========== */
    .text-green { color: var(--accent-green); }
    .text-red { color: var(--accent-red); }
    .text-muted { color: var(--text-secondary); }
</style>
```

**Update `<body>`:**
```html
<body>
    <h1 style="padding: 20px;">151 Trading Strategies</h1>
    <p style="padding: 0 20px;" class="text-muted">Dark theme active</p>
</body>
```

**Test criteria:**
- [ ] Page has dark background (#1a1a2e)
- [ ] Text is light colored (#eaeaea)
- [ ] "Dark theme active" text appears muted/gray

**Commit message:** `style: add dark theme CSS foundation`

---

## Phase 2: Data Layer

### Task 2.1: Create DataProvider Interface and Constants

**Goal:** Define the interface that all data providers must implement, plus app constants.

**File to modify:** `index.html`

**Add inside `<script>` (replace the library verification code):**
```javascript
// ============================================================
// SECTION 1: CONSTANTS & CONFIGURATION
// ============================================================

const CONFIG = {
    defaultSymbol: 'BTC',
    defaultTimeframe: '1h',
    defaultThreshold: 3,
    candleLimit: 500,
    maxStrategies: 5,
    retryDelays: [1000, 2000, 4000, 8000],
};

const TIMEFRAMES = ['1m', '3m', '5m', '15m', '30m', '1h', '2h', '4h', '8h', '12h', '1d', '3d', '1w', '1M'];

// ============================================================
// SECTION 2: DATA PROVIDER INTERFACE
// ============================================================

/**
 * DataProvider Interface (documentation only - JS has no interfaces)
 *
 * Any data provider must implement:
 * - name: string
 * - async getSymbols(): Promise<{symbol: string, name: string}[]>
 * - async fetchCandles(symbol, interval, limit): Promise<Candle[]>
 * - subscribeCandles(symbol, interval, onCandle): void
 * - unsubscribe(): void
 *
 * Candle format:
 * { time: number, open: number, high: number, low: number, close: number, volume: number }
 */

// Placeholder - will be replaced in Task 2.2
let dataProvider = null;

// ============================================================
// INITIALIZATION (temporary)
// ============================================================

console.log('Config loaded:', CONFIG);
console.log('Timeframes:', TIMEFRAMES);
```

**Test criteria:**
- [ ] Console shows `Config loaded:` with the config object
- [ ] Console shows `Timeframes:` with all 14 intervals
- [ ] No JavaScript errors

**Commit message:** `feat(data): add constants and DataProvider interface documentation`

---

### Task 2.2: Implement Hyperliquid REST (fetchCandles)

**Goal:** Fetch historical candle data from Hyperliquid's REST API.

**File to modify:** `index.html`

**Add after the interface documentation:**
```javascript
// ============================================================
// SECTION 3: HYPERLIQUID DATA PROVIDER
// ============================================================

const HyperliquidProvider = {
    name: 'hyperliquid',

    // REST endpoint
    baseUrl: 'https://api.hyperliquid.xyz/info',

    // WebSocket endpoint (used later)
    wsUrl: 'wss://api.hyperliquid.xyz/ws',

    // WebSocket connection (initialized later)
    ws: null,

    /**
     * Get available trading symbols
     * @returns {Promise<{symbol: string, name: string}[]>}
     */
    async getSymbols() {
        try {
            const response = await fetch(this.baseUrl, {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify({ type: 'meta' })
            });
            const data = await response.json();

            // Extract perpetual symbols from universe
            return data.universe.map(item => ({
                symbol: item.name,
                name: item.name
            }));
        } catch (error) {
            console.error('Failed to fetch symbols:', error);
            throw error;
        }
    },

    /**
     * Fetch historical candles
     * @param {string} symbol - e.g., 'BTC'
     * @param {string} interval - e.g., '1h'
     * @param {number} limit - number of candles
     * @returns {Promise<Candle[]>}
     */
    async fetchCandles(symbol, interval, limit = 500) {
        try {
            // Calculate start time (go back enough to get `limit` candles)
            const intervalMs = this._intervalToMs(interval);
            const endTime = Date.now();
            const startTime = endTime - (limit * intervalMs);

            const response = await fetch(this.baseUrl, {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify({
                    type: 'candleSnapshot',
                    req: {
                        coin: symbol,
                        interval: interval,
                        startTime: startTime,
                        endTime: endTime
                    }
                })
            });

            const data = await response.json();

            // Normalize to our candle format
            return data.map(candle => ({
                time: Math.floor(candle.t / 1000), // Convert ms to seconds for LWC
                open: parseFloat(candle.o),
                high: parseFloat(candle.h),
                low: parseFloat(candle.l),
                close: parseFloat(candle.c),
                volume: parseFloat(candle.v)
            }));
        } catch (error) {
            console.error('Failed to fetch candles:', error);
            throw error;
        }
    },

    /**
     * Convert interval string to milliseconds
     * @private
     */
    _intervalToMs(interval) {
        const units = {
            'm': 60 * 1000,
            'h': 60 * 60 * 1000,
            'd': 24 * 60 * 60 * 1000,
            'w': 7 * 24 * 60 * 60 * 1000,
            'M': 30 * 24 * 60 * 60 * 1000
        };
        const match = interval.match(/^(\d+)([mhdwM])$/);
        if (!match) throw new Error(`Invalid interval: ${interval}`);
        return parseInt(match[1]) * units[match[2]];
    },

    // Placeholder methods (implemented in Task 2.3)
    subscribeCandles(symbol, interval, onCandle) {
        console.warn('WebSocket subscription not yet implemented');
    },

    unsubscribe() {
        console.warn('WebSocket unsubscribe not yet implemented');
    }
};

// Set as the active provider
dataProvider = HyperliquidProvider;
```

**Add test code at the end of `<script>`:**
```javascript
// ============================================================
// TEMPORARY TEST CODE (remove after verification)
// ============================================================

async function testDataProvider() {
    console.log('--- Testing DataProvider ---');

    // Test 1: Fetch symbols
    console.log('Test 1: Fetching symbols...');
    const symbols = await dataProvider.getSymbols();
    console.log(`✓ Got ${symbols.length} symbols. First 5:`, symbols.slice(0, 5));

    // Test 2: Fetch candles
    console.log('Test 2: Fetching BTC 1h candles...');
    const candles = await dataProvider.fetchCandles('BTC', '1h', 10);
    console.log(`✓ Got ${candles.length} candles. Latest:`, candles[candles.length - 1]);

    // Test 3: Verify candle format
    const candle = candles[0];
    const hasAllFields = ['time', 'open', 'high', 'low', 'close', 'volume']
        .every(field => typeof candle[field] === 'number');
    console.log(`✓ Candle format valid:`, hasAllFields);

    console.log('--- All tests passed ---');
}

testDataProvider().catch(console.error);
```

**Test criteria:**
- [ ] Console shows "Got X symbols" with a number > 50
- [ ] Console shows "Got 10 candles" with candle data
- [ ] Candle format valid: true
- [ ] No CORS errors (Hyperliquid API allows browser requests)

**Commit message:** `feat(data): implement Hyperliquid REST API for symbols and candles`

---

### Task 2.3: Implement Hyperliquid WebSocket (subscribeCandles)

**Goal:** Subscribe to real-time candle updates via WebSocket.

**File to modify:** `index.html`

**Replace the placeholder `subscribeCandles` and `unsubscribe` methods:**
```javascript
    /**
     * Subscribe to real-time candle updates
     * @param {string} symbol
     * @param {string} interval
     * @param {function} onCandle - callback receiving normalized candle
     */
    subscribeCandles(symbol, interval, onCandle) {
        // Close existing connection if any
        this.unsubscribe();

        this.ws = new WebSocket(this.wsUrl);
        this._onCandle = onCandle;
        this._currentSymbol = symbol;
        this._currentInterval = interval;

        this.ws.onopen = () => {
            console.log('WebSocket connected');
            // Subscribe to candle updates
            this.ws.send(JSON.stringify({
                method: 'subscribe',
                subscription: {
                    type: 'candle',
                    coin: symbol,
                    interval: interval
                }
            }));
        };

        this.ws.onmessage = (event) => {
            try {
                const message = JSON.parse(event.data);

                // Handle candle updates
                if (message.channel === 'candle' && message.data) {
                    const candle = message.data;
                    const normalized = {
                        time: Math.floor(candle.t / 1000),
                        open: parseFloat(candle.o),
                        high: parseFloat(candle.h),
                        low: parseFloat(candle.l),
                        close: parseFloat(candle.c),
                        volume: parseFloat(candle.v)
                    };
                    this._onCandle(normalized);
                }
            } catch (err) {
                console.error('WebSocket message parse error:', err);
            }
        };

        this.ws.onerror = (error) => {
            console.error('WebSocket error:', error);
        };

        this.ws.onclose = () => {
            console.log('WebSocket closed');
        };
    },

    /**
     * Unsubscribe and close WebSocket connection
     */
    unsubscribe() {
        if (this.ws) {
            // Send unsubscribe before closing
            if (this.ws.readyState === WebSocket.OPEN) {
                this.ws.send(JSON.stringify({
                    method: 'unsubscribe',
                    subscription: {
                        type: 'candle',
                        coin: this._currentSymbol,
                        interval: this._currentInterval
                    }
                }));
            }
            this.ws.close();
            this.ws = null;
        }
    }
```

**Update test code:**
```javascript
async function testDataProvider() {
    console.log('--- Testing DataProvider ---');

    // Test 1: Fetch symbols
    console.log('Test 1: Fetching symbols...');
    const symbols = await dataProvider.getSymbols();
    console.log(`✓ Got ${symbols.length} symbols`);

    // Test 2: Fetch candles
    console.log('Test 2: Fetching BTC 1h candles...');
    const candles = await dataProvider.fetchCandles('BTC', '1h', 10);
    console.log(`✓ Got ${candles.length} candles`);

    // Test 3: WebSocket subscription
    console.log('Test 3: Testing WebSocket (5 second test)...');
    let updateCount = 0;
    dataProvider.subscribeCandles('BTC', '1m', (candle) => {
        updateCount++;
        console.log(`  WebSocket update #${updateCount}:`, candle.close);
    });

    // Stop after 5 seconds
    setTimeout(() => {
        dataProvider.unsubscribe();
        console.log(`✓ Received ${updateCount} WebSocket updates in 5 seconds`);
        console.log('--- All tests passed ---');
    }, 5000);
}

testDataProvider().catch(console.error);
```

**Test criteria:**
- [ ] Console shows "WebSocket connected"
- [ ] Console shows WebSocket updates with price values
- [ ] After 5 seconds, shows "Received X WebSocket updates"
- [ ] Console shows "WebSocket closed" after test completes

**Commit message:** `feat(data): implement Hyperliquid WebSocket for real-time candles`

---

### Task 2.4: Add Connection State Management

**Goal:** Track connection state and implement retry logic.

**File to modify:** `index.html`

**Add after CONFIG:**
```javascript
// ============================================================
// SECTION 1.5: CONNECTION STATE
// ============================================================

const ConnectionState = {
    CONNECTING: 'connecting',
    CONNECTED: 'connected',
    RECONNECTING: 'reconnecting',
    DISCONNECTED: 'disconnected'
};

let connectionState = ConnectionState.DISCONNECTED;
let connectionStateListeners = [];

function setConnectionState(state) {
    connectionState = state;
    connectionStateListeners.forEach(fn => fn(state));
    console.log('Connection state:', state);
}

function onConnectionStateChange(listener) {
    connectionStateListeners.push(listener);
}
```

**Update HyperliquidProvider's `subscribeCandles`:**
```javascript
    subscribeCandles(symbol, interval, onCandle) {
        this.unsubscribe();
        setConnectionState(ConnectionState.CONNECTING);

        this.ws = new WebSocket(this.wsUrl);
        this._onCandle = onCandle;
        this._currentSymbol = symbol;
        this._currentInterval = interval;
        this._reconnectAttempt = 0;

        this.ws.onopen = () => {
            console.log('WebSocket connected');
            setConnectionState(ConnectionState.CONNECTED);
            this._reconnectAttempt = 0;

            this.ws.send(JSON.stringify({
                method: 'subscribe',
                subscription: {
                    type: 'candle',
                    coin: symbol,
                    interval: interval
                }
            }));
        };

        this.ws.onmessage = (event) => {
            try {
                const message = JSON.parse(event.data);
                if (message.channel === 'candle' && message.data) {
                    const candle = message.data;
                    const normalized = {
                        time: Math.floor(candle.t / 1000),
                        open: parseFloat(candle.o),
                        high: parseFloat(candle.h),
                        low: parseFloat(candle.l),
                        close: parseFloat(candle.c),
                        volume: parseFloat(candle.v)
                    };
                    this._onCandle(normalized);
                }
            } catch (err) {
                console.error('WebSocket message parse error:', err);
            }
        };

        this.ws.onerror = (error) => {
            console.error('WebSocket error:', error);
        };

        this.ws.onclose = () => {
            console.log('WebSocket closed');
            this._handleReconnect(symbol, interval, onCandle);
        };
    },

    _handleReconnect(symbol, interval, onCandle) {
        if (this._reconnectAttempt >= CONFIG.retryDelays.length) {
            setConnectionState(ConnectionState.DISCONNECTED);
            console.error('Max reconnection attempts reached');
            return;
        }

        setConnectionState(ConnectionState.RECONNECTING);
        const delay = CONFIG.retryDelays[this._reconnectAttempt];
        console.log(`Reconnecting in ${delay}ms... (attempt ${this._reconnectAttempt + 1})`);

        setTimeout(() => {
            this._reconnectAttempt++;
            this.subscribeCandles(symbol, interval, onCandle);
        }, delay);
    },
```

**Remove the test code from previous task** — we'll test manually now.

**Test criteria:**
- [ ] Console shows "Connection state: connecting" then "connected"
- [ ] If you disconnect network briefly, see "reconnecting" state
- [ ] After reconnection, state returns to "connected"

**Commit message:** `feat(data): add connection state management with auto-reconnect`

---

## Phase 3: Basic UI Shell

### Task 3.1: Create HTML Layout Structure

**Goal:** Build the two-row header + chart container layout.

**File to modify:** `index.html`

**Replace `<body>` content:**
```html
<body>
    <!-- Row 1: Symbol, Timeframe, Status -->
    <div id="row1" class="control-row">
        <div class="control-group">
            <select id="symbol-select">
                <option value="BTC">BTC</option>
            </select>
            <select id="timeframe-select">
                <!-- Populated by JS -->
            </select>
        </div>
        <div class="status-indicator" id="status-indicator">
            <span class="status-dot"></span>
            <span class="status-text">Connecting...</span>
        </div>
    </div>

    <!-- Row 2: Strategies, Threshold, Confluence -->
    <div id="row2" class="control-row">
        <div class="strategy-slots" id="strategy-slots">
            <div class="strategy-slot" id="strategy-slot-1">
                <select class="strategy-select" data-slot="1">
                    <option value="">+ Add Strategy</option>
                </select>
                <button class="settings-btn" data-slot="1" style="display:none;">⚙</button>
            </div>
            <!-- More slots added dynamically -->
        </div>
        <div class="confluence-controls">
            <label>
                Threshold:
                <input type="range" id="threshold-slider" min="1" max="5" value="3">
                <span id="threshold-value">3</span>
            </label>
            <div class="confluence-badge" id="confluence-badge">
                <span>No strategies</span>
            </div>
        </div>
    </div>

    <!-- Settings Panel (hidden by default) -->
    <div id="settings-panel" class="settings-panel" style="display:none;">
        <div class="settings-header">
            <span id="settings-title">Strategy Settings</span>
            <button id="settings-close">×</button>
        </div>
        <div id="settings-content">
            <!-- Populated dynamically -->
        </div>
    </div>

    <!-- Chart Container -->
    <div id="chart-container"></div>

    <!-- Signal Log (collapsed) -->
    <div id="signal-log" class="signal-log collapsed">
        <div class="signal-log-header" id="signal-log-toggle">
            <span>▶ Signals (<span id="signal-count">0</span>)</span>
            <button id="export-csv-btn">Export CSV</button>
        </div>
        <div class="signal-log-content" id="signal-log-content">
            <!-- Populated dynamically -->
        </div>
    </div>

    <!-- Toast Container -->
    <div id="toast-container"></div>

    <script>
        // ... existing JS code ...
    </script>
</body>
```

**Test criteria:**
- [ ] Page shows two control rows at top
- [ ] Chart container area is visible (empty)
- [ ] Signal log bar at bottom
- [ ] All elements have IDs matching the design

**Commit message:** `feat(ui): create HTML layout structure`

---

### Task 3.2: Style the Layout with CSS

**Goal:** Apply dark theme styling to the layout.

**File to modify:** `index.html`

**Add to `<style>` (after utility classes):**
```css
/* ========== Layout ========== */
.control-row {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 10px 16px;
    background: var(--bg-secondary);
    border-bottom: 1px solid var(--border-color);
}

.control-group {
    display: flex;
    gap: 12px;
    align-items: center;
}

/* ========== Status Indicator ========== */
.status-indicator {
    display: flex;
    align-items: center;
    gap: 6px;
    font-size: 12px;
}

.status-dot {
    width: 8px;
    height: 8px;
    border-radius: 50%;
    background: var(--accent-yellow);
}

.status-dot.connected { background: var(--accent-green); }
.status-dot.reconnecting { background: var(--accent-yellow); }
.status-dot.disconnected { background: var(--accent-red); }

/* ========== Dropdowns ========== */
select {
    padding: 8px 12px;
    background: var(--bg-tertiary);
    color: var(--text-primary);
    border: 1px solid var(--border-color);
    border-radius: 4px;
    font-size: 14px;
    cursor: pointer;
}

select:hover {
    border-color: var(--accent-blue);
}

/* ========== Strategy Slots ========== */
.strategy-slots {
    display: flex;
    gap: 8px;
    flex-wrap: wrap;
}

.strategy-slot {
    display: flex;
    align-items: center;
    gap: 4px;
}

.strategy-select {
    min-width: 160px;
}

.settings-btn {
    padding: 6px 10px;
    background: var(--bg-tertiary);
    border: 1px solid var(--border-color);
    border-radius: 4px;
    color: var(--text-primary);
    cursor: pointer;
}

.settings-btn:hover {
    background: var(--bg-primary);
}

/* ========== Confluence Controls ========== */
.confluence-controls {
    display: flex;
    align-items: center;
    gap: 16px;
}

.confluence-controls label {
    display: flex;
    align-items: center;
    gap: 8px;
    font-size: 12px;
    color: var(--text-secondary);
}

#threshold-slider {
    width: 80px;
}

#threshold-value {
    min-width: 16px;
    text-align: center;
}

.confluence-badge {
    padding: 6px 12px;
    background: var(--bg-tertiary);
    border: 1px solid var(--border-color);
    border-radius: 4px;
    font-size: 12px;
    font-weight: 600;
}

.confluence-badge.bullish {
    border-color: var(--accent-green);
    color: var(--accent-green);
}

.confluence-badge.bearish {
    border-color: var(--accent-red);
    color: var(--accent-red);
}

/* ========== Settings Panel ========== */
.settings-panel {
    background: var(--bg-secondary);
    border-bottom: 1px solid var(--border-color);
    padding: 12px 16px;
}

.settings-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 12px;
}

#settings-close {
    background: none;
    border: none;
    color: var(--text-secondary);
    font-size: 20px;
    cursor: pointer;
}

/* ========== Chart Container ========== */
#chart-container {
    flex: 1;
    min-height: 400px;
    background: var(--bg-tertiary);
}

/* ========== Signal Log ========== */
.signal-log {
    background: var(--bg-secondary);
    border-top: 1px solid var(--border-color);
}

.signal-log-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 10px 16px;
    cursor: pointer;
}

.signal-log-header:hover {
    background: var(--bg-tertiary);
}

.signal-log-content {
    max-height: 200px;
    overflow-y: auto;
    display: none;
}

.signal-log.expanded .signal-log-content {
    display: block;
}

.signal-log.expanded .signal-log-header span::before {
    content: '▼ ';
}

.signal-log.collapsed .signal-log-header span::before {
    content: '▶ ';
}

.signal-log-header span::before {
    content: '';
}

#export-csv-btn {
    padding: 4px 12px;
    background: var(--bg-tertiary);
    border: 1px solid var(--border-color);
    border-radius: 4px;
    color: var(--text-primary);
    font-size: 12px;
    cursor: pointer;
}

/* ========== Toast Notifications ========== */
#toast-container {
    position: fixed;
    top: 16px;
    right: 16px;
    z-index: 1000;
    display: flex;
    flex-direction: column;
    gap: 8px;
}

.toast {
    padding: 12px 16px;
    background: var(--bg-secondary);
    border: 1px solid var(--border-color);
    border-radius: 4px;
    color: var(--text-primary);
    font-size: 13px;
    animation: slideIn 0.3s ease;
}

.toast.success { border-color: var(--accent-green); }
.toast.error { border-color: var(--accent-red); }
.toast.warning { border-color: var(--accent-yellow); }

@keyframes slideIn {
    from {
        transform: translateX(100%);
        opacity: 0;
    }
    to {
        transform: translateX(0);
        opacity: 1;
    }
}
```

**Test criteria:**
- [ ] Page has proper dark theme appearance
- [ ] Two control rows are visually distinct
- [ ] Status indicator shows yellow dot
- [ ] Dropdowns have dark styling
- [ ] Layout fills the viewport height

**Commit message:** `style: apply dark theme styling to layout components`

---

### Task 3.3: Initialize Chart with Lightweight Charts

**Goal:** Create a basic candlestick chart that resizes properly.

**File to modify:** `index.html`

**Add to JavaScript (new section after HyperliquidProvider):**
```javascript
// ============================================================
// SECTION 4: CHART MANAGER
// ============================================================

const ChartManager = {
    chart: null,
    candleSeries: null,
    container: null,

    /**
     * Initialize the chart
     */
    init() {
        this.container = document.getElementById('chart-container');

        this.chart = LightweightCharts.createChart(this.container, {
            layout: {
                background: { type: 'solid', color: '#0f0f1a' },
                textColor: '#a0a0a0',
            },
            grid: {
                vertLines: { color: '#1a1a2e' },
                horzLines: { color: '#1a1a2e' },
            },
            crosshair: {
                mode: LightweightCharts.CrosshairMode.Normal,
            },
            rightPriceScale: {
                borderColor: '#2a2a4a',
            },
            timeScale: {
                borderColor: '#2a2a4a',
                timeVisible: true,
                secondsVisible: false,
            },
        });

        this.candleSeries = this.chart.addCandlestickSeries({
            upColor: '#00c853',
            downColor: '#ff5252',
            borderUpColor: '#00c853',
            borderDownColor: '#ff5252',
            wickUpColor: '#00c853',
            wickDownColor: '#ff5252',
        });

        // Handle resize
        window.addEventListener('resize', () => this.resize());
        this.resize();
    },

    /**
     * Resize chart to fit container
     */
    resize() {
        if (this.chart && this.container) {
            this.chart.applyOptions({
                width: this.container.clientWidth,
                height: this.container.clientHeight,
            });
        }
    },

    /**
     * Set candle data
     * @param {Candle[]} candles
     */
    setData(candles) {
        if (this.candleSeries) {
            this.candleSeries.setData(candles);
            this.chart.timeScale().fitContent();
        }
    },

    /**
     * Update the latest candle
     * @param {Candle} candle
     */
    updateCandle(candle) {
        if (this.candleSeries) {
            this.candleSeries.update(candle);
        }
    },

    /**
     * Add markers to the chart
     * @param {Marker[]} markers
     */
    setMarkers(markers) {
        if (this.candleSeries) {
            this.candleSeries.setMarkers(markers);
        }
    }
};
```

**Add initialization at end of script:**
```javascript
// ============================================================
// APP INITIALIZATION
// ============================================================

async function initApp() {
    console.log('Initializing app...');

    // Initialize chart
    ChartManager.init();
    console.log('Chart initialized');

    // Load initial data
    try {
        const candles = await dataProvider.fetchCandles(
            CONFIG.defaultSymbol,
            CONFIG.defaultTimeframe,
            CONFIG.candleLimit
        );
        console.log(`Loaded ${candles.length} candles`);

        ChartManager.setData(candles);
        console.log('Chart data set');

    } catch (error) {
        console.error('Failed to initialize:', error);
    }
}

// Start the app when DOM is ready
document.addEventListener('DOMContentLoaded', initApp);
```

**Test criteria:**
- [ ] Candlestick chart renders in the chart container
- [ ] Chart shows BTC price data
- [ ] Green candles for up, red candles for down
- [ ] Chart resizes when window is resized
- [ ] Crosshair works on hover

**Commit message:** `feat(chart): initialize Lightweight Charts with candlestick series`

---

### Task 3.4: Wire Up Real-Time Updates

**Goal:** Connect WebSocket updates to the chart.

**File to modify:** `index.html`

**Update `initApp()` function:**
```javascript
async function initApp() {
    console.log('Initializing app...');

    // Initialize chart
    ChartManager.init();

    // Track candles in memory
    let candles = [];

    // Load initial data
    try {
        candles = await dataProvider.fetchCandles(
            CONFIG.defaultSymbol,
            CONFIG.defaultTimeframe,
            CONFIG.candleLimit
        );
        console.log(`Loaded ${candles.length} candles`);
        ChartManager.setData(candles);

        // Subscribe to real-time updates
        dataProvider.subscribeCandles(
            CONFIG.defaultSymbol,
            CONFIG.defaultTimeframe,
            (newCandle) => {
                // Check if this updates the last candle or adds a new one
                const lastCandle = candles[candles.length - 1];

                if (lastCandle && lastCandle.time === newCandle.time) {
                    // Update existing candle
                    candles[candles.length - 1] = newCandle;
                } else {
                    // New candle - append and trim
                    candles.push(newCandle);
                    if (candles.length > CONFIG.candleLimit) {
                        candles.shift();
                    }
                }

                ChartManager.updateCandle(newCandle);
            }
        );

    } catch (error) {
        console.error('Failed to initialize:', error);
    }
}
```

**Update status indicator based on connection state (add before initApp):**
```javascript
// ============================================================
// SECTION 5: UI CONTROLLER
// ============================================================

const UIController = {
    updateStatus(state) {
        const dot = document.querySelector('.status-dot');
        const text = document.querySelector('.status-text');

        dot.className = 'status-dot ' + state;

        const labels = {
            connecting: 'Connecting...',
            connected: 'Live',
            reconnecting: 'Reconnecting...',
            disconnected: 'Disconnected'
        };

        text.textContent = labels[state] || state;
    }
};

// Listen to connection state changes
onConnectionStateChange((state) => {
    UIController.updateStatus(state);
});
```

**Test criteria:**
- [ ] Chart loads with historical data
- [ ] Latest candle updates in real-time (watch the rightmost candle)
- [ ] Status indicator shows "Live" with green dot when connected
- [ ] Price changes are reflected immediately

**Commit message:** `feat(chart): wire up real-time WebSocket updates to chart`

---

## Phase 4: Core Controls

### Task 4.1: Populate and Handle Timeframe Selector

**Goal:** Let users switch between timeframes.

**File to modify:** `index.html`

**Add to UIController:**
```javascript
    initTimeframeSelect() {
        const select = document.getElementById('timeframe-select');

        // Populate options
        TIMEFRAMES.forEach(tf => {
            const option = document.createElement('option');
            option.value = tf;
            option.textContent = tf;
            if (tf === CONFIG.defaultTimeframe) {
                option.selected = true;
            }
            select.appendChild(option);
        });

        // Handle changes
        select.addEventListener('change', (e) => {
            const newTimeframe = e.target.value;
            console.log('Timeframe changed to:', newTimeframe);
            AppState.setTimeframe(newTimeframe);
        });
    },
```

**Add AppState management (new section):**
```javascript
// ============================================================
// SECTION 6: APP STATE
// ============================================================

const AppState = {
    symbol: CONFIG.defaultSymbol,
    timeframe: CONFIG.defaultTimeframe,
    strategies: [],
    threshold: CONFIG.defaultThreshold,
    candles: [],

    async setTimeframe(tf) {
        this.timeframe = tf;
        await this.reload();
    },

    async setSymbol(sym) {
        this.symbol = sym;
        await this.reload();
    },

    async reload() {
        console.log(`Reloading: ${this.symbol} ${this.timeframe}`);

        // Unsubscribe from current feed
        dataProvider.unsubscribe();

        // Fetch new data
        this.candles = await dataProvider.fetchCandles(
            this.symbol,
            this.timeframe,
            CONFIG.candleLimit
        );

        ChartManager.setData(this.candles);

        // Resubscribe
        dataProvider.subscribeCandles(
            this.symbol,
            this.timeframe,
            (newCandle) => {
                const lastCandle = this.candles[this.candles.length - 1];
                if (lastCandle && lastCandle.time === newCandle.time) {
                    this.candles[this.candles.length - 1] = newCandle;
                } else {
                    this.candles.push(newCandle);
                    if (this.candles.length > CONFIG.candleLimit) {
                        this.candles.shift();
                    }
                }
                ChartManager.updateCandle(newCandle);
            }
        );
    }
};
```

**Update initApp() to use UIController:**
```javascript
async function initApp() {
    console.log('Initializing app...');

    ChartManager.init();
    UIController.initTimeframeSelect();

    await AppState.reload();
}
```

**Test criteria:**
- [ ] Timeframe dropdown shows all 14 options
- [ ] Default is "1h"
- [ ] Changing timeframe reloads chart with new data
- [ ] WebSocket reconnects for new timeframe

**Commit message:** `feat(ui): implement timeframe selector with state management`

---

### Task 4.2: Implement Symbol Selector with Choices.js

**Goal:** Create a searchable symbol dropdown using Choices.js.

**File to modify:** `index.html`

**Add to UIController:**
```javascript
    symbolChoices: null,

    async initSymbolSelect() {
        const select = document.getElementById('symbol-select');

        // Fetch available symbols
        const symbols = await dataProvider.getSymbols();

        // Clear and populate
        select.innerHTML = '';
        symbols.forEach(sym => {
            const option = document.createElement('option');
            option.value = sym.symbol;
            option.textContent = sym.symbol;
            if (sym.symbol === CONFIG.defaultSymbol) {
                option.selected = true;
            }
            select.appendChild(option);
        });

        // Initialize Choices.js
        this.symbolChoices = new Choices(select, {
            searchEnabled: true,
            searchPlaceholderValue: 'Search symbols...',
            itemSelectText: '',
            shouldSort: false,
        });

        // Handle changes
        select.addEventListener('change', (e) => {
            const newSymbol = e.target.value;
            console.log('Symbol changed to:', newSymbol);
            AppState.setSymbol(newSymbol);
        });
    },
```

**Update initApp():**
```javascript
async function initApp() {
    console.log('Initializing app...');

    ChartManager.init();
    UIController.initTimeframeSelect();
    await UIController.initSymbolSelect();

    await AppState.reload();
}
```

**Add Choices.js dark theme CSS overrides to `<style>`:**
```css
/* ========== Choices.js Overrides ========== */
.choices {
    margin-bottom: 0;
}

.choices__inner {
    background: var(--bg-tertiary);
    border-color: var(--border-color);
    min-height: 38px;
    padding: 4px 8px;
}

.choices__list--single {
    padding: 4px 16px 4px 4px;
}

.choices__list--dropdown {
    background: var(--bg-secondary);
    border-color: var(--border-color);
}

.choices__list--dropdown .choices__item--selectable.is-highlighted {
    background: var(--bg-tertiary);
}

.choices__input {
    background: var(--bg-tertiary);
    color: var(--text-primary);
}

.choices[data-type*="select-one"] .choices__input {
    background: var(--bg-secondary);
}
```

**Test criteria:**
- [ ] Symbol dropdown is searchable
- [ ] Shows all Hyperliquid symbols (50+)
- [ ] Typing filters the list
- [ ] Selecting a symbol reloads the chart
- [ ] Dark theme styling applied

**Commit message:** `feat(ui): implement searchable symbol selector with Choices.js`

---

### Task 4.3: Implement Threshold Slider

**Goal:** Make the threshold slider functional.

**File to modify:** `index.html`

**Add to UIController:**
```javascript
    initThresholdSlider() {
        const slider = document.getElementById('threshold-slider');
        const valueDisplay = document.getElementById('threshold-value');

        // Set initial value
        slider.value = CONFIG.defaultThreshold;
        valueDisplay.textContent = CONFIG.defaultThreshold;

        // Handle changes
        slider.addEventListener('input', (e) => {
            const value = parseInt(e.target.value);
            valueDisplay.textContent = value;
            AppState.setThreshold(value);
        });
    },
```

**Add to AppState:**
```javascript
    setThreshold(value) {
        this.threshold = value;
        console.log('Threshold set to:', value);
        // Recalculate confluence (implemented later)
        this.recalculateSignals();
    },

    recalculateSignals() {
        // Placeholder - will be implemented with strategies
        console.log('Recalculating signals...');
    }
```

**Update initApp():**
```javascript
async function initApp() {
    console.log('Initializing app...');

    ChartManager.init();
    UIController.initTimeframeSelect();
    UIController.initThresholdSlider();
    await UIController.initSymbolSelect();

    await AppState.reload();
}
```

**Test criteria:**
- [ ] Slider moves between 1 and 5
- [ ] Value display updates as slider moves
- [ ] Console logs threshold changes

**Commit message:** `feat(ui): implement threshold slider control`

---

### Task 4.4: Implement Signal Log Toggle

**Goal:** Make the signal log collapsible.

**File to modify:** `index.html`

**Add to UIController:**
```javascript
    initSignalLog() {
        const log = document.getElementById('signal-log');
        const toggle = document.getElementById('signal-log-toggle');

        toggle.addEventListener('click', () => {
            log.classList.toggle('collapsed');
            log.classList.toggle('expanded');
        });
    },

    addSignalToLog(signal) {
        const content = document.getElementById('signal-log-content');
        const countEl = document.getElementById('signal-count');

        const entry = document.createElement('div');
        entry.className = 'signal-entry';
        entry.innerHTML = `
            <span class="signal-time">${this.formatTime(signal.time)}</span>
            <span class="signal-strategy">${signal.strategyName}</span>
            <span class="signal-type ${signal.type}">${signal.type === 'buy' ? '▲' : '▼'} ${signal.type}</span>
            <span class="signal-price">$${signal.price.toLocaleString()}</span>
        `;

        content.insertBefore(entry, content.firstChild);

        // Update count
        const count = content.children.length;
        countEl.textContent = count;
    },

    formatTime(timestamp) {
        const date = new Date(timestamp * 1000);
        return date.toLocaleTimeString('en-US', {
            hour: '2-digit',
            minute: '2-digit'
        });
    },

    clearSignalLog() {
        document.getElementById('signal-log-content').innerHTML = '';
        document.getElementById('signal-count').textContent = '0';
    }
```

**Add signal log entry CSS to `<style>`:**
```css
.signal-entry {
    display: flex;
    gap: 16px;
    padding: 8px 16px;
    border-bottom: 1px solid var(--border-color);
    font-size: 13px;
}

.signal-entry:hover {
    background: var(--bg-tertiary);
}

.signal-time {
    color: var(--text-secondary);
    min-width: 60px;
}

.signal-strategy {
    flex: 1;
}

.signal-type {
    min-width: 60px;
    font-weight: 600;
}

.signal-type.buy { color: var(--accent-green); }
.signal-type.sell { color: var(--accent-red); }

.signal-price {
    min-width: 100px;
    text-align: right;
}
```

**Update initApp():**
```javascript
async function initApp() {
    console.log('Initializing app...');

    ChartManager.init();
    UIController.initTimeframeSelect();
    UIController.initThresholdSlider();
    UIController.initSignalLog();
    await UIController.initSymbolSelect();

    await AppState.reload();
}
```

**Test criteria:**
- [ ] Clicking signal log header expands/collapses it
- [ ] Arrow icon changes direction (▶ vs ▼)

**Commit message:** `feat(ui): implement collapsible signal log`

---

### Task 4.5: Implement Toast Notifications

**Goal:** Show toast messages for events.

**File to modify:** `index.html`

**Add new section:**
```javascript
// ============================================================
// SECTION 7: TOAST MANAGER
// ============================================================

const ToastManager = {
    container: null,

    init() {
        this.container = document.getElementById('toast-container');
    },

    show(message, type = 'info', duration = 3000) {
        const toast = document.createElement('div');
        toast.className = `toast ${type}`;
        toast.textContent = message;

        this.container.appendChild(toast);

        if (duration > 0) {
            setTimeout(() => {
                toast.remove();
            }, duration);
        }

        return toast; // Return for manual removal
    },

    success(message, duration) {
        return this.show('✓ ' + message, 'success', duration);
    },

    error(message, duration) {
        return this.show('✗ ' + message, 'error', duration);
    },

    warning(message, duration) {
        return this.show('⚠ ' + message, 'warning', duration);
    }
};
```

**Update connection state listener:**
```javascript
onConnectionStateChange((state) => {
    UIController.updateStatus(state);

    // Show toasts for state changes
    if (state === ConnectionState.CONNECTED) {
        ToastManager.success('Connected to Hyperliquid');
    } else if (state === ConnectionState.RECONNECTING) {
        ToastManager.warning('Connection lost, reconnecting...', 0);
    } else if (state === ConnectionState.DISCONNECTED) {
        ToastManager.error('Disconnected. Check your network.', 0);
    }
});
```

**Update initApp():**
```javascript
async function initApp() {
    console.log('Initializing app...');

    ToastManager.init();
    ChartManager.init();
    UIController.initTimeframeSelect();
    UIController.initThresholdSlider();
    UIController.initSignalLog();
    await UIController.initSymbolSelect();

    await AppState.reload();
}
```

**Test criteria:**
- [ ] Toast appears on connection
- [ ] Toast slides in from right
- [ ] Toast auto-dismisses after 3 seconds
- [ ] Different colors for success/warning/error

**Commit message:** `feat(ui): implement toast notification system`

---

## Phase 5: Strategy Engine

### Task 5.1: Create Strategy Registry Structure

**Goal:** Define how strategies are registered and organized.

**File to modify:** `index.html`

**Add new section:**
```javascript
// ============================================================
// SECTION 8: STRATEGY REGISTRY
// ============================================================

const StrategyCategories = {
    TREND: 'Trend Following',
    REVERSION: 'Mean Reversion',
    MOMENTUM: 'Momentum',
    VOLATILITY: 'Volatility',
    SUPPORT: 'Support/Resistance',
    OPTIONS: 'Options-Adapted',
    ML: 'Machine Learning',
    MULTI: 'Multi-Factor'
};

// All strategies will be registered here
const StrategyRegistry = [];

/**
 * Register a strategy
 */
function registerStrategy(strategy) {
    // Validate required fields
    const required = ['id', 'name', 'category', 'calculate'];
    for (const field of required) {
        if (!strategy[field]) {
            throw new Error(`Strategy missing required field: ${field}`);
        }
    }

    // Add defaults
    strategy.params = strategy.params || {};
    strategy.description = strategy.description || '';
    strategy.warning = strategy.warning || null;
    strategy.sourceSection = strategy.sourceSection || '';

    StrategyRegistry.push(strategy);
}

/**
 * Get strategies grouped by category
 */
function getStrategiesByCategory() {
    const grouped = {};

    for (const strategy of StrategyRegistry) {
        if (!grouped[strategy.category]) {
            grouped[strategy.category] = [];
        }
        grouped[strategy.category].push(strategy);
    }

    return grouped;
}

/**
 * Get a strategy by ID
 */
function getStrategyById(id) {
    return StrategyRegistry.find(s => s.id === id);
}
```

**Test criteria:**
- [ ] `registerStrategy()` function exists
- [ ] `getStrategiesByCategory()` returns empty object (no strategies yet)
- [ ] No JavaScript errors

**Commit message:** `feat(strategy): create strategy registry structure`

---

### Task 5.2: Implement First Strategy (SMA)

**Goal:** Implement the Single Moving Average strategy as a template.

**Reference:** `references/151.txt` section 3.11

**File to modify:** `index.html`

**Add after StrategyRegistry code:**
```javascript
// ============================================================
// SECTION 9: STRATEGY IMPLEMENTATIONS
// ============================================================

// --- Helper Functions ---

function calculateSMA(candles, period, priceKey = 'close') {
    const result = [];
    for (let i = 0; i < candles.length; i++) {
        if (i < period - 1) {
            result.push(null);
        } else {
            let sum = 0;
            for (let j = 0; j < period; j++) {
                sum += candles[i - j][priceKey];
            }
            result.push(sum / period);
        }
    }
    return result;
}

// --- Strategy: Single Moving Average (Section 3.11) ---

registerStrategy({
    id: 'sma',
    name: 'Single Moving Average',
    category: StrategyCategories.TREND,
    sourceSection: '3.11',
    description: 'Buy when price crosses above SMA, sell when below.',

    params: {
        period: { default: 20, min: 2, max: 500, label: 'Period' }
    },

    calculate(candles, params) {
        const period = params.period || this.params.period.default;
        const sma = calculateSMA(candles, period);
        const signals = [];

        for (let i = 1; i < candles.length; i++) {
            if (sma[i] === null || sma[i - 1] === null) continue;

            const prevPrice = candles[i - 1].close;
            const currPrice = candles[i].close;
            const prevSMA = sma[i - 1];
            const currSMA = sma[i];

            // Price crossed above SMA
            if (prevPrice <= prevSMA && currPrice > currSMA) {
                signals.push({
                    time: candles[i].time,
                    type: 'buy',
                    strength: 1,
                    reason: `Price crossed above SMA(${period})`
                });
            }
            // Price crossed below SMA
            else if (prevPrice >= prevSMA && currPrice < currSMA) {
                signals.push({
                    time: candles[i].time,
                    type: 'sell',
                    strength: 1,
                    reason: `Price crossed below SMA(${period})`
                });
            }
        }

        return signals;
    }
});
```

**Add test code (temporary):**
```javascript
// Test SMA strategy
const testCandles = [
    { time: 1, close: 100 },
    { time: 2, close: 102 },
    { time: 3, close: 101 },
    { time: 4, close: 105 },
    { time: 5, close: 108 },
    { time: 6, close: 103 },
    { time: 7, close: 99 },
];

const smaStrategy = getStrategyById('sma');
const signals = smaStrategy.calculate(testCandles, { period: 3 });
console.log('SMA test signals:', signals);
```

**Test criteria:**
- [ ] Strategy is registered in StrategyRegistry
- [ ] `getStrategyById('sma')` returns the strategy
- [ ] Calculate returns signals array
- [ ] Signals have correct format (time, type, strength, reason)

**Commit message:** `feat(strategy): implement Single Moving Average (SMA) strategy`

---

### Task 5.3: Implement More Core Strategies

**Goal:** Add RSI, MACD, and Two Moving Averages strategies.

**File to modify:** `index.html`

**Add these strategies:**

```javascript
// --- Helper: Calculate EMA ---

function calculateEMA(candles, period, priceKey = 'close') {
    const result = [];
    const multiplier = 2 / (period + 1);

    for (let i = 0; i < candles.length; i++) {
        if (i === 0) {
            result.push(candles[i][priceKey]);
        } else if (i < period - 1) {
            // Use SMA for initial values
            let sum = 0;
            for (let j = 0; j <= i; j++) {
                sum += candles[j][priceKey];
            }
            result.push(sum / (i + 1));
        } else {
            const ema = (candles[i][priceKey] - result[i - 1]) * multiplier + result[i - 1];
            result.push(ema);
        }
    }
    return result;
}

// --- Helper: Calculate RSI ---

function calculateRSI(candles, period = 14) {
    const result = [];
    const gains = [];
    const losses = [];

    for (let i = 0; i < candles.length; i++) {
        if (i === 0) {
            result.push(null);
            continue;
        }

        const change = candles[i].close - candles[i - 1].close;
        gains.push(change > 0 ? change : 0);
        losses.push(change < 0 ? -change : 0);

        if (i < period) {
            result.push(null);
        } else {
            const avgGain = gains.slice(-period).reduce((a, b) => a + b, 0) / period;
            const avgLoss = losses.slice(-period).reduce((a, b) => a + b, 0) / period;

            if (avgLoss === 0) {
                result.push(100);
            } else {
                const rs = avgGain / avgLoss;
                result.push(100 - (100 / (1 + rs)));
            }
        }
    }
    return result;
}

// --- Strategy: RSI ---

registerStrategy({
    id: 'rsi',
    name: 'Relative Strength Index',
    category: StrategyCategories.REVERSION,
    sourceSection: '3.9',
    description: 'Buy when RSI crosses above oversold, sell when crosses below overbought.',

    params: {
        period: { default: 14, min: 2, max: 100, label: 'Period' },
        overbought: { default: 70, min: 50, max: 100, label: 'Overbought' },
        oversold: { default: 30, min: 0, max: 50, label: 'Oversold' }
    },

    calculate(candles, params) {
        const period = params.period || this.params.period.default;
        const overbought = params.overbought || this.params.overbought.default;
        const oversold = params.oversold || this.params.oversold.default;

        const rsi = calculateRSI(candles, period);
        const signals = [];

        for (let i = 1; i < candles.length; i++) {
            if (rsi[i] === null || rsi[i - 1] === null) continue;

            // RSI crossed above oversold
            if (rsi[i - 1] <= oversold && rsi[i] > oversold) {
                signals.push({
                    time: candles[i].time,
                    type: 'buy',
                    strength: 1,
                    reason: `RSI crossed above ${oversold} (oversold)`
                });
            }
            // RSI crossed below overbought
            else if (rsi[i - 1] >= overbought && rsi[i] < overbought) {
                signals.push({
                    time: candles[i].time,
                    type: 'sell',
                    strength: 1,
                    reason: `RSI crossed below ${overbought} (overbought)`
                });
            }
        }

        return signals;
    }
});

// --- Strategy: Two Moving Averages (Section 3.12) ---

registerStrategy({
    id: 'sma-cross',
    name: 'Two Moving Averages',
    category: StrategyCategories.TREND,
    sourceSection: '3.12',
    description: 'Buy when fast MA crosses above slow MA, sell when below.',

    params: {
        fastPeriod: { default: 10, min: 2, max: 200, label: 'Fast Period' },
        slowPeriod: { default: 30, min: 5, max: 500, label: 'Slow Period' }
    },

    calculate(candles, params) {
        const fastPeriod = params.fastPeriod || this.params.fastPeriod.default;
        const slowPeriod = params.slowPeriod || this.params.slowPeriod.default;

        const fastMA = calculateSMA(candles, fastPeriod);
        const slowMA = calculateSMA(candles, slowPeriod);
        const signals = [];

        for (let i = 1; i < candles.length; i++) {
            if (fastMA[i] === null || slowMA[i] === null ||
                fastMA[i - 1] === null || slowMA[i - 1] === null) continue;

            // Fast crossed above slow (Golden Cross)
            if (fastMA[i - 1] <= slowMA[i - 1] && fastMA[i] > slowMA[i]) {
                signals.push({
                    time: candles[i].time,
                    type: 'buy',
                    strength: 1,
                    reason: `SMA(${fastPeriod}) crossed above SMA(${slowPeriod})`
                });
            }
            // Fast crossed below slow (Death Cross)
            else if (fastMA[i - 1] >= slowMA[i - 1] && fastMA[i] < slowMA[i]) {
                signals.push({
                    time: candles[i].time,
                    type: 'sell',
                    strength: 1,
                    reason: `SMA(${fastPeriod}) crossed below SMA(${slowPeriod})`
                });
            }
        }

        return signals;
    }
});

// --- Strategy: MACD ---

registerStrategy({
    id: 'macd',
    name: 'MACD',
    category: StrategyCategories.MOMENTUM,
    sourceSection: '3.12',
    description: 'Buy on MACD line crossing above signal line, sell on crossing below.',

    params: {
        fastPeriod: { default: 12, min: 2, max: 100, label: 'Fast EMA' },
        slowPeriod: { default: 26, min: 5, max: 200, label: 'Slow EMA' },
        signalPeriod: { default: 9, min: 2, max: 50, label: 'Signal Period' }
    },

    calculate(candles, params) {
        const fastPeriod = params.fastPeriod || this.params.fastPeriod.default;
        const slowPeriod = params.slowPeriod || this.params.slowPeriod.default;
        const signalPeriod = params.signalPeriod || this.params.signalPeriod.default;

        const fastEMA = calculateEMA(candles, fastPeriod);
        const slowEMA = calculateEMA(candles, slowPeriod);

        // MACD line = Fast EMA - Slow EMA
        const macdLine = fastEMA.map((fast, i) =>
            (fast !== null && slowEMA[i] !== null) ? fast - slowEMA[i] : null
        );

        // Signal line = EMA of MACD line
        const signalLine = [];
        const multiplier = 2 / (signalPeriod + 1);
        for (let i = 0; i < macdLine.length; i++) {
            if (macdLine[i] === null) {
                signalLine.push(null);
            } else if (signalLine.length === 0 || signalLine[signalLine.length - 1] === null) {
                signalLine.push(macdLine[i]);
            } else {
                const signal = (macdLine[i] - signalLine[i - 1]) * multiplier + signalLine[i - 1];
                signalLine.push(signal);
            }
        }

        const signals = [];

        for (let i = slowPeriod + signalPeriod; i < candles.length; i++) {
            if (macdLine[i] === null || signalLine[i] === null ||
                macdLine[i - 1] === null || signalLine[i - 1] === null) continue;

            // MACD crossed above signal
            if (macdLine[i - 1] <= signalLine[i - 1] && macdLine[i] > signalLine[i]) {
                signals.push({
                    time: candles[i].time,
                    type: 'buy',
                    strength: 1,
                    reason: 'MACD crossed above signal line'
                });
            }
            // MACD crossed below signal
            else if (macdLine[i - 1] >= signalLine[i - 1] && macdLine[i] < signalLine[i]) {
                signals.push({
                    time: candles[i].time,
                    type: 'sell',
                    strength: 1,
                    reason: 'MACD crossed below signal line'
                });
            }
        }

        return signals;
    }
});

console.log('Registered strategies:', StrategyRegistry.map(s => s.name));
```

**Test criteria:**
- [ ] Console shows 4 registered strategies
- [ ] Each strategy can be retrieved by ID
- [ ] RSI, MACD, SMA-Cross all calculate without errors

**Commit message:** `feat(strategy): implement RSI, MACD, and Two Moving Averages strategies`

---

### Task 5.4: Wire Up Strategy Selection UI

**Goal:** Create strategy dropdowns that populate from the registry.

**File to modify:** `index.html`

**Add to UIController:**
```javascript
    strategyChoices: {},

    initStrategySelectors() {
        const grouped = getStrategiesByCategory();
        const slot = document.querySelector('.strategy-select[data-slot="1"]');

        // Build choices format
        const choices = [{ value: '', label: '+ Add Strategy', selected: true }];

        for (const [category, strategies] of Object.entries(grouped)) {
            choices.push({
                label: category,
                id: category,
                disabled: false,
                choices: strategies.map(s => ({
                    value: s.id,
                    label: s.warning ? `${s.name} ⚠️` : s.name,
                    customProperties: { warning: s.warning }
                }))
            });
        }

        // Initialize first slot with Choices.js
        this.strategyChoices[1] = new Choices(slot, {
            searchEnabled: true,
            searchPlaceholderValue: 'Search strategies...',
            itemSelectText: '',
            choices: choices,
            shouldSort: false,
        });

        slot.addEventListener('change', (e) => {
            const strategyId = e.target.value;
            if (strategyId) {
                this.onStrategySelected(1, strategyId);
            }
        });
    },

    onStrategySelected(slot, strategyId) {
        const strategy = getStrategyById(strategyId);
        if (!strategy) return;

        console.log(`Slot ${slot}: Selected ${strategy.name}`);

        // Show warning toast if adapted strategy
        if (strategy.warning) {
            ToastManager.warning(strategy.warning, 5000);
        }

        // Show settings button
        const settingsBtn = document.querySelector(`.settings-btn[data-slot="${slot}"]`);
        if (settingsBtn) {
            settingsBtn.style.display = 'inline-block';
        }

        // Add to AppState
        AppState.addStrategy(slot, strategyId, {});
    },
```

**Add to AppState:**
```javascript
    activeStrategies: {}, // { slot: { id, params, signals } }

    addStrategy(slot, strategyId, params) {
        const strategy = getStrategyById(strategyId);
        if (!strategy) return;

        // Merge with defaults
        const finalParams = {};
        for (const [key, config] of Object.entries(strategy.params)) {
            finalParams[key] = params[key] !== undefined ? params[key] : config.default;
        }

        this.activeStrategies[slot] = {
            id: strategyId,
            params: finalParams,
            signals: []
        };

        this.recalculateSignals();
    },

    removeStrategy(slot) {
        delete this.activeStrategies[slot];
        this.recalculateSignals();
    },

    recalculateSignals() {
        console.log('Recalculating signals for', Object.keys(this.activeStrategies).length, 'strategies');

        for (const [slot, config] of Object.entries(this.activeStrategies)) {
            const strategy = getStrategyById(config.id);
            if (strategy && this.candles.length > 0) {
                config.signals = strategy.calculate(this.candles, config.params);
                console.log(`Slot ${slot} (${strategy.name}): ${config.signals.length} signals`);
            }
        }

        // Update chart markers
        this.updateChartMarkers();
    },

    updateChartMarkers() {
        const colors = ['#2962FF', '#FF6D00', '#00C853', '#AA00FF', '#FFD600'];
        const allMarkers = [];

        for (const [slot, config] of Object.entries(this.activeStrategies)) {
            const color = colors[(parseInt(slot) - 1) % colors.length];

            for (const signal of config.signals) {
                allMarkers.push({
                    time: signal.time,
                    position: signal.type === 'buy' ? 'belowBar' : 'aboveBar',
                    color: color,
                    shape: signal.type === 'buy' ? 'arrowUp' : 'arrowDown',
                    text: signal.type === 'buy' ? '▲' : '▼'
                });
            }
        }

        // Sort by time
        allMarkers.sort((a, b) => a.time - b.time);
        ChartManager.setMarkers(allMarkers);
    }
```

**Update initApp():**
```javascript
async function initApp() {
    console.log('Initializing app...');

    ToastManager.init();
    ChartManager.init();
    UIController.initTimeframeSelect();
    UIController.initThresholdSlider();
    UIController.initSignalLog();
    UIController.initStrategySelectors();
    await UIController.initSymbolSelect();

    await AppState.reload();
}
```

**Test criteria:**
- [ ] Strategy dropdown shows categories with strategies
- [ ] Selecting a strategy shows log message
- [ ] Strategy warning shows toast if applicable
- [ ] Markers appear on chart after selection

**Commit message:** `feat(ui): wire up strategy selection with chart markers`

---

## Phase 6: Remaining Implementation Tasks

The remaining tasks follow the same pattern. For brevity, here are the task summaries:

### Task 6.1: Implement Strategy Settings Panel
- Show/hide settings when gear icon clicked
- Populate with strategy parameters
- Apply button updates params and recalculates

### Task 6.2: Implement Confluence Badge
- Calculate confluence from active strategy signals
- Update badge text and color
- Apply threshold filtering

### Task 6.3: Implement Background Zones
- Add series for confluence zones
- Update when confluence changes

### Task 6.4: Implement CSV Export
- Collect all signals
- Format as CSV
- Trigger download

### Task 6.5: Implement URL State
- Parse URL hash on load
- Update URL hash on state change
- Handle invalid params gracefully

---

## Phase 7: Add Remaining Strategies

### Task 7.1-7.10: Implement All 151 Strategies

Each strategy follows the same pattern as Task 5.2-5.3. Implement in batches:

1. **Trend Following** (~15 strategies): Three MA, EMA variants, etc.
2. **Mean Reversion** (~12 strategies): Bollinger Bands, etc.
3. **Momentum** (~10 strategies): Price momentum, etc.
4. **Volatility** (~8 strategies): ATR, etc.
5. **Support/Resistance** (~8 strategies): Pivot points, Donchian, etc.
6. **Options-Adapted** (~60 strategies): Simplified signals based on outlook
7. **Machine Learning** (~5 strategies): Simplified KNN, etc.
8. **Multi-Factor** (~10 strategies): Combined signals

For Options-Adapted strategies, include warning:
```javascript
warning: 'Adapted from options strategy. Original requires strike prices and expirations.'
```

---

## Phase 8: Final Polish

### Task 8.1: Add README Documentation
- Usage instructions
- Strategy list
- Screenshots

### Task 8.2: Add Favicon
- Create simple icon
- Place in assets/

### Task 8.3: Final Testing Checklist

Manual testing checklist:

- [ ] Page loads without errors
- [ ] Symbols load and are searchable
- [ ] All 14 timeframes work
- [ ] Strategies appear in grouped dropdown
- [ ] Selecting strategy shows markers
- [ ] Multiple strategies (up to 5) work together
- [ ] Confluence badge updates correctly
- [ ] Threshold slider affects signals
- [ ] Settings panel works for all strategies
- [ ] Signal log shows entries
- [ ] CSV export downloads file
- [ ] URL sharing works (copy URL, open in new tab)
- [ ] Connection status indicator works
- [ ] Toasts appear for events
- [ ] WebSocket reconnection works (disable/enable network)

### Task 8.4: Final Commit and Push

```bash
git add -A
git commit -m "feat: complete 151 trading strategies visualization tool"
git push -u origin claude/brainstorm-feature-design-QxcIk
```

---

## Quick Reference

### File Locations

| What | Where |
|------|-------|
| Main app | `index.html` |
| Design doc | `docs/plans/2026-01-20-trading-strategies-tool-design.md` |
| This plan | `docs/plans/2026-01-20-trading-strategies-tool-implementation.md` |
| Strategy reference | `references/151.txt` |

### Key Commands

```bash
# Serve locally
python3 -m http.server 8080

# Check for JS errors
# Open browser DevTools → Console

# Commit pattern
git add index.html
git commit -m "feat(scope): description"
```

### Commit Message Format

```
type(scope): description

Types: feat, fix, style, refactor, docs, test
Scopes: data, ui, chart, strategy, state
```

---

## Troubleshooting

### "Choices is not defined"
CDN failed to load. Check network tab. Try different CDN or version.

### Chart not rendering
Check container has height. Add `min-height: 400px` to `#chart-container`.

### WebSocket not connecting
Check browser console for CORS errors. Hyperliquid should allow browser connections.

### Strategies not calculating
Check candles array has data. Check for NaN in calculations.

### Markers not showing
Check marker times match candle times (both in seconds, not milliseconds).
