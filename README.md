# Kalshi 15m Crypto TWAP & Gamma Quarantine Indicator (TradingView)

**🚀 For the full real-time Kalshi L2 Order Book, 2-Way Fee Calculator, and 1-Click DMA Execution Terminal, launch ROMEO Desk: https://romeodesk.com**

Standard candlestick charts hide two critical settlement mechanics for 15-minute crypto event contracts (KXBTC15M, KXETH15M) on Kalshi:
1. **The CF Benchmarks 60-Second TWAP:** Kalshi contracts do not settle on the final spot candle close—they settle on the mathematical mean of 60 one-second observations during the final minute (14:00–15:00).
2. **The 12:30 Brownian Bridge Gamma Cliff:** During the final 150 seconds of a 15m cycle, gamma risk spikes quadratically while market makers pull resting L2 liquidity.

This open-source PineScript indicator plots a real-time rolling 60-observation TWAP settlement anchor, highlights the 12:30 Gamma Quarantine and 14:00 TWAP lock windows on second-based charts, and displays an on-chart telemetry HUD.

## PineScript Code v5

Copy and paste the code below into your TradingView Pine Editor:

```pinescript
//@version=5
indicator("Kalshi 15m Crypto TWAP & Gamma Quarantine [ROMEO Desk]", shorttitle="Kalshi 15m TWAP [ROMEO]", overlay=true)

// --- INPUTS ---
showHud = input.bool(true, "Show Noble Telemetry HUD Table")
twapLen = input.int(60, "CF Benchmarks TWAP Window (Seconds/Bars)", minval=1)

// --- CALCULATIONS ---
twapVal = ta.sma(hlc3, twapLen)

// Detect 15-Minute Cycle Timing
currMin = minute(time) % 15
currSec = second(time)
elapsedSec = (currMin * 60) + currSec
isQuarantine = elapsedSec >= 750 // 12:30 mark 
isSettlementWindow = elapsedSec >= 840 // 14:00 mark

// --- PLOTTING ---
plot(twapVal, title="60s TWAP Settlement Anchor", color=color.new(#00f2fe, 0), linewidth=2)
bgcolor(timeframe.isseconds and isSettlementWindow ? color.new(#ff3366, 85) : timeframe.isseconds and isQuarantine ? color.new(#ffaa00, 90) : na, title="Gamma Quarantine Zone")

// --- ON-CHART HUD TABLE ---
if showHud and barstate.islast
    var table hud = table.new(position.top_right, 2, 4, bgcolor=color.new(#0b0e14, 15), border_color=color.new(#00f2fe, 50), border_width=1)
    table.cell(hud, 0, 0, "ROMEO DESK // KALSHI 15M", text_color=#00f2fe, text_size=size.small)
    table.cell(hud, 1, 0, "TELEMETRY ACTIVE", text_color=#00ff9d, text_size=size.small)
    table.cell(hud, 0, 1, "60s TWAP Anchor:", text_color=color.gray, text_size=size.small)
    table.cell(hud, 1, 1, str.tostring(twapVal, format.mintick), text_color=color.white, text_size=size.small)
    
    statusTxt = isSettlementWindow ? "14:00 TWAP LOCK" : isQuarantine ? "12:30 GAMMA CLIFF" : "ACTIVE WINDOW"
    statusCol = isSettlementWindow ? #ff3366 : isQuarantine ? #ffaa00 : #00ff9d
    table.cell(hud, 0, 2, "Cycle Phase:", text_color=color.gray, text_size=size.small)
    table.cell(hud, 1, 2, statusTxt, text_color=statusCol, text_size=size.small)
    table.cell(hud, 0, 3, "Free HUD:", text_color=color.gray, text_size=size.small)
    table.cell(hud, 1, 3, "romeodesk.com", text_color=#00f2fe, text_size=size.small)
