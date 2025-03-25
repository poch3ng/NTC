我會幫你將策略改成使用 ATR（平均真實波幅，Average True Range）版本。根據你的要求，我假設你希望用 ATR 替代原本固定的 9% 漲幅門檻，作為啟動「跌破 10 日均線半出」的動態條件。這樣可以讓策略更靈活，適應不同標的的波動性。以下是逐步改進的過程和最終程式碼。

---

### 修改思路
1. **ATR 取代固定 9%**：
   - 原策略中，價格需從進場點漲 9% 後才啟用 10 日均線半出。
   - 改為 ATR 版本後，啟動門檻將基於 ATR，例如進場價格加上一定倍數的 ATR（如 2 倍 ATR）。
2. **保持其他邏輯**：
   - 進場仍為突破 40 日高點且 60 日均線向上。
   - 停損仍為跌破 20 日低點。
   - 出場仍為 10 日均線半出（在達到 ATR 門檻後）和 20 日均線全出。
3. **動態適應**：
   - ATR 會根據市場波動自動調整門檻，避免固定百分比在高波動或低波動標的上不適用的問題。

---

### 修改後的程式碼
以下是將策略改成 ATR 版本的 Pine Script 程式碼：

```pinescript
// This Pine Script™ code is subject to the Mozilla Public License 2.0 at https://mozilla.org/MPL/2.0/
// © poch3ng

//@version=5
strategy("BO", overlay = true, pyramiding = 5, initial_capital = 100000)

// 參數
lengthHigh = input.int(40, "突破高點週期")
lengthLow = input.int(20, "停損低點週期")
ma10Length = 10
ma20Length = 20
ma60Length = 60
positionPercent = input.float(1, "資金分配(%)") * 0.01
atrLength = input.int(14, "ATR 週期")  // ATR 計算週期，預設 14
atrMultiplier = input.float(2, "ATR 倍數", step=0.1)  // ATR 倍數，預設 2

// 計算均線
ma10 = ta.sma(close, ma10Length)
ma20 = ta.sma(close, ma20Length)
ma60 = ta.sma(close, ma60Length)
ma60Up = ma60[ma60Length-1] > ma60[ma60Length]

// 計算高低點
highestHigh = ta.highest(high, lengthHigh)
lowestLow = ta.lowest(low, lengthLow)

// 計算 ATR
atr = ta.atr(atrLength)

// 追蹤進場價格和利潤門檻
var float entryPrice = 0.0
var float profitTarget = 0.0
var bool profitTargetReached = false

// 進場條件：突破 40 日高點，且 60 日均線向上，且無持倉
if ma60Up and strategy.position_size == 0 
    positionSize = (strategy.initial_capital * positionPercent) / close
    strategy.entry("BreakHigh40", strategy.long, qty=positionSize, stop=highestHigh)
    entryPrice := close * 1.09  // 記錄進場價格（模擬追價）
    profitTarget := entryPrice + (atr * atrMultiplier)  // 動態利潤門檻：進場價 + ATR * 倍數
    profitTargetReached := false  // 重置標記

// 檢查是否達到利潤門檻
if strategy.position_size > 0 and close >= profitTarget
    profitTargetReached := true

// 出場邏輯
if strategy.position_size > 0
    // 停損：跌破 20 日低點（任何時候有效）
    strategy.exit("StopLoss20Low", from_entry="BreakHigh40", stop=lowestLow)

    // 部分出場：僅在達到 ATR 利潤門檻後啟用「跌破 10 日均線出一半」
    var bool hasHalfExited = false
    if profitTargetReached and close < ma10 and close > entryPrice and not hasHalfExited
        strategy.close("BreakHigh40", qty_percent=50, comment="Half Exit MA10")
        hasHalfExited := true

    // 全部出場：跌破 20 日均線
    if close < ma20
        strategy.close("BreakHigh40", comment="Full Exit MA20")
        hasHalfExited := false  // 重置標記

// 畫圖參考
plot(highestHigh, title="40日高點", color=color.green)
plot(lowestLow, title="20日低點", color=color.red)
plot(ma10, title="MA10", color=color.orange)
plot(ma20, title="MA20", color=color.blue)
plot(profitTarget, title="Profit Target (ATR)", color=color.yellow, style=plot.style_dashed)  // 可視化 ATR 門檻
```

---

### 修改重點說明
1. **新增 ATR 參數**：
   - `atrLength`：ATR 的計算週期，預設為 14（常用值，可調整）。
   - `atrMultiplier`：ATR 的倍數，預設為 2，決定利潤門檻的寬鬆程度。
   - `atr = ta.atr(atrLength)`：計算當前 ATR 值。

2. **動態利潤門檻**：
   - 原版：`profitTarget := entryPrice * 1.09`（固定 9%）。
   - ATR 版：`profitTarget := entryPrice + (atr * atrMultiplier)`，門檻隨 ATR 動態變化。
   - 例如，若 `entryPrice = 109`，當前 `atr = 5`，`atrMultiplier = 2`，則 `profitTarget = 109 + (5 * 2) = 119`。

3. **出場邏輯保持一致**：
   - 未達 `profitTarget` 前：僅有 20 日低點停損和 20 日均線全出。
   - 達到 `profitTarget` 後：啟用「跌破 10 日均線半出」。

4. **可視化**：
   - 新增 `plot(profitTarget)`，用虛線顯示動態利潤門檻，方便觀察。

---

### ATR 版的優勢
1. **適應性強**：
   - ATR 根據市場波動動態調整門檻，高波動標的（如加密貨幣）門檻較寬，低波動標的（如藍籌股）門檻較窄，避免固定 9% 的局限。
2. **更靈活的移動停損**：
   - 利潤門檻隨每日 ATR 更新，貼近實際價格行為。
3. **通用性**：
   - 適用於不同市場和標的，不需為每個標的手動調整百分比。

---

### 潛在問題與優化建議
1. **ATR 倍數選擇**：
   - 預設 2 倍 ATR 可能偏保守或激進，需回測不同倍數（如 1.5、2.5、3）找到最佳值。
   - 倍數過低：門檻太容易達成，失去移動停損的意義；倍數過高：難以觸發 10 日均線半出。
2. **波動性影響**：
   - 在極端波動市場，ATR 可能突然放大，導致門檻過高，策略退化為只有停損和全出。
3. **建議**：
   - **回測**：測試不同 `atrLength`（如 10、14、20）和 `atrMultiplier`（如 1.5、2、3），觀察勝率和盈虧比。
   - **動態調整**：可加入條件，例如若 ATR 激增（如前日 2 倍），限制門檻變化。
   - **結合選股**：若應用於「半年漲幅 > 90%」的股票，這類標的波動通常較大，建議從較高的 `atrMultiplier`（如 2.5 或 3）開始測試。

---

### 結論
這個 ATR 版本保留了你原有的突破進場和分階段出場邏輯，同時用 ATR 替代固定 9%，讓策略更具適應性。它的效果取決於：
- 標的波動性：高波動標的（如科技股）可能更適合。
- ATR 參數：需通過回測優化 `atrLength` 和 `atrMultiplier`。

建議你在 TradingView 上對目標標的（如台股期貨或美股）進行回測，確認 ATR 版的表現。如果你有特定標的或市場想測試，告訴我，我可以幫你進一步模擬或調整！