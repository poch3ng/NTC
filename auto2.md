以下為加入「最多只持有 20 個幣種」的完整流程設計：


---

1. 幣種篩選自動化程式（包含排除市值前 20 大及限縮最多 20 種）



(1) 篩選流程

從交易所 API 獲取幣種清單（以 /USDT 為例）。

取得市值前 20 大幣種清單，排除之。

篩選出符合 ATR、交易量等條件的幣種。


(2) 過濾後若幣種超過 20 個，根據 ATR 比例（atr_ratio）從高到低排序，取前 20 名。

(3) 程式碼範例

import ccxt
import pandas as pd
import requests

exchange = ccxt.binance({
    'apiKey': 'your_api_key',
    'secret': 'your_secret_key'
})

def get_top_20_marketcap_coins():
    url = "https://api.coingecko.com/api/v3/coins/markets"
    params = {
        'vs_currency': 'usd',
        'order': 'market_cap_desc',
        'per_page': 20,
        'page': 1
    }
    response = requests.get(url, params=params)
    data = response.json()
    top_20 = [item['symbol'].upper() for item in data]
    return top_20

def calculate_atr(data):
    data['TR'] = data[['high', 'low', 'close']].apply(
        lambda row: max(
            row['high'] - row['low'],
            abs(row['high'] - row['close']),
            abs(row['low'] - row['close'])
        ), axis=1)
    return data['TR'].rolling(window=14).mean().iloc[-1]

def filter_coins(symbols, max_coins=20):
    top_20_coins = get_top_20_marketcap_coins()
    selected_coins = []
    for symbol in symbols:
        base = symbol.split('/')[0]
        if base in top_20_coins:
            continue
        try:
            ohlcv = exchange.fetch_ohlcv(symbol, timeframe='1h', limit=100)
            df = pd.DataFrame(ohlcv, columns=['timestamp', 'open', 'high', 'low', 'close', 'volume'])
            df['ATR'] = calculate_atr(df)
            atr_ratio = df['ATR'].iloc[-1] / df['close'].iloc[-1]
            avg_volume = df['volume'].mean()

            if atr_ratio > 0.05 and avg_volume > 500000:
                selected_coins.append({
                    'symbol': symbol,
                    'atr_ratio': atr_ratio,
                    'avg_volume': avg_volume
                })
        except:
            pass

    # 若超過 20 個幣種，根據 ATR 比例排序後取前 20 個
    if len(selected_coins) > max_coins:
        selected_coins = sorted(selected_coins, key=lambda x: x['atr_ratio'], reverse=True)[:max_coins]

    return selected_coins

# 主程式執行範例
markets = exchange.fetch_tickers()
symbols = [market for market in markets.keys() if '/USDT' in market]
filtered_coins = filter_coins(symbols, max_coins=20)
print("選定幣種：", filtered_coins)


---

2. 資金比例分配



(1) 根據 ATR 比例計算權重分配資金。

total_fund = 10000  # USD

def allocate_funds(filtered_coins, total_fund):
    total_atr_ratio = sum([coin['atr_ratio'] for coin in filtered_coins])
    for coin in filtered_coins:
        coin['fund'] = total_fund * (coin['atr_ratio'] / total_atr_ratio)
    return filtered_coins

allocated_funds = allocate_funds(filtered_coins, total_fund)

for coin in allocated_funds:
    print(f"幣種: {coin['symbol']}, 分配資金: ${coin['fund']:.2f}, ATR比例: {coin['atr_ratio']:.4f}")


---

3. 最終執行規劃



完成篩選後，針對入選的最多 20 種幣種進行網格策略掛單。

定期（6-12 小時）重新篩選，更新資金配置。

部署於伺服器自動執行。


透過本流程，可達成對市值前 20 大幣種剃除後，再從高波動、高流動性的中小市值幣種中挑選出不超過 20 種進行投資配置。

