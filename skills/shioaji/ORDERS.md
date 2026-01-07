# Orders 下單

This document covers placing, modifying, and canceling orders in Shioaji.
本文件說明如何在 Shioaji 中下單、改單和刪單。

---

## Prerequisites 前置條件

Before placing orders, ensure CA is activated:
下單前請確認已啟用憑證：

```python
import shioaji as sj

api = sj.Shioaji()
api.login(api_key="YOUR_KEY", secret_key="YOUR_SECRET")

# Activate CA 啟用憑證 (required 必須)
api.activate_ca(
    ca_path="/path/to/Sinopac.pfx",
    ca_passwd="YOUR_PASSWORD"
)
```

---

## Stock Orders 股票下單

### Basic Stock Order 基本股票下單

```python
contract = api.Contracts.Stocks["2330"]

order = api.Order(
    price=580,
    quantity=1,
    action=sj.constant.Action.Buy,
    price_type=sj.constant.StockPriceType.LMT,
    order_type=sj.constant.OrderType.ROD,
    order_lot=sj.constant.StockOrderLot.Common,
    account=api.stock_account,
)

trade = api.place_order(contract, order)
```

### Order Parameters 訂單參數

| Parameter 參數 | Type 類型 | Description 說明 |
|----------------|-----------|------------------|
| `price` | float/int | Order price 委託價格 |
| `quantity` | int | Order quantity 委託數量 |
| `action` | Action | Buy/Sell 買/賣 |
| `price_type` | PriceType | LMT/MKT/MKP 限價/市價/範圍市價 |
| `order_type` | OrderType | ROD/IOC/FOK 委託條件 |
| `order_lot` | OrderLot | Common/Odd/IntradayOdd/Fixing 交易單位 |
| `order_cond` | OrderCond | Cash/MarginTrading/ShortSelling 信用條件 |
| `account` | Account | Trading account 交易帳戶 |
| `custom_field` | str | Memo (max 6 chars) 備註（最多6字元）|

### Market Order 市價單

```python
order = api.Order(
    price=0,  # Price ignored for MKT 市價單忽略價格
    quantity=1,
    action=sj.constant.Action.Buy,
    price_type=sj.constant.StockPriceType.MKT,
    order_type=sj.constant.OrderType.IOC,  # MKT requires IOC/FOK 市價須 IOC/FOK
    account=api.stock_account,
)
```

### Odd Lot Orders 零股下單

```python
# Intraday odd lot 盤中零股 (9:00-13:30)
order = api.Order(
    price=580,
    quantity=100,  # Less than 1000 shares 小於1000股
    action=sj.constant.Action.Buy,
    price_type=sj.constant.StockPriceType.LMT,
    order_type=sj.constant.OrderType.ROD,
    order_lot=sj.constant.StockOrderLot.IntradayOdd,
    account=api.stock_account,
)

# After-hours odd lot 盤後零股 (13:40-14:30)
order = api.Order(
    price=580,
    quantity=100,
    action=sj.constant.Action.Buy,
    price_type=sj.constant.StockPriceType.LMT,
    order_type=sj.constant.OrderType.ROD,
    order_lot=sj.constant.StockOrderLot.Odd,
    account=api.stock_account,
)
```

### Margin Trading 融資融券

```python
# Margin buy 融資買進
order = api.Order(
    price=580,
    quantity=1,
    action=sj.constant.Action.Buy,
    price_type=sj.constant.StockPriceType.LMT,
    order_type=sj.constant.OrderType.ROD,
    order_cond=sj.constant.StockOrderCond.MarginTrading,
    account=api.stock_account,
)

# Short sell 融券賣出
order = api.Order(
    price=580,
    quantity=1,
    action=sj.constant.Action.Sell,
    price_type=sj.constant.StockPriceType.LMT,
    order_type=sj.constant.OrderType.ROD,
    order_cond=sj.constant.StockOrderCond.ShortSelling,
    account=api.stock_account,
)
```

### Day Trading 現股當沖

```python
# Day trade buy (first leg) 當沖買進（第一筆）
order = api.Order(
    price=580,
    quantity=1,
    action=sj.constant.Action.Buy,
    price_type=sj.constant.StockPriceType.LMT,
    order_type=sj.constant.OrderType.ROD,
    daytrade_short=True,  # Enable day trade 啟用當沖
    account=api.stock_account,
)

# Day trade sell (close position) 當沖賣出（平倉）
order = api.Order(
    price=590,
    quantity=1,
    action=sj.constant.Action.Sell,
    price_type=sj.constant.StockPriceType.LMT,
    order_type=sj.constant.OrderType.ROD,
    daytrade_short=True,
    account=api.stock_account,
)
```

---

## Futures Orders 期貨下單

### Basic Futures Order 基本期貨下單

```python
contract = api.Contracts.Futures["TXFC0"]  # Current month 近月

order = api.Order(
    price=18000,
    quantity=1,
    action=sj.constant.Action.Buy,
    price_type=sj.constant.FuturesPriceType.LMT,
    order_type=sj.constant.OrderType.ROD,
    octype=sj.constant.FuturesOCType.Auto,  # Auto open/close 自動開平
    account=api.futopt_account,
)

trade = api.place_order(contract, order)
```

### Futures Parameters 期貨參數

