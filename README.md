//@version=6
indicator("Paradise Trading Pro v6 - Final", overlay=true, max_bars_back=500)

// ================================================================================
// PARADISE TRADING PRO - FINAL VERSION (NO TELEGRAM)
// Features: Liquidity Zones, RSI, MACD, Volume, TP/SL, Buy/Sell Signals
// ================================================================================

// ===== INPUT PARAMETERS - BASIC =====
lookback = input.int(20, title="Swing Lookback Period", minval=5, maxval=100)
atrLength = input.int(14, title="ATR Length", minval=5, maxval=100)
maLength = input.int(20, title="Moving Average Length", minval=5, maxval=200)
maType = input.string("SMA", title="MA Type", options=["SMA", "EMA", "WMA"])

// ===== INPUT PARAMETERS - RSI =====
rsiEnabled = input.bool(true, title="Enable RSI Indicator")
rsiLength = input.int(14, title="RSI Length", minval=5, maxval=100)
rsiOverbought = input.int(70, title="RSI Overbought Level", minval=50, maxval=100)
rsiOversold = input.int(30, title="RSI Oversold Level", minval=0, maxval=50)

// ===== INPUT PARAMETERS - MACD =====
macdEnabled = input.bool(true, title="Enable MACD Indicator")
macdFast = input.int(12, title="MACD Fast EMA", minval=5, maxval=50)
macdSlow = input.int(26, title="MACD Slow EMA", minval=5, maxval=100)
macdSignalLen = input.int(9, title="MACD Signal Line", minval=5, maxval=50)

// ===== INPUT PARAMETERS - VOLUME =====
volumeEnabled = input.bool(true, title="Enable Volume Analysis")
volumeMaLen = input.int(20, title="Volume MA Length", minval=5, maxval=100)

// ===== INPUT PARAMETERS - TP/SL =====
tpslEnabled = input.bool(true, title="Enable TP/SL Calculation")
tpMultiplier = input.float(2.0, title="TP Multiplier (ATR x)", minval=0.5, maxval=5.0, step=0.1)
slMultiplier = input.float(1.5, title="SL Multiplier (ATR x)", minval=0.5, maxval=5.0, step=0.1)

// ===== INPUT PARAMETERS - VISUALS =====
showLiquidityZones = input.bool(true, title="Show Liquidity Zones")
showZoneFill = input.bool(true, title="Fill Liquidity Zone")
showTrendMA = input.bool(true, title="Show Trend Moving Average")
showBuySellSignals = input.bool(true, title="Show Buy/Sell Signals")
showTPSL = input.bool(true, title="Show TP/SL Levels")
showInfoTable = input.bool(true, title="Show Info Table")

// ===== CORE CALCULATIONS =====

// 1. LIQUIDITY ZONES - SWING HIGHS & LOWS
highestPrice = ta.highest(high, lookback)
lowestPrice = ta.lowest(low, lookback)
resistance = highestPrice
support = lowestPrice

// 2. VOLATILITY - ATR
atrValue = ta.atr(atrLength)

// 3. TREND MOVING AVERAGE
trendMA = if maType == "EMA"
    ta.ema(close, maLength)
else if maType == "WMA"
    ta.wma(close, maLength)
else
    ta.sma(close, maLength)

isTrendUp = close > trendMA
isTrendDown = close < trendMA

// 4. RSI CALCULATION
rsiValue = ta.rsi(close, rsiLength)
isRSIOverbought = rsiValue > rsiOverbought
isRSIOversold = rsiValue < rsiOversold
isRSINeutral = rsiValue >= rsiOversold and rsiValue <= rsiOverbought

// 5. MACD CALCULATION
[macdLine, macdSignal, macdHist] = ta.macd(close, macdFast, macdSlow, macdSignalLen)
isMACDBullish = macdLine > macdSignal
isMACDBearish = macdLine < macdSignal

// 6. VOLUME ANALYSIS
volumeMA = ta.sma(volume, volumeMaLen)
isVolumeHigh = volume > volumeMA
isVolumeLow = volume < volumeMA

// 7. TAKE PROFIT & STOP LOSS LEVELS
takeProfitLevel = close + (atrValue * tpMultiplier)
stopLossLevel = close - (atrValue * slMultiplier)
riskRewardRatio = (takeProfitLevel - close) / math.abs(close - stopLossLevel)

