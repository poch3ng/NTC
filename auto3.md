以下為整合「排除市值前20大、最多只持有20個幣種、匯出清單與資金比例」的完整流程與程式範例：


---

1. 篩選流程與設定



從交易所 API 取得所有幣種對 USDT 的交易對。

從 CoinGecko 取得市值前 20 大幣種，將其剃除。

篩選出符合以下條件的幣種：

ATR 占價格比 > 5%。

24 小時平均交易量 > 500,000 美金。


若符合條件的幣種超過 20 個，則以 ATR 比例（atr_ratio）由高到低排序後取前 20 個。


2. 資金比例分配



使用 ATR 比例來分配資金。

資金計算公式：
Weight_i = (ATR_Ratio_i) / Σ(ATR_Ratio_all)
Fund_i = 總資金 * Weight_i


3. 匯出清單



將篩選結果（含符幣種、ATR比例、平均成交量、分配資金額度）匯出成 CSV 檔案，以供後續策略執行使用。



---

程式範例：

import ccxt
import pandas as pd
import requests

# 初始化交易所
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
            # 剃除市值前 20 大的幣種
            continue
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
        except:
            pass

    # 若超過 20 個，取 ATR 比例最高的前 20
    if len(selected_coins) > max_coins:
        selected_coins = sorted(selected_coins, key=lambda x: x['atr_ratio'], reverse=True)[:max_coins]

    return selected_coins

def allocate_funds(filtered_coins, total_fund=10000):
    total_atr_ratio = sum([coin['atr_ratio'] for coin in filtered_coins])
    for coin in filtered_coins:
        coin['fund'] = total_fund * (coin['atr_ratio'] / total_atr_ratio)
    return filtered_coins

# 主程式執行範例
markets = exchange.fetch_tickers()
symbols = [m for m in markets.keys() if '/USDT' in m]
filtered_coins = filter_coins(symbols, max_coins=20)

# 資金分配
allocated_funds = allocate_funds(filtered_coins, total_fund=10000)

# 匯出清單成 CSV 檔案 (包含 symbol, atr_ratio, avg_volume, fund)
df_filtered = pd.DataFrame(allocated_funds)
df_filtered.to_csv('filtered_coins_with_fund.csv', index=False)

print("已篩選並分配資金的幣種清單匯出至 filtered_coins_with_fund.csv:")
print(df_filtered)


---

透過上述程式流程，即可：

剃除市值前20大幣種。

根據 ATR 及交易量篩選出符合條件的幣種，最多保留 20 種。

根據 ATR 比例分配資金。

將篩選結果（包含分配資金）完整匯出成 CSV 檔供後續策略使用。