| Parameter 參數 | Type 類型 | Description 說明 |
|----------------|-----------|------------------|
| `price` | float/int | Order price 委託價格 |
| `quantity` | int | Number of contracts 口數 |
| `action` | Action | Buy/Sell 買/賣 |
| `price_type` | FuturesPriceType | LMT/MKT/MKP 限價/市價/範圍市價 |
| `order_type` | OrderType | ROD/IOC/FOK 委託條件 |
| `octype` | FuturesOCType | Auto/NewPosition/Cover 自動/新倉/平倉 |
| `account` | Account | Futures account 期貨帳戶 |

### Open/Close Type 開平倉類型

```python
sj.constant.FuturesOCType.Auto        # Auto 自動 (recommended 推薦)
sj.constant.FuturesOCType.NewPosition # Open new 新倉
sj.constant.FuturesOCType.Cover       # Close 平倉
sj.constant.FuturesOCType.DayTrade    # Day trade 當沖
```

---

## Options Orders 選擇權下單

### Basic Options Order 基本選擇權下單

```python
# Buy call option 買進買權
contract = api.Contracts.Options["TXO202401C18000"]

order = api.Order(
    price=100,
    quantity=1,
    action=sj.constant.Action.Buy,
    price_type=sj.constant.FuturesPriceType.LMT,
    order_type=sj.constant.OrderType.ROD,
    octype=sj.constant.FuturesOCType.Auto,
    account=api.futopt_account,
)

trade = api.place_order(contract, order)
```

### Sell Options 賣出選擇權

```python
# Sell put option 賣出賣權
contract = api.Contracts.Options["TXO202401P17000"]

order = api.Order(
    price=50,
    quantity=1,
    action=sj.constant.Action.Sell,
    price_type=sj.constant.FuturesPriceType.LMT,
    order_type=sj.constant.OrderType.ROD,
    octype=sj.constant.FuturesOCType.Auto,
    account=api.futopt_account,
)

trade = api.place_order(contract, order)
```

---

## Modify Orders 改單

### Change Price 改價

```python
api.update_order(trade=trade, price=585)
```

### Reduce Quantity 減量

Note: Can only reduce, not increase.
注意：只能減少，不能增加。

```python
api.update_order(trade=trade, qty=1)
```

---

## Cancel Orders 刪單

```python
api.cancel_order(trade)

# Or cancel by trade object 或透過 trade 物件
api.cancel_order(trade=trade)
```

---

## Order Status 訂單狀態

### Update Status 更新狀態

```python
# Update all order status 更新所有訂單狀態
api.update_status(api.stock_account)
api.update_status(api.futopt_account)
```

### List Trades 列出交易

```python
# List all trades 列出所有交易
trades = api.list_trades()

for trade in trades:
    print(f"Order: {trade.order.id}")
    print(f"Status: {trade.status.status}")
    print(f"Deal Quantity: {trade.status.deal_quantity}")
```

### Trade Status Values 交易狀態值

| Status 狀態 | Description 說明 |
|-------------|------------------|
| `PendingSubmit` | Submitting 傳送中 |
| `PreSubmitted` | Pre-submitted 預約中 |
| `Submitted` | Submitted 已送出 |
| `Failed` | Failed 失敗 |
| `Cancelled` | Cancelled 已取消 |
| `Filled` | Fully filled 全部成交 |
| `PartFilled` | Partially filled 部分成交 |

### Trade Object Attributes 交易物件屬性

```python
trade.contract        # Contract 合約
trade.order           # Order details 訂單詳情
trade.order.id        # Order ID 訂單編號
trade.order.seqno     # Sequence number 序號
trade.order.action    # Buy/Sell 買賣方向
trade.order.price     # Order price 委託價
trade.order.quantity  # Order quantity 委託量
trade.status          # Status object 狀態物件
trade.status.status   # Status value 狀態值
trade.status.deal_quantity  # Filled quantity 成交量
trade.status.cancel_quantity # Cancelled quantity 取消量
```

---

## Order Callbacks 訂單回報

### Set Trade Callback 設定交易回報

```python
@api.on_order_callback
def order_callback(stat, msg):
    print(f"Status: {stat}")
    print(f"Message: {msg}")
```

### Callback Message Fields 回報訊息欄位

```python
msg["operation"]    # Operation type 操作類型
msg["order"]        # Order info 訂單資訊
msg["status"]       # Status info 狀態資訊
msg["contract"]     # Contract info 合約資訊
```

---

## Best Practices 最佳實踐

### 1. Always Check Order Status 總是檢查訂單狀態

```python
trade = api.place_order(contract, order)
api.update_status(api.stock_account)

if trade.status.status == sj.constant.Status.Filled:
    print("Order filled!")
elif trade.status.status == sj.constant.Status.Failed:
    print(f"Order failed: {trade.status.msg}")
```

### 2. Use Callbacks for Real-time Updates 使用回報獲取即時更新

```python
@api.on_order_callback
def order_callback(stat, msg):
    if stat == sj.constant.OrderState.TFTDeal:
        print(f"Deal: {msg}")
```

### 3. Handle Errors Gracefully 優雅處理錯誤

```python
try:
    trade = api.place_order(contract, order)
except Exception as e:
    print(f"Order error: {e}")
```
