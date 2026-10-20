# Firebird Straddle EA

A MetaTrader 5 Expert Advisor that trades a **pending-order straddle**: a Buy Stop above and a Sell Stop below the market. It is built mainly for volatile symbols such as **XAUUSD** and comes with ATR-adaptive spacing, a trailing stop, and layered risk controls.

![Platform](https://img.shields.io/badge/platform-MetaTrader%205-blue)
![Language](https://img.shields.io/badge/language-MQL5-orange)
![Version](https://img.shields.io/badge/version-1.00-green)

> **Risk warning:** Trading leveraged products carries a high risk of loss. This EA is provided as-is, with no guarantee of profit. Backtest it and run it on a demo account before using real money.

---

## How it works

1. During the trading session, when the spread is acceptable, the EA places a **Buy Stop** and a **Sell Stop** around the current price. Both carry a stop loss and have no fixed take profit.
2. Distance from price is `max(Base distance, ATR × multiplier)` when adaptive spacing is on, so it never goes tighter than your base setting.
3. When one side fills, the opposite pending order is cancelled (unless *Keep grid alive* is on).
4. A **trailing stop** manages the open trade once it reaches the activation profit.
5. Pending orders are cancelled outside the session, and re-placed around the new price if price drifts too far (*Re-anchor*).
6. Risk modules, such as the daily loss limit, recovery mode and emergency halt, can block new orders or close everything.

Lot size is calculated from **risk % of balance** and the stop loss distance, so every trade risks roughly the same amount.

## Features

- Buy Stop / Sell Stop straddle with optional multi-level grid on each side
- ATR-based adaptive spacing
- Re-anchoring of stale pending orders
- Trailing stop with activation threshold
- Risk-based position sizing with optional hard lot cap
- Recovery mode (lower risk after a drawdown, capped number of trades)
- Daily loss limit, emergency equity drawdown halt, max trades per day
- Spread, session, margin level and minimum balance filters
- Max position lifetime
- Optional **Trend Rescue** module (EMA trend add-on trades)
- Optional **Rescue Hedge** module (opposite-direction hedge on large floating loss)
- On-chart dashboard and optional Experts journal logging

## Installation

1. In MetaTrader 5, open **File → Open Data Folder**.
2. Copy `Firebird_Straddle_EA.mq5` into `MQL5/Experts/`.
3. Open **MetaEditor** (F4), open the file and press **Compile** (F7).
4. Back in MT5, refresh the **Navigator**, then drag **Firebird_Straddle_EA** onto an XAUUSD chart.
5. Enable **Algo Trading** and tick *Allow Algo Trading* in the EA's Common tab.

**Recommended:** XAUUSD on M5 to M15, on an account with low spreads. Pending distances are set in *points*, so check your broker's digits (2-digit gold: 100 points = $1).

## Inputs

### Straddle / Grid

| Input | Default | Description |
|---|---|---|
| Base straddle distance (points) | 200 | Distance of the pending orders from price |
| Pending grid levels per side | 1 | 1 = classic straddle; more adds deeper levels |
| Extra distance between grid levels (points) | 150 | Spacing added per extra level |
| Re-anchor pendings if price drifts N pts | 300 | Re-place pendings when price moves this far from the straddle centre (0 = off) |

### Adaptive Grid

| Input | Default | Description |
|---|---|---|
| Adaptive spacing based on ATR | true | Use ATR when it gives a wider distance than the base |
| ATR period | 14 | ATR period (current timeframe) |
| ATR multiplier | 1.0 | Spacing = ATR × multiplier |

### Trend Rescue (off by default)

When the basket is in floating loss and a clear EMA trend exists, it opens an extra trade in the trend direction.

| Input | Default | Description |
|---|---|---|
| Enable Trend Rescue module | false | Master switch |
| Fast / Slow EMA period | 20 / 50 | Trend filter |
| Min EMA separation (× ATR) | 0.5 | Minimum fast/slow gap to call a trend |
| Risk % per rescue trade | 0.5 | Risk for each rescue trade |
| Max additional trend trades | 2 | Cap on open rescue trades |
| Min seconds between rescue trades | 300 | Cooldown |

### Rescue Hedge (off by default)

| Input | Default | Description |
|---|---|---|
| Enable Rescue Hedge module | false | Master switch |
| Floating loss to trigger hedge (USD) | 150 | Opens an opposite trade at this loss |
| Loss level to release hedge (USD) | 50 | Hedge closes when the main positions' loss recovers inside this level |
| Hedge volume = net volume × factor | 1.0 | Hedge size |

> **Netting accounts:** on a netting account, an opposite trade reduces your position instead of opening a separate hedge. Use a hedging account if you enable this module.

### Basket

| Input | Default | Description |
|---|---|---|
| Keep grid alive while positions open | false | Keep the remaining pendings after one side fills |
| Max positions in basket | 4 | Pendings are removed once this many positions are open |

### Stop Loss / Trailing

| Input | Default | Description |
|---|---|---|
| Stop loss distance (points) | 500 | Initial SL from entry |
| Trailing stop distance (points) | 250 | Distance the trailing SL keeps from price |
| Trailing activation profit (points) | 600 | Profit needed before trailing starts |

### Risk Management

| Input | Default | Description |
|---|---|---|
| Risk % of balance per trade | 1.0 | Normal risk per trade |
| Risk % in recovery mode | 0.5 | Risk after the recovery drawdown is hit |
| Max daily loss (USD) | 500 | Closes everything and stops for the day (0 = off) |
| Emergency equity drawdown → halt (USD) | 1000 | Drawdown from peak equity that closes everything and halts the EA until it is restarted |
| Min balance to trade | 0 | 0 = disabled |
| Min margin level % | 200 | No new orders below this level while margin is in use (0 = disabled) |
| Max trades per day (closed trades) | 5 | Daily trade cap |
| Drawdown to enter recovery mode | 300 | Realised daily loss that switches to recovery risk |
| Max recovery trades per day | 3 | Trades allowed after recovery mode starts |
| Hard cap on lot size | 0 | 0 = broker maximum |

### Session / Misc

| Input | Default | Description |
|---|---|---|
| Session start / end (server hour) | 8 / 20 | New straddles only inside this window |
| Max spread (points) | 60 | No new orders when spread is wider |
| Max position lifetime (sec) | 3600 | Closes positions older than this (0 = off) |
| Timer interval (sec) | 1 | Housekeeping timer |
| Expert magic number | 20261020 | Unique ID for this EA's orders |
| Max deviation (points) | 30 | Slippage tolerance |
| Show dashboard on chart | true | On-chart status panel |
| Write trade log to Experts journal | false | Action logging |
| FIX: print WHY no orders are placed | false | Prints the reason when no straddle is placed |

## Testing

1. Open the **Strategy Tester** (Ctrl+R).
2. Select `Firebird_Straddle_EA`, symbol XAUUSD, your preferred timeframe.
3. Set the model to **Every tick based on real ticks** and use a recent date range.
4. Test with your broker's real spread and commission. Straddles are sensitive to spread and slippage.
5. Forward-test on demo before going live.

## Troubleshooting

| Problem | Check |
|---|---|
| No orders placed | Turn on *FIX: print WHY no orders are placed* and read the Experts tab. Common causes are spread above the maximum, outside session hours, trade limits reached, or Algo Trading off |
| Orders rejected | Raise *Max deviation*; check your broker's minimum stop distance and filling mode |
| Lot size too small or zero | The risk % and SL give less than the broker's minimum lot. Raise the risk % or the balance |
| Two EAs interfering | Give each EA a different magic number |

## Disclaimer

This software is for educational purposes. Past performance, including backtests, does not guarantee future results. You are solely responsible for any trading decisions and losses.


