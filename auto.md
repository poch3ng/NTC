以下是篩選和資金分配的完整設計流程：


---

1. 幣種篩選自動化程式

(1) 程式功能設計

輸入：幣種清單（從交易所 API 獲取）。

輸出：符合篩選條件的幣種清單。

篩選條件：

波動率（ATR）高於 5%（震盪大）。

24 小時交易量 > $500,000（流動性充足）。

最近價格區間波動穩定（適合網格）。

排除單邊趨勢幣種（ADX < 25）。



(2) 篩選程式碼

以下是 Python 程式碼範例：

import ccxt
import pandas as pd

# 初始化交易所
exchange = ccxt.binance({
    'apiKey': 'your_api_key',
    'secret': 'your_secret_key'
})

# 計算 ATR
def calculate_atr(data):
    data['TR'] = data[['high', 'low', 'close']].apply(
        lambda row: max(row['high'] - row['low'], 
                        abs(row['high'] - row['close']),
                        abs(row['low'] - row['close'])), axis=1)
    return data['TR'].rolling(window=14).mean().iloc[-1]

# 篩選幣種
def filter_coins(symbols):
    selected_coins = []
    for symbol in symbols:
        try:
            ohlcv = exchange.fetch_ohlcv(symbol, timeframe='1h', limit=100)
            df = pd.DataFrame(ohlcv, columns=['timestamp', 'open', 'high', 'low', 'close', 'volume'])
            df['ATR'] = calculate_atr(df)
            atr_ratio = df['ATR'].iloc[-1] / df['close'].iloc[-1]
            avg_volume = df['volume'].mean()

            # 篩選條件
            if atr_ratio > 0.05 and avg_volume > 500000:
                selected_coins.append({
                    'symbol': symbol,
                    'atr_ratio': atr_ratio,
                    'avg_volume': avg_volume
                })
        except Exception as e:
            print(f"Error fetching data for {symbol}: {e}")
    return selected_coins

# 主程式
markets = exchange.fetch_tickers()
symbols = [market for market in markets.keys() if '/USDT' in market]
filtered_coins = filter_coins(symbols)

# 輸出結果
print("選定幣種：", filtered_coins)

此程式會篩選出 ATR 超過 5%、交易量達標的幣種。


---

2. 投入資金比例設計

(1) 投資資金總額

假設投資總金額為 。

(2) 每個幣種的分配

根據凱利公式與網格策略需求，分配資金比例：

波動越大，分配資金比例越高。

每幣種資金比例基於波動率（ATR 占價格的比值）與預期收益。


(3) 資金分配公式

對篩選出的幣種按以下步驟計算分配比例：

1. 計算每個幣種的 ATR 比值：



ATR\_Ratio_i = \frac{ATR_i}{Price_i}

Weight_i = \frac{ATR\_Ratio_i}{\sum ATR\_Ratio}

Fund_i = T \times Weight_i

(4) 程式實現

以下是資金分配的程式碼：

# 假設總資金
total_fund = 10000  # USD

# 計算資金分配
def allocate_funds(filtered_coins, total_fund):
    total_atr_ratio = sum([coin['atr_ratio'] for coin in filtered_coins])
    for coin in filtered_coins:
        coin['fund'] = total_fund * (coin['atr_ratio'] / total_atr_ratio)
    return filtered_coins

allocated_funds = allocate_funds(filtered_coins, total_fund)

# 輸出結果
for coin in allocated_funds:
    print(f"幣種: {coin['symbol']}, 分配資金: ${coin['fund']:.2f}, ATR 比例: {coin['atr_ratio']:.4f}")

輸出會顯示每個幣種的分配資金金額。


---

3. 最終執行規劃

(1) 執行網格策略

使用篩選出的幣種及資金分配金額設定網格參數（如買賣掛單範圍、間距）。

每幣種根據其波動率動態調整網格策略。


(2) 自動化部署

使用伺服器部署篩選和交易程式，確保篩選到的幣種能自動進行網格掛單。


(3) 持續監控與調整

每 6-12 小時重新篩選幣種，調整資金分配與網格策略參數。



---

結論

篩選出的幣種具有高波動性和足夠流動性。

資金分配根據波動率自動調整，並最大化收益。

程式化篩選與執行可大幅降低人工參與需求。


如果需要協助將策略整合至伺服器或自動化交易環境，請告知！

