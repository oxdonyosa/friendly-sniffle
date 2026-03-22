# BTC 15-min Window Backtest

A browser-based backtesting dashboard for the BTC UP/DOWN 15-min Polymarket strategy. Fetches real historical candles from Binance and runs your live bot's indicator logic against them.

No server needed. No install. Just open the HTML file in a browser.

---

## Live demo

Host on GitHub Pages and visit `yourdomain.xyz/backtest.html`

---

## How to use

1. Download or clone this repo
2. Open `backtest.html` in any modern browser
3. Select a date range (30 days, 3 months, 6 months, or 1 year)
4. Pick your trading windows from the dropdown — all 96 daily 15-min slots are available
5. Adjust strategy parameters
6. Click **Fetch candles & run backtest**

The dashboard pulls real BTCUSDT 15-min candles from Binance's public API, runs every candle through the indicator checklist, and shows you how the strategy would have performed.

---

## What gets tested

Every candle is run through the same 10 filters the live bot uses:

| Filter | What it checks |
|--------|---------------|
| ATR expanded | Current ATR above 20-period average |
| VWAP clear | Price more than 0.15% away from VWAP |
| EMA aligned | EMA 9/21 pointing same direction as VWAP |
| EMA diverging | EMA gap widening, not converging |
| MACD accelerating | Histogram growing in same direction |
| Volume confirmed | Current candle ≥ 1.3x average volume |
| Body committed | Candle body ratio ≥ 0.6 (no dojis) |
| RSI(8) aligned | RSI above/below 50 agrees with direction |
| BB compressed | Bollinger Band width below 85th percentile |
| Chainlink spread | Assumed OK in backtest (no on-chain data) |

Direction is determined by majority vote of VWAP, EMA, MACD and RSI signals. Resolution is based on whether the **next candle** closed UP or DOWN from its open — the same logic Polymarket uses for the UP/DOWN market resolution.

---

## Window selector

All 96 daily windows (every 15 minutes across 24 hours) are available in the dropdown, grouped by session:

- **Overnight** — 12am to 4am ET
- **Pre-market** — 4am to 9am ET
- **NYSE morning** — 9am to 12pm ET
- **Afternoon** — 12pm to 4pm ET
- **After-hours** — 4pm to 8pm ET
- **Late night** — 8pm to 12am ET

Priority windows (8:30, 8:45, 9:00, 9:15 AM ET) are marked with ★ and are the recommended starting point — these align with macro data releases and the NYSE open where the live bot has the highest conviction.

Quick select buttons:
- **Priority windows** — 8:30, 8:45, 9:00, 9:15 AM ET
- **Market hours** — all windows 9am–4pm
- **Pre-market** — 4am–9am only
- **All 96** — test everything
- **Clear all** — reset

---

## Strategy parameters

| Parameter | Default | What it does |
|-----------|---------|--------------|
| Start balance | $100 | Initial bankroll |
| Risk per trade | 5% | Fraction of balance staked per signal |
| Strong edge threshold | 30% | Edge % that fires a STRONG signal |
| Normal edge min | 15% | Minimum edge for a standard signal |
| Min filters required | 6/10 | How many of the 10 filters must pass |

---

## Reading the results

**Win rate** — percentage of signals where the next candle went in the predicted direction. Anything consistently above 55% on real candles is meaningful.

**Max drawdown** — the largest peak-to-trough drop during the simulation. Keep this under 30% for live trading comfort.

**Equity curve** — the compounded balance over time. A smooth upward curve is better than a volatile one even if the final number is similar.

**Win rate by window chart** — shows which time slots performed best. Green = above 55%, amber = 45–55%, red = below 45%. Use this to decide which windows to enable on the live bot.

**Strategy verdict** — auto-generated summary:
- Validated on real data — win rate, return and drawdown all healthy
- Marginal positive edge — slight profitability, needs more data
- Profitable but risky — good return but drawdown too high, lower risk
- No consistent edge — break-even or losing, keep tuning
- Losing on real data — strategy losing money, filters need work

---

## Data source

Candles are fetched from `data-api.binance.vision` — Binance's public browser-friendly endpoint. No API key required. Requests are batched at 1000 candles per call with a small delay between batches to avoid rate limits.

A 1-year backtest fetches roughly 35,000 candles across ~35 requests and takes about 10–15 seconds.

---

## Limitations

- Chainlink spread check is assumed OK (no on-chain data in browser)
- VWAP is anchored to the start of the candle dataset, not the session open
- No slippage or trading fees are modelled
- Resolution proxy is next-candle open→close direction, which is a close but not exact match to Chainlink's actual resolution price

For the most accurate edge measurement, log real trades from the live bot (entry price, YES/NO price, outcome) and replay those instead.

---

## Related

- **Live bot** — `btc-window-bot/` — the Railway-deployed Telegram signal bot
- **Polymarket market** — `btc-updown-15m-{timestamp}` slug pattern
- **Resolution source** — Chainlink BTC/USD `0xF4030086522a5bEEa4988F8cA5B36dbC97BeE88c`