// ===== BUY SIGNAL CONDITIONS =====
// Buy Signal: Price bounces from support with bullish confirmation
buyCondition1 = close > support and close[1] <= support
buyCondition2 = isTrendUp
buyCondition3 = (not rsiEnabled) or isRSINeutral or isRSIOversold
buyCondition4 = (not macdEnabled) or isMACDBullish
buyCondition5 = (not volumeEnabled) or isVolumeHigh

isBuySignal = showBuySellSignals and buyCondition1 and buyCondition2 and buyCondition3 and buyCondition4 and buyCondition5

// ===== SELL SIGNAL CONDITIONS =====
// Sell Signal: Price rejects resistance with bearish confirmation
sellCondition1 = close < resistance and close[1] >= resistance
sellCondition2 = isTrendDown
sellCondition3 = (not rsiEnabled) or isRSINeutral or isRSIOverbought
sellCondition4 = (not macdEnabled) or isMACDBearish
sellCondition5 = (not volumeEnabled) or isVolumeHigh

isSellSignal = showBuySellSignals and sellCondition1 and sellCondition2 and sellCondition3 and sellCondition4 and sellCondition5

// ===== PLOTTING: LIQUIDITY ZONES =====
plot(showLiquidityZones ? resistance : na, 
     title="Resistance Zone", 
     color=color.new(color.red, 0), 
     linewidth=2, 
     style=plot.style_dashed)

plot(showLiquidityZones ? support : na, 
     title="Support Zone", 
     color=color.new(color.green, 0), 
     linewidth=2, 
     style=plot.style_dashed)

// Fill Liquidity Zone
p_high = plot(showLiquidityZones and showZoneFill ? resistance : na, display=display.none)
p_low = plot(showLiquidityZones and showZoneFill ? support : na, display=display.none)
fill(p_high, p_low, color=color.new(color.blue, 90), title="Liquidity Zone Fill")

// ===== PLOTTING: TREND MOVING AVERAGE =====
plot(showTrendMA ? trendMA : na, 
     title="Trend MA", 
     color=color.orange, 
     linewidth=2)

// ===== PLOTTING: TP/SL LEVELS =====
plot(showTPSL and tpslEnabled ? takeProfitLevel : na, 
     title="Take Profit Level", 
     color=color.new(color.lime, 50), 
     linewidth=1, 
     style=plot.style_dashed)

plot(showTPSL and tpslEnabled ? stopLossLevel : na, 
     title="Stop Loss Level", 
     color=color.new(color.maroon, 50), 
     linewidth=1, 
     style=plot.style_dashed)

// ===== PLOTTING: BUY/SELL SIGNALS =====
plotshape(isBuySignal, 
          title="BUY Signal", 
          style=shape.labelup, 
          location=location.belowbar, 
          color=color.green, 
          text="BUY", 
          textcolor=color.white, 
          size=size.small)

plotshape(isSellSignal, 
          title="SELL Signal", 
          style=shape.labeldown, 
          location=location.abovebar, 
          color=color.red, 
          text="SELL", 
          textcolor=color.white, 
          size=size.small)

// ===== ALERTS (LOCAL ONLY - NO TELEGRAM) =====
alertcondition(isBuySignal, 
               title="🟢 BUY SIGNAL", 
               message="BUY SIGNAL: Price bounced from Support")

alertcondition(isSellSignal, 
               title="🔴 SELL SIGNAL", 
               message="SELL SIGNAL: Price rejected Resistance")

alertcondition(isRSIOverbought and isTrendUp, 
               title="⚠️ OVERBOUGHT", 
               message="WARNING: RSI is OVERBOUGHT")

alertcondition(isRSIOversold and isTrendDown, 
               title="⚠️ OVERSOLD", 
               message="WARNING: RSI is OVERSOLD")

