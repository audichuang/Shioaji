# Market Data 市場資料

This document covers credit enquiries, short stock sources, and scanners.
本文件說明資券餘額查詢、券源查詢和掃描器排行。

---

## Credit Enquiries 資券餘額查詢

Query margin and short unit information for stocks.
查詢股票的融資融券餘額資訊。

```python
contracts = [
    api.Contracts.Stocks["2330"],
    api.Contracts.Stocks["2890"],
]

credit_enquires = api.credit_enquires(contracts)
```

### CreditEnquire Attributes 資券餘額屬性

```python
enquire.update_time   # str: Update time 更新時間
enquire.system        # str: System code 系統代碼
enquire.stock_id      # str: Stock code 股票代碼
enquire.margin_unit   # int: Margin units 融資餘額
enquire.short_unit    # int: Short units 融券餘額
```

### Convert to Polars 轉換為 Polars

```python
import polars as pl

df = pl.DataFrame([
    {
        "stock_id": c.stock_id,
        "system": c.system,
        "margin_unit": c.margin_unit,
        "short_unit": c.short_unit,
        "update_time": c.update_time,
    }
    for c in credit_enquires
])
```

---

## Short Stock Sources 券源查詢

Query available short stock sources (借券來源).
查詢可借券數量。

```python
contracts = [
    api.Contracts.Stocks["2330"],
    api.Contracts.Stocks["2317"],
]

short_sources = api.short_stock_sources(contracts)
```

### ShortStockSource Attributes 券源屬性

```python
source.code               # str: Stock code 股票代碼
source.short_stock_source # int: Available shares 可借券數量
source.ts                 # int: Timestamp 時間戳
```

### Convert to Polars 轉換為 Polars

```python
import polars as pl

df = pl.DataFrame([
    {
        "code": s.code,
        "short_stock_source": s.short_stock_source,
        "ts": s.ts,
    }
    for s in short_sources
]).with_columns(
    pl.col("ts").cast(pl.Datetime("ns"))
)
```

---

## Scanners 掃描器排行

Get market rankings by various criteria.
依各種條件取得市場排行。

### Scanner Types 掃描器類型

```python
import shioaji as sj

sj.constant.ScannerType.ChangePercentRank  # 漲跌幅排行
sj.constant.ScannerType.ChangePriceRank    # 漲跌價排行
sj.constant.ScannerType.DayRangeRank       # 振幅排行
sj.constant.ScannerType.VolumeRank         # 成交量排行
sj.constant.ScannerType.AmountRank         # 成交金額排行
```

### Query Scanners 查詢排行

```python
# Top 10 gainers 漲幅前 10 名
scanners = api.scanners(
    scanner_type=sj.constant.ScannerType.ChangePercentRank,
    ascending=False,  # False = descending 由大到小
    count=10,
)

# Top 10 losers 跌幅前 10 名
scanners = api.scanners(
    scanner_type=sj.constant.ScannerType.ChangePercentRank,
    ascending=True,  # True = ascending 由小到大
    count=10,
)

# Top volume 成交量排行
scanners = api.scanners(
    scanner_type=sj.constant.ScannerType.VolumeRank,
    ascending=False,
    count=10,
)
```

### Scanner Attributes 掃描器屬性

```python
scan.date            # str: Trade date 交易日
scan.code            # str: Stock code 股票代碼
scan.name            # str: Stock name 股票名稱
scan.ts              # int: Timestamp 時間戳
scan.open            # float: Open price 開盤價
scan.high            # float: High price 最高價
scan.low             # float: Low price 最低價
scan.close           # float: Close price 收盤價
scan.price_range     # float: Day range 振幅
scan.change_price    # float: Price change 漲跌價
scan.change_type     # int: Change type 漲跌類型
scan.average_price   # float: Average price 均價
scan.volume          # int: Last volume 最後成交量
scan.total_volume    # int: Total volume 總成交量
scan.amount          # int: Last amount 最後成交金額
scan.total_amount    # int: Total amount 總成交金額
scan.yesterday_volume # int: Yesterday volume 昨日成交量
scan.volume_ratio    # float: Volume ratio 量比
scan.buy_price       # float: Bid price 買價
scan.buy_volume      # int: Bid volume 買量
scan.sell_price      # float: Ask price 賣價
scan.sell_volume     # int: Ask volume 賣量
scan.bid_orders      # int: Bid side orders 內盤成交單數
scan.bid_volumes     # int: Bid side volume 內盤成交量
scan.ask_orders      # int: Ask side orders 外盤成交單數
scan.ask_volumes     # int: Ask side volume 外盤成交量
scan.tick_type       # int: 1=外盤, 2=內盤, 0=無法判定
```

### Convert to Polars 轉換為 Polars

```python
import polars as pl

scanners = api.scanners(
    scanner_type=sj.constant.ScannerType.ChangePercentRank,
    count=50,
)

df = pl.DataFrame([
    {
        "code": s.code,
        "name": s.name,
        "close": s.close,
        "change_price": s.change_price,
        "total_volume": s.total_volume,
        "volume_ratio": s.volume_ratio,
    }
    for s in scanners
])

# Filter high volume ratio 篩選量比高的
high_volume = df.filter(pl.col("volume_ratio") > 2)
```

---

## Disposition & Attention Stocks 處置及注意股

Query disposition and attention stocks list.
查詢處置股及注意股清單。

For detailed information, see the official documentation:
詳細資訊請參考官方文檔：

https://sinotrade.github.io/tutor/market_data/disposition_attention/

---

## Reference 參考資料

- Credit enquiries 資券餘額: https://sinotrade.github.io/tutor/market_data/credit_enquires/
- Short stock sources 券源: https://sinotrade.github.io/tutor/market_data/short_stock_source/
- Scanners 掃描器: https://sinotrade.github.io/tutor/market_data/scanners/
