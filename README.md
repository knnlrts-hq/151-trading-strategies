# 151 Trading Strategies Visualization Tool

A powerful, interactive tool for visualizing and testing trading strategies on live cryptocurrency data from Hyperliquid.

## Features

- **Real-time Data**: Live candlestick charts with WebSocket updates from Hyperliquid
- **59 Trading Strategies**: Comprehensive set of technical analysis strategies across 8 categories
- **Multi-Strategy Analysis**: Run up to 5 strategies simultaneously with confluence detection
- **Customizable Parameters**: Fine-tune each strategy's parameters via settings panel
- **Signal Log**: Track all buy/sell signals with CSV export
- **URL State**: Share your analysis via URL parameters
- **Dark Theme**: Professional trading interface design

## Quick Start

```bash
# Option 1: Python
python3 -m http.server 8080

# Option 2: Node.js
npx serve .

# Then open http://localhost:8080
```

## Strategy Categories

### Trend Following (8 strategies)
- Single Moving Average (SMA)
- Two Moving Averages (SMA Cross)
- Three Moving Averages
- EMA Cross
- Donchian Channel
- Parabolic SAR
- MACD

### Mean Reversion (5 strategies)
- Bollinger Bands
- Stochastic Oscillator
- Williams %R
- Commodity Channel Index (CCI)
- Bollinger Band Squeeze

### Momentum (6 strategies)
- Relative Strength Index (RSI)
- Rate of Change (ROC)
- Average Directional Index (ADX)
- Price Momentum
- Chaikin Money Flow (CMF)
- Awesome Oscillator

### Volatility (5 strategies)
- ATR Trailing Stop
- Keltner Channel
- Volatility Breakout
- ATR Channel Breakout
- Volatility Contraction

### Support/Resistance (5 strategies)
- Pivot Points
- Fibonacci Retracement
- Donchian Breakout
- Support/Resistance Levels
- Ichimoku Cloud

### Options-Adapted (20 strategies)
Simplified signals based on options strategy concepts:
- Bull Call Spread, Bear Put Spread
- Long Straddle, Iron Condor
- Covered Call, Protective Put
- Collar, Butterfly Spread
- Calendar Spread, Diagonal Spread
- And more...

### Multi-Factor (10 strategies)
Combined indicators for robust signals:
- Triple Indicator
- Trend + Momentum
- Volume Price Trend (VPT)
- Elder Ray
- TRIX, Vortex Indicator
- Ultimate Oscillator
- Mass Index
- Know Sure Thing (KST)
- Coppock Curve

## Usage Guide

1. **Select Symbol**: Choose from 100+ cryptocurrency perpetuals
2. **Set Timeframe**: From 1m to 1M intervals
3. **Add Strategies**: Select up to 5 strategies to analyze
4. **Configure Parameters**: Click the gear icon to adjust settings
5. **Analyze Confluence**: View combined signal strength in the badge
6. **Export Signals**: Download signal history as CSV

## Technical Details

- Single HTML file (~6000 lines)
- Zero build dependencies
- Uses Lightweight Charts for visualization
- Choices.js for searchable dropdowns
- Hyperliquid API for real-time data

## Documentation

- [Design Document](docs/plans/2026-01-20-trading-strategies-tool-design.md)
- [Implementation Plan](docs/plans/2026-01-20-trading-strategies-tool-implementation.md)

## License

MIT License
