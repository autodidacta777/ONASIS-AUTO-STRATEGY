//@version=6
strategy("Onasis Auto Strategy PRO", overlay=true, default_qty_type=strategy.percent_of_equity, default_qty_value=10)

// =========================
// INPUTS
// =========================
htf = input.timeframe("60", "HTF")

tenkanLen = input.int(9)
kijunLen  = input.int(26)

bodyRatio = input.float(0.1, "Doji cuerpo %")
minRange  = input.float(0.5, "Rango mínimo")

sl_points = input.int(300, "Stop Loss (puntos)")
tp_points = input.int(600, "Take Profit (puntos)")

// =========================
// ICHIMOKU LTF
// =========================
tenkan = (ta.highest(high, tenkanLen) + ta.lowest(low, tenkanLen)) / 2
kijun  = (ta.highest(high, kijunLen)  + ta.lowest(low, kijunLen))  / 2

// =========================
// ICHIMOKU HTF
// =========================
htf_tenkan = request.security(syminfo.tickerid, htf, (ta.highest(high, tenkanLen) + ta.lowest(low, tenkanLen)) / 2)
htf_kijun  = request.security(syminfo.tickerid, htf, (ta.highest(high, kijunLen) + ta.lowest(low, kijunLen)) / 2)

// =========================
// DOJI
// =========================
body = math.abs(close - open)
range = high - low
isDoji = body <= range * bodyRatio and range > minRange

// =========================
// ZONA KIJUN
// =========================
nearKijun = math.abs(close - kijun) <= range

// =========================
// DIRECCIÓN
// =========================
bullLTF = tenkan > kijun
bearLTF = tenkan < kijun

bullHTF = htf_tenkan > htf_kijun
bearHTF = htf_tenkan < htf_kijun

// =========================
// CONDICIONES
// =========================
buySignal  = isDoji and nearKijun and bullLTF and bullHTF
sellSignal = isDoji and nearKijun and bearLTF and bearHTF

// =========================
// EJECUCIÓN
// =========================
if (buySignal)
    strategy.entry("BUY", strategy.long)

if (sellSignal)
    strategy.entry("SELL", strategy.short)

// =========================
// SL / TP
// =========================
strategy.exit("TP/SL BUY", from_entry="BUY", profit=tp_points, loss=sl_points)
strategy.exit("TP/SL SELL", from_entry="SELL", profit=tp_points, loss=sl_points)

// =========================
// VISUAL
// =========================
plotshape(buySignal,  location=location.belowbar, color=color.green, style=shape.labelup,   text="BUY")
plotshape(sellSignal, location=location.abovebar, color=color.red,   style=shape.labeldown, text="SELL")