// ===== INFORMATION TABLE =====
if showInfoTable
    infoTable = table.new(position.top_right, 3, 10, border_color=color.gray, border_width=1)
    
    // Header Row
    table.cell(infoTable, 0, 0, "PARADISE TRADING", bgcolor=color.blue, text_color=color.white, text_size=size.small)
    table.cell(infoTable, 1, 0, "VALUE", bgcolor=color.blue, text_color=color.white, text_size=size.small)
    table.cell(infoTable, 2, 0, "STATUS", bgcolor=color.blue, text_color=color.white, text_size=size.small)
    
    // Price
    table.cell(infoTable, 0, 1, "Current Price", bgcolor=color.new(color.gray, 50), text_color=color.white, text_size=size.small)
    table.cell(infoTable, 1, 1, str.tostring(close, "0.00000"), bgcolor=color.new(color.gray, 50), text_color=color.yellow, text_size=size.small)
    
    // Support
    table.cell(infoTable, 0, 2, "Support Level", bgcolor=color.new(color.green, 30), text_color=color.white, text_size=size.small)
    table.cell(infoTable, 1, 2, str.tostring(support, "0.00000"), bgcolor=color.new(color.green, 30), text_color=color.lime, text_size=size.small)
    
    // Resistance
    table.cell(infoTable, 0, 3, "Resistance Level", bgcolor=color.new(color.red, 30), text_color=color.white, text_size=size.small)
    table.cell(infoTable, 1, 3, str.tostring(resistance, "0.00000"), bgcolor=color.new(color.red, 30), text_color=color.orange, text_size=size.small)
    
    // Trend
    trendColor = isTrendUp ? color.new(color.green, 30) : color.new(color.red, 30)
    trendText = isTrendUp ? "UPTREND ↑" : "DOWNTREND ↓"
    trendTextColor = isTrendUp ? color.lime : color.red
    table.cell(infoTable, 0, 4, "Trend Direction", bgcolor=trendColor, text_color=color.white, text_size=size.small)
    table.cell(infoTable, 1, 4, trendText, bgcolor=trendColor, text_color=trendTextColor, text_size=size.small)
    
    // RSI
    rsiDisplay = rsiEnabled ? str.tostring(rsiValue, "0.00") : "OFF"
    rsiStatusColor = isRSIOverbought ? color.red : isRSIOversold ? color.green : color.gray
    rsiStatusText = isRSIOverbought ? "OB" : isRSIOversold ? "OS" : "NL"
    table.cell(infoTable, 0, 5, "RSI Value", bgcolor=color.new(color.purple, 30), text_color=color.white, text_size=size.small)
    table.cell(infoTable, 1, 5, rsiDisplay, bgcolor=color.new(color.purple, 30), text_color=rsiStatusColor, text_size=size.small)
    table.cell(infoTable, 2, 5, rsiStatusText, bgcolor=color.new(color.purple, 30), text_color=rsiStatusColor, text_size=size.small)
    
    // MACD
    macdDisplay = macdEnabled ? (isMACDBullish ? "BULLISH" : "BEARISH") : "OFF"
    macdTextColor = isMACDBullish ? color.green : color.red
    table.cell(infoTable, 0, 6, "MACD Status", bgcolor=color.new(color.teal, 30), text_color=color.white, text_size=size.small)
    table.cell(infoTable, 1, 6, macdDisplay, bgcolor=color.new(color.teal, 30), text_color=macdTextColor, text_size=size.small)
    
    // Volume
    volumeDisplay = volumeEnabled ? (isVolumeHigh ? "HIGH" : "LOW") : "OFF"
    volumeTextColor = isVolumeHigh ? color.green : color.orange
    table.cell(infoTable, 0, 7, "Volume Status", bgcolor=color.new(color.navy, 30), text_color=color.white, text_size=size.small)
    table.cell(infoTable, 1, 7, volumeDisplay, bgcolor=color.new(color.navy, 30), text_color=volumeTextColor, text_size=size.small)
    
    // Take Profit & Stop Loss
    table.cell(infoTable, 0, 8, "Take Profit", bgcolor=color.new(color.lime, 30), text_color=color.white, text_size=size.small)
    table.cell(infoTable, 1, 8, tpslEnabled ? str.tostring(takeProfitLevel, "0.00000") : "OFF", bgcolor=color.new(color.lime, 30), text_color=color.lime, text_size=size.small)
    table.cell(infoTable, 2, 8, "R:R", bgcolor=color.new(color.lime, 30), text_color=color.lime, text_size=size.small)
    
    table.cell(infoTable, 0, 9, "Stop Loss", bgcolor=color.new(color.maroon, 30), text_color=color.white, text_size=size.small)
    table.cell(infoTable, 1, 9, tpslEnabled ? str.tostring(stopLossLevel, "0.00000") : "OFF", bgcolor=color.new(color.maroon, 30), text_color=color.red, text_size=size.small)
    table.cell(infoTable, 2, 9, tpslEnabled ? str.tostring(riskRewardRatio, "0.00") : "OFF", bgcolor=color.new(color.maroon, 30), text_color=color.yellow, text_size=size.small)
