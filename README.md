# BTC 15-min Window Backtest

A browser-based backtesting dashboard for the BTC UP/DOWN 15-min Polymarket strategy.

Fetches real historical candles directly from Binance and runs your live bot's full indicator logic against them — no server, no install, no API keys.

**Live:** [oxdonyosa.github.io/friendly-sniffle](https://oxdonyosa.github.io/friendly-sniffle/)

---

## What it does

- Pulls real BTCUSDT 15-min candles from Binance for any date range
- Runs every candle through the same 10 filters the live bot uses
- Lets you pick any of the 96 daily 15-min windows to test
- Filters by candle direction — UP (green close), DOWN (red close), or both
- Shows equity curve, win rate by window, and a strategy verdict
- Compounds P&L trade by trade so you see realistic bankroll growth

---

## How to use

1. Open the live link above (or clone and open `index.html` locally)
2. Pick a **date range** — presets: 30 days, 3 months, 6 months, 1 year
3. Select **trading windows** from the dropdown — all 96 daily slots available
4. Set your **candle direction filter** — UP only, DOWN only, or both
5. Adjust **strategy parameters** to match your live bot settings
6. Click **Fetch candles & run backtest**

---

## Trading windows

All 96 daily windows (every 15 minutes across 24 hours ET) are in the dropdown, grouped by session:

| Session | Hours ET |
|---------|----------|
| Overnight | 12am – 4am |
| Pre-market | 4am – 9am |
| NYSE morning | 9am – 12pm |
| Afternoon | 12pm – 4pm |
| After-hours | 4pm – 8pm |
| Late night | 8pm – 12am |

**Priority windows** (★) — 8:30, 8:45, 9:00, 9:15 AM ET — marked in the list. These align with macro data releases and the NYSE open where the live bot has highest conviction.

**Quick select:** Priority windows · Market hours · Pre-market · All 96 · Clear all

---

## Candle direction filter

| Filter | What it tests |
|--------|--------------|
| Both | All candles regardless of close |
| UP only | Candles where `close > open` (green) |
| DOWN only | Candles where `close < open` (red) |

Use this to find out whether your strategy performs better in bullish or bearish conditions.

---

## The 10 filters

| # | Filter | Condition |
|---|--------|-----------|
| 1 | ATR expanded | Current ATR > 20-period average |
| 2 | VWAP clear | Price > 0.15% away from VWAP |
| 3 | EMA aligned | EMA 9/21 direction agrees with VWAP |
| 4 | EMA diverging | EMA gap widening, not converging |
| 5 | MACD accelerating | Histogram growing in same direction |
| 6 | Volume confirmed | Volume ≥ 1.3x 20-period average |
| 7 | Body committed | Candle body ratio ≥ 0.6 (no dojis) |
| 8 | RSI(8) aligned | RSI above/below 50 agrees with direction |
| 9 | BB compressed | BB width below 85th percentile |
| 10 | Chainlink spread | Assumed OK in backtest |

Direction is determined by majority vote of VWAP, EMA, MACD, and RSI signals. Resolution is based on whether the **next candle** closed UP or DOWN from its open.

---

## Strategy parameters

| Parameter | Default | Description |
|-----------|---------|-------------|
| Start balance | $100 | Initial bankroll |
| Risk per trade | 5% | Fraction of balance per signal |
| Strong edge threshold | 30% | Edge % that fires a STRONG signal |
| Normal edge min | 15% | Minimum edge for a NORMAL signal |
| Min filters required | 6/10 | Filters that must pass |

---

## Reading the results

**Win rate** — above 55% on real candles is meaningful. 50% = coin flip, no edge.

**Max drawdown** — largest peak-to-trough drop. Keep under 30% for live trading.

**Equity curve** — compounded balance over time. Smooth upward slope beats a volatile one.

**Win rate by window** — green = above 55%, amber = 45–55%, red = below 45%. Shows which time slots actually work.

**Strategy verdict:**

| Verdict | Meaning |
|---------|---------|
| Validated on real data | Win rate, return, and drawdown all healthy |
| Marginal positive edge | Slight profitability — needs more data to confirm |
| Profitable but risky | Good return but drawdown too high — lower risk |
| No consistent edge | Break-even — keep tuning filters |
| Losing on real data | Strategy losing money — filters need work |

---

## Data source

Candles: `data-api.binance.vision` — Binance's public browser-friendly endpoint. No API key needed. Batched at 1000 candles per request with a small delay between calls.

A 1-year backtest fetches ~35,000 candles across ~35 requests and takes about 15 seconds.

---

## Limitations

- Chainlink spread check is skipped — not needed in backtesting since Binance is the data source and there is no Chainlink price to compare against
- No slippage or fees modelled
- VWAP resets daily at UTC midnight — close but not exact to a session-anchored VWAP

**Resolution method:** uses `next candle open vs current candle open` — the closest available proxy to what Polymarket actually measures (Chainlink price at window close vs window open). More accurate than next-candle close.

For the most accurate edge measurement, log real trades from the live bot and replay those.

---

## Related

- **Twitter** — [@don_yosa](https://x.com/don_yosa)

- **Live signal bot** — deploys on Railway, sends Telegram signals before each 15-min window
- **Polymarket market slug pattern** — `btc-updown-15m-{unix_timestamp}`
- **Resolution source** — Chainlink BTC/USD on Ethereum mainnet
