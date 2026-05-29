# Heikin-Ashi Candlestick Alert — MQL4 Script

A MetaTrader 4 script that computes **Heikin-Ashi (HA) candlestick values** natively from raw OHLC data each cycle using the standard four-component averaging formula, tracks the smoothed open and close across consecutive iterations via persistent `PrevHAOpen` and `PrevHAClose` globals, and fires independent alerts for both **trend reversal** (HA candle color flip from the prior bar) and **trend continuation** (consecutive same-color HA candles) — with `AlertReversals` and `AlertContinuations` boolean flags allowing selective monitoring of either signal class independently.

---

## Overview

Heikin-Ashi, meaning "average bar" in Japanese, reconstructs each candlestick using averaged OHLC values to produce a smoothed representation of price action that filters out much of the noise inherent in standard candlestick charts. The defining characteristic of Heikin-Ashi is the recursive open: `HAOpen = (PrevHAOpen + PrevHAClose) / 2`. This averaging of the prior bar's open and close into the current bar's open creates a continuous smoothing chain that links each bar to its predecessor — producing long, uninterrupted sequences of same-color candles during strong trends, and short mixed sequences during consolidations or transitions. A color flip from the prior bar signals a potential reversal; a color match signals continuation. This script implements both signal types natively without requiring `iCustom()`, maintaining the HA open chain through `PrevHAOpen` and `PrevHAClose` persistent globals that persist the recursive link across the monitoring loop's minute-by-minute iterations.

> **Note on file naming:** This file is distributed as `Trend_Strength_Indicator__TSI__001.mq4` but implements a Heikin-Ashi Candlestick Alert. The README documents the actual implemented logic.

---

## Features

- **Native HA construction** — `HAClose = (O + C + H + L) / 4`; `HAOpen = (PrevHAOpen + PrevHAClose) / 2`; `HAHigh = MathMax(MathMax(H, HAClose), HAOpen)`; `HALow = MathMin(MathMin(L, HAClose), HAOpen)` — all four HA values computed fresh each cycle
- **Reversal detection** — `AlertReversals` gate with `PrevHAOpen != 0 && PrevHAClose != 0` first-cycle guard; `PrevHAClose > PrevHAOpen && HAClose < HAOpen` → **Bearish Reversal Detected**; `PrevHAClose < PrevHAOpen && HAClose > HAOpen` → **Bullish Reversal Detected**
- **Continuation detection** — `AlertContinuations` gate; `HAClose > HAOpen && PrevHAClose > PrevHAOpen` → **Bullish Trend Continuation**; `HAClose < HAOpen && PrevHAClose < PrevHAOpen` → **Bearish Trend Continuation** — both current and prior bars must match in color for continuation to fire
- **Independent alert gating** — `AlertReversals` and `AlertContinuations` evaluated in separate `if` blocks; both can be active simultaneously without interference
- **Persistent HA chain** — `PrevHAOpen = HAOpen` and `PrevHAClose = HAClose` updated unconditionally at cycle end
- **Three notification channels:** sound alert, email, and mobile push
- **Lightweight loop** — polls once per minute (`Sleep(60000)`)

---

## How It Works

1. Every minute, `CalculateHeikinAshi()` computes all four HA values using raw OHLC and prior-cycle state
2. If `AlertReversals && PrevHAOpen != 0 && PrevHAClose != 0`: prior/current color flip conditions evaluated
3. If `AlertContinuations`: same-color consecutive bar conditions evaluated independently
4. `PrevHAOpen = HAOpen` and `PrevHAClose = HAClose` updated at cycle end

---

## Input Parameters

| Parameter              | Type            | Default     | Description                                                     |
|------------------------|-----------------|-------------|-----------------------------------------------------------------|
| `TradeSymbol`          | string          | `EURUSD`    | Symbol for analysis                                             |
| `Timeframe`            | ENUM_TIMEFRAMES | `PERIOD_H1` | Timeframe for HA computation                                    |
| `AlertReversals`       | bool            | `true`      | Fire alerts on HA candle color flips (trend reversals)          |
| `AlertContinuations`   | bool            | `true`      | Fire alerts on consecutive same-color HA bars (continuations)   |
| `EnableAlerts`         | bool            | `true`      | Fire an on-screen/sound alert                                   |
| `EnableEmail`          | bool            | `false`     | Send an email notification                                      |
| `EnablePush`           | bool            | `false`     | Send a mobile push notification                                 |

---

## Alert Message Format

```
Bullish Reversal Detected detected on EURUSD (Timeframe: PERIOD_H1)
Price: 1.08420
```

---

## Installation

1. Copy `Trend_Strength_Indicator__TSI__001.mq4` to `MQL4/Scripts/` in your MT4 data folder
2. Compile in MetaEditor (F7)
3. Drag onto any chart from Navigator → Scripts
4. Configure inputs and click **OK**

---

## Requirements

- MetaTrader 4 (`#property strict` compatible build)
- MQL4 compiler (MetaEditor)

---

## License

MIT License

Copyright (c) 2026

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
