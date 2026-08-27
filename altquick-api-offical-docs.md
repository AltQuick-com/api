If you want to trade on our exchange accountless using market orders, please see our AltQuick.com/swap API: https://altquick.com/swap/api.html

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [Public Rest API for AltQuick](#public-rest-api-for-altquick)
  - [General API Information](#general-api-information)
  - [HTTP Return Codes](#http-return-codes)
  - [Error Codes](#error-codes)
  - [General Information on Endpoints](#general-information-on-endpoints)
- [Endpoint security type](#endpoint-security-type)
- [SIGNED (TRADE and USER_DATA) Endpoint security](#signed-trade-and-user_data-endpoint-security)
  - [SIGNED Endpoint Examples for POST /api/v1/order](#signed-endpoint-examples-for-post-apiv1order)
    - [Example: Query string request](#example-query-string-request)
- [Public API Endpoints](#public-api-endpoints)
  - [Terminology](#terminology)
  - [General endpoints](#general-endpoints)
    - [Test connectivity](#test-connectivity)
    - [Check server time](#check-server-time)
    - [Exchange information](#exchange-information)
    - [System status](#system-status)
    - [Asset status](#asset-status)
    - [Bitcoin miner fee](#bitcoin-miner-fee)
  - [Market Data endpoints](#market-data-endpoints)
    - [Order book](#order-book)
    - [Recent trades list](#recent-trades-list)
    - [Kline/Candlestick data](#klinecandlestick-data)
    - [24hr ticker price change statistics](#24hr-ticker-price-change-statistics)
    - [Market price ticker](#market-price-ticker)
    - [Market order book ticker](#market-order-book-ticker)
  - [Account endpoints](#account-endpoints)
    - [New order  (TRADE)](#new-order--trade)
    - [Query order (USER_DATA)](#query-order-user_data)
    - [Cancel order (TRADE)](#cancel-order-trade)
    - [Current open orders (USER_DATA)](#current-open-orders-user_data)
    - [All orders (USER_DATA)](#all-orders-user_data)
    - [Account information (USER_DATA)](#account-information-user_data)
    - [Validate address (USER_DATA)](#validate-address-user_data)
    - [Withdraw (USER_DATA)](#withdraw-user_data)
    - [Query withdraw (USER_DATA)](#query-withdraw-user_data)
    - [Account trade list (USER_DATA)](#account-trade-list-user_data)
    - [Deposit history (USER_DATA)](#deposit-history-user_data)
    - [Withdraw history (USER_DATA)](#withdraw-history-user_data)
    - [Deposit address (USER_DATA)](#deposit-address-user_data)
    - [New deposit address (USER_DATA)](#new-deposit-address-user_data)
  - [Api v2](#api-v2)
     - [Markets](#markets)
     - [Ticker](#ticker)
     - [Assets](#assets)
     - [Orderbook](#orderbook)
     - [Trades](#trades)
  - [Legacy public endpoints](#legacy-public-endpoints)
- [Filters](#filters)
  - [Market filters](#market-filters)
    - [PRICE_FILTER](#price_filter)
    - [LOT_SIZE](#lot_size)
    - [MARKET_LOT_SIZE](#market_lot_size)
    - [MARKET_QUANTITY_INPUT](#market_quantity_input)
    - [MAX_NUM_ORDERS](#max_num_orders)
    - [MIN_ORDER_SIZE](#min_order_size)
  - [Exchange Filters](#exchange-filters)
    - [EXCHANGE_MAX_NUM_ORDERS](#exchange_max_num_orders)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# Public Rest API for AltQuick

## General API Information
* The base endpoint is: **https://altquick.com**
* All documented `/api/v1` and `/api/v2` endpoints return JSON unless noted.
* Time and timestamp fields are in **milliseconds**, except:
  * `/api/v2/orderbook` `timestamp` is unix **seconds**
  * `/api/v1/allOrders` `startTime` / `endTime` are unix **seconds**
* Market symbols are `QUOTE_BASE`, for example `BTC_CLAM` (price in BTC, quantity in CLAM).
* Public v1 market-data routes send `Access-Control-Allow-Origin: *`.

## HTTP Return Codes

* HTTP `200` is used for successful responses.
* HTTP `409` is used for most `/api/v1` application errors (invalid market, bad signature, missing API key, etc.).
* HTTP `404` is used by some v2 / legacy routes when the market is unknown (plain text body).
* HTTP `429` is reserved for request-rate limits on some account flows.
* HTTP `5XX` return codes are used for internal errors.

## Error Codes
* Any endpoint can return an error.

Typical `/api/v1` payload:

```javascript
{
  "code": -1104,
  "message": "Invalid market"
}
```

Some signed-account failures return:

```javascript
{
  "errors": {
    "msg": "INVALID_API_KEY"
  }
}
```

Specific codes are defined in [errors.md](./errors.md).

## General Information on Endpoints
* For `GET`, `POST`, and `DELETE` `/api/v1` endpoints, parameters are sent as a **query string**. They are not read from a JSON body.
* Parameters may be sent in any order, except `signature` must be the **last** query parameter on signed requests.

# Endpoint security type
* Each endpoint has a security type, stated next to the name of the endpoint.
    * If no security type is stated, assume `NONE`.
* API keys are passed via the `X-MBX-APIKEY` header.
* API keys and secret keys **are case sensitive**.

Security Type | Description
------------ | ------------
NONE | Endpoint can be accessed freely.
TRADE | Endpoint requires a valid API key and HMAC signature.
USER_DATA | Endpoint requires a valid API key and HMAC signature.

* `TRADE` and `USER_DATA` endpoints are `SIGNED` endpoints.

# SIGNED (TRADE and USER_DATA) Endpoint security
* `SIGNED` endpoints require an additional parameter, `signature`, as the **last** query-string parameter.
* Signatures use `HMAC SHA256`. Use your `secretKey` as the key and `totalParams` as the value.
* `totalParams` is the raw query string **with `&signature=...` removed**.
* The hex `signature` is **not case sensitive**.

Example:

```
query string: market=BTC_CLAM&side=buy&type=limit&quantity=1&price=0.1&timestamp=1499827319559&signature=...
signed payload: market=BTC_CLAM&side=buy&type=limit&quantity=1&price=0.1&timestamp=1499827319559
```

`timestamp` and `recvWindow` may be included (and must be covered by the signature if sent). **They are not currently validated by the server.** Requests are accepted based on API key + signature only.

## SIGNED Endpoint Examples for POST /api/v1/order
Linux command-line example using `echo`, `openssl`, and `curl`.

Key | Value
------------ | ------------
apiKey | vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A
secretKey | NhqPtmdSJYdKjVHjA7PZj4Mge3R5YNiP1e3UZjInClVN65XAbvqqM6A7H5fATj0j


Parameter | Value
------------ | ------------
market | BTC_CLAM
side | buy
type | limit
quantity | 1
price | 0.1
timestamp | 1499827319559

### Example: Query string request
* **queryString:** market=BTC_CLAM&side=buy&type=limit&quantity=1&price=0.1&timestamp=1499827319559
* **HMAC SHA256 signature:**

    ```
    [linux]$ echo -n "market=BTC_CLAM&side=buy&type=limit&quantity=1&price=0.1&timestamp=1499827319559" | openssl dgst -sha256 -hmac "NhqPtmdSJYdKjVHjA7PZj4Mge3R5YNiP1e3UZjInClVN65XAbvqqM6A7H5fATj0j"
    (stdin)= 87054d1502cb16cce8d709e3ee47617bc23aa2869750ab1e1c6ec5c1908a09ef
    ```

* **curl command:**

    ```
    [linux]$ curl -H "X-MBX-APIKEY: vmPUZE6mv9SD5VNHk4HlWFsOr6aKE2zvsw0MuIgwCIPy6utIco14y7Ju91duEh8A" -X POST 'https://altquick.com/api/v1/order?market=BTC_CLAM&side=buy&type=limit&quantity=1&price=0.1&timestamp=1499827319559&signature=87054d1502cb16cce8d709e3ee47617bc23aa2869750ab1e1c6ec5c1908a09ef'
    ```

# Public API Endpoints
## Terminology
* `base` is the asset you are buying or selling (quantity).
* `quote` is the asset used to price the market. On AltQuick this is BTC for listed pairs.

**Order status (`order_state`):**

* new
* booked
* partial
* filled
* cancelled
* rejected

**Order types (`orderTypes`, `type`):**

* limit
* market

**Order side (`side`):**

* buy
* sell

**Kline/Candlestick chart intervals:**

m -> minutes; h -> hours; d -> days; M -> months

* 1m
* 3m
* 5m
* 15m
* 30m
* 1h
* 2h
* 4h
* 6h
* 8h
* 12h
* 1d
* 3d
* 7d
* 1M

## General endpoints
### Test connectivity
```
GET /api/v1/ping
```
Test connectivity to the Rest API.

**Parameters:**
NONE

**Response:**
HTTP 200 with an empty body (`Content-Length: 0`).

### Check server time
```
GET /api/v1/time
```
Current server time.

**Parameters:**
NONE

**Response:**
```javascript
{
  "timezone": "UTC",
  "serverTime": 1787851810564
}
```

### Exchange information
```
GET /api/v1/exchangeInfo
```
Current exchange trading rules and market information.

**Parameters:**
NONE

**Response:**
```javascript
{
  "timezone": "UTC",
  "serverTime": 1787851810965,
  "exchangeFilters": [
    {
      "filterType": "MAX_NUM_ORDERS",
      "maxNumOrders": 100
    },
    {
      "filterType": "EXCHANGE_MAX_NUM_ORDERS",
      "maxNumOrdersExchange": 1000
    }
  ],
  "markets": [
    {
      "id": 1,
      "symbol": "BTC_CLAM",
      "quote": "BTC",
      "base": "CLAM",
      "orderTypes": [
        "limit",
        "market"
      ],
      "filters": [
        {
          "filterType": "PRICE_FILTER",
          "minPrice": "0.00000010",
          "maxPrice": "10000.00000000",
          "tickSize": "0.00000100"
        },
        {
          "filterType": "LOT_SIZE",
          "minQty": "0.01000000",
          "maxQty": "100000.00000000",
          "stepSize": "0.01000000"
        },
        {
          "filterType": "MARKET_LOT_SIZE",
          "minQty": "0.00000000",
          "maxQty": "63100.00000000",
          "stepSize": "0.00000000"
        },
        {
          "filterType": "MARKET_QUANTITY_INPUT",
          "buy": "quote",
          "sell": "base"
        }
      ]
    }
  ]
}
```

### System status
```
GET /api/v1/systemStatus
```
Per-asset deposit, withdraw, and trading status, plus fees and chain tip.

**Parameters:**
NONE

**Response:**
```javascript
{
  "lastUpdate": "Thu, 27 Aug 2026 17:30:10 UTC",
  "currentTime": "Thu, 27 Aug 2026 17:30:10 UTC",
  "assets": [
    {
      "asset": "BTC",
      "assetType": "coin",
      "deposits": true,
      "depositFee": "0.00000000",
      "withdrawals": true,
      "trading": true,
      "pendingDeposits": 0,
      "pendingWithdrawals": 0,
      "lastHeight": 868797,
      "lastHash": "cf9e7b24d70e011482b07e0321dfde8b4eb3875290fb839ee120d988e9b41ed0",
      "maint": "",
      "withdrawFee": "0.00050000",
      "tradeFee": "1%"
    }
  ]
}
```

### Asset status
```
GET /api/v1/status/{asset}
```
Deposit and withdraw flags for one asset.

**Parameters:** path `asset` is required, for example `BTC`.

**Response:**
```javascript
{
  "asset": "BTC",
  "depositsActive": true,
  "withdrawalsActive": true
}
```

### Bitcoin miner fee
```
GET /api/v1/btcfee
```
Current BTC withdraw fee rate used by the exchange, in sats per vbyte.

**Parameters:**
NONE

**Response:**
```javascript
{
  "satsPerVbyte": 4
}
```

## Market Data endpoints
### Order book
```
GET /api/v1/depth
```

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
market | STRING | YES |
limit | INT | NO | Default 100. Valid values: `5`, `10`, `20`, `50`, `100`, `500`, `1000`, `5000`. Other values fall back to 100.

The server currently returns up to `limit - 1` price levels per side.

**Response:**
```javascript
{
  "bids": [
    {
      "price": "0.00000106",
      "quantity": "4.96874451"
    }
  ],
  "asks": [
    {
      "price": "0.0000011",
      "quantity": "1.23636364"
    }
  ],
  "sequence": 397825
}
```

### Recent trades list
```
GET /api/v1/trades
```
Recent public trades (taker side).

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
market | STRING | YES |
limit | INT | NO | Default 500; max 1000.

**Response:**
```javascript
[
  {
    "id": 40298,
    "market": "BTC_CLAM",
    "maker": false,
    "side": "sell",
    "quantity": "0.17",
    "price": "0.00000106",
    "timestamp": 1787829474547
  }
]
```

### Kline/Candlestick data
```
GET /api/v1/klines
```
Kline/candlestick bars for a market, identified by open time.

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
market | STRING | YES |
interval | ENUM | YES | See interval list above.
startTime | LONG | NO | Open time in milliseconds.
endTime | LONG | NO | Close time in milliseconds.
limit | INT | NO | Default 2880; max 2880.

* If `startTime` and `endTime` are omitted, the most recent klines are returned.
* The currently forming candle may be included, so the array can be one bar longer than `limit`.

**Response:**
```javascript
[
  [
    1787842800000,   // Open time
    "0.00000106",    // Open
    "0.00000106",    // High
    "0.00000106",    // Low
    "0.00000106",    // Close
    "0",             // Volume (base)
    1787846400000,   // Close time
    0                // Number of trades
  ]
]
```

### 24hr ticker price change statistics
```
GET /api/v1/ticker/24hr
```
24 hour rolling window price change statistics.

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
market | STRING | NO |

* If `market` is omitted, tickers for all markets are returned as an array.

**Response:**
```javascript
{
  "market": "BTC_CLAM",
  "priceChange": "0",
  "priceChangePercent": "0",
  "weightedAvgPrice": "0.0000011",
  "prevClosePrice": "0.00000106",
  "lastPrice": "0.00000106",
  "lastQty": "0.17",
  "bidPrice": "0.00000106",
  "askPrice": "0.0000011",
  "openPrice": "0.00000106",
  "highPrice": "0.00000111",
  "lowPrice": "0.00000106",
  "volume": "2681.22604431",
  "quoteVolume": "0.00296892",
  "weeklyVolume": "3056.40204424",
  "weeklyQuoteVolume": "0.003377",
  "count": 7,
  "openTime": "2026-08-27T00:00:01Z",
  "closeTime": "2026-08-28T00:00:01Z"
}
```

Responses currently also include unused GORM fields (`ID`, `CreatedAt`, `UpdatedAt`, `DeletedAt`). Do not depend on those.

### Market price ticker
```
GET /api/v1/ticker/price
```
Latest price for a market or markets.

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
market | STRING | NO |

* If `market` is omitted, prices for all markets are returned as an array.

**Response:**
```javascript
{
  "market": "BTC_CLAM",
  "price": "0.00000106"
}
```

### Market order book ticker
```
GET /api/v1/ticker/bookTicker
```
Best price/qty on the order book for a market or markets.

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
market | STRING | NO |

* If `market` is omitted, book tickers for all markets are returned as an array.

**Response:**
```javascript
{
  "market": "BTC_CLAM",
  "bidPrice": "0.00000106",
  "bidQuantity": "4.96874451",
  "askPrice": "0.0000011",
  "askQuantity": "1.23636364"
}
```

## Account endpoints
All account endpoints below require `X-MBX-APIKEY` and `signature`, except as noted.

`orderID` is a **UUID string**, not an integer. `orderID` and `orderId` are both accepted. There is no `origClientOrderID`.

### New order  (TRADE)
```
POST /api/v1/order  (HMAC SHA256)
```
Also available as `POST /api/v1/openOrder`.

Send in a new order.

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
market | STRING | YES |
side | ENUM | YES | `buy` or `sell`
type | ENUM | YES | `limit` or `market`
quantity | DECIMAL | YES | For market **buy**, quantity is in the **quote** asset (BTC). For market **sell** and all limit orders, quantity is in the **base** asset.
price | DECIMAL | NO | Required for `limit`.
timestamp | LONG | NO | Included in the signature if sent. Not currently validated.
recvWindow | LONG | NO | Included in the signature if sent. Not currently validated.

Type | Additional mandatory parameters
------------ | ------------
`limit` | `quantity`, `price`
`market` | `quantity`

Limit orders below `0.00000010` BTC notional are rejected with `-9000`.
A user may have at most 100 open orders on a market (`-1005`).

**Response:**
```javascript
{
  "order_id": "72253ef2-abf1-4078-8375-c6ee690ec406",
  "user_id": "ca41eaeb-21af-4918-9131-91d19ba2a87a",
  "price": "0.00000100",
  "quantity": "2.00000000",
  "market": "BTC_CLAM",
  "remaining": "2.00000000",
  "side": "sell",
  "order_type": "limit",
  "order_state": "booked",
  "created_at": "2019-11-21T18:25:38.780Z"
}
```

Market orders that take liquidity also include a `trades` array:

```javascript
{
  "order_id": "72253ef2-abf1-4078-8375-c6ee690ec406",
  "user_id": "ca41eaeb-21af-4918-9131-91d19ba2a87a",
  "price": "0.00011000",
  "quantity": "2.00000000",
  "market": "BTC_CLAM",
  "side": "sell",
  "order_type": "market",
  "order_state": "filled",
  "created_at": "2019-11-21T18:25:38.780Z",
  "trades": [
    {
      "order_id": "72253ef2-abf1-4078-8375-c6ee690ec406",
      "user_id": "ca41eaeb-21af-4918-9131-91d19ba2a87a",
      "trade_id": 9086,
      "market": "BTC_CLAM",
      "side": "sell",
      "price": "0.00011000",
      "quantity": "0.91000000",
      "maker": false,
      "total": "0.00010010",
      "commission": "0.00000051",
      "commission_asset": "BTC",
      "date": "2019-11-21T18:25:38.780Z"
    }
  ]
}
```

### Query order (USER_DATA)
```
GET /api/v1/order (HMAC SHA256)
```
Check an order's status.

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
market | STRING | YES |
orderID | STRING | YES | UUID. `orderId` is also accepted.
timestamp | LONG | NO |
recvWindow | LONG | NO |

**Response:** stored order object. In addition to the fields on open/cancel responses, this includes `reserved`, `reservedNow`, `commission_pct`, and `reject_reason`.

```javascript
{
  "order_id": "72253ef2-abf1-4078-8375-c6ee690ec406",
  "user_id": "ca41eaeb-21af-4918-9131-91d19ba2a87a",
  "price": "0.00000100",
  "quantity": "2.00000000",
  "remaining": "2.00000000",
  "market": "BTC_CLAM",
  "side": "sell",
  "order_type": "market",
  "order_state": "filled"
}
```

### Cancel order (TRADE)
```
DELETE /api/v1/order  (HMAC SHA256)
```
Cancel an active order.

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
orderID | STRING | YES | UUID. `orderId` is also accepted.
market | STRING | NO | Not required by the server.
timestamp | LONG | NO |
recvWindow | LONG | NO |

**Response:**
```javascript
{
  "order_id": "72253ef2-abf1-4078-8375-c6ee690ec406",
  "user_id": "ca41eaeb-21af-4918-9131-91d19ba2a87a",
  "price": "0.00110000",
  "quantity": "2.00000000",
  "market": "BTC_CLAM",
  "remaining": "2.00000000",
  "side": "sell",
  "order_type": "limit",
  "order_state": "cancelled",
  "created_at": "2019-11-08T19:44:06.981Z"
}
```

### Current open orders (USER_DATA)
```
GET /api/v1/openOrders  (HMAC SHA256)
```
Get all open orders on a market.

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
market | STRING | YES |
timestamp | LONG | NO |
recvWindow | LONG | NO |

**Response:**
```javascript
[
  {
    "order_id": "72253ef2-abf1-4078-8375-c6ee690ec406",
    "user_id": "ca41eaeb-21af-4918-9131-91d19ba2a87a",
    "price": "0.00110000",
    "quantity": "10.00000000",
    "market": "BTC_ETH",
    "side": "buy",
    "remaining": "10.00000000",
    "order_type": "limit",
    "order_state": "booked",
    "created_at": "2019-11-08T19:44:06.981Z"
  }
]
```

### All orders (USER_DATA)
```
GET /api/v1/allOrders (HMAC SHA256)
```
Get account orders: active, canceled, or filled.

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
market | STRING | YES |
startTime | LONG | NO | Unix **seconds**. If omitted while `endTime` is set, defaults to now minus 24 hours.
endTime | LONG | NO | Unix **seconds**. If omitted while `startTime` is set, defaults to now.
limit | INT | NO | Default 500; max 1000.
timestamp | LONG | NO |
recvWindow | LONG | NO |

**Response:** same object shape as open orders, as an array.

### Account information (USER_DATA)
```
GET /api/v1/account (HMAC SHA256)
```
Current account balances and permissions.

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
timestamp | LONG | NO |
recvWindow | LONG | NO |

**Response:**
```javascript
{
  "canTrade": true,
  "canWithdraw": true,
  "balances": [
    {
      "asset": "BTC",
      "free": "999.80160609",
      "pending": "0.00000000",
      "locked": "0.04140000"
    },
    {
      "asset": "CLAM",
      "free": "998.88888890",
      "pending": "0.00000000",
      "locked": "0.00000000"
    }
  ]
}
```

### Validate address (USER_DATA)
```
POST /api/v1/validateaddress
```
Check whether an address is valid for an asset. Requires `X-MBX-APIKEY`. Signature is not required.

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
asset | STRING | YES |
address | STRING | YES |

**Response:**
```javascript
{
  "ok": true
}
```

### Withdraw (USER_DATA)
```
POST /api/v1/withdraw  (HMAC SHA256)
```
Request a withdraw.

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
asset | STRING | YES |
destination | STRING | YES |
quantity | DECIMAL | YES | Amount in whole coins, not satoshis.
fee | DECIMAL | NO | BTC only: optional sats/vbyte override.
timestamp | LONG | NO |
recvWindow | LONG | NO |

**Response:** a single withdraw object. `amount` and `fee` are integers in the asset's base units (satoshis / 1e8).

```javascript
{
  "user_id": "ca41eaeb-21af-4918-9131-91d19ba2a87a",
  "symbol": "CLAM",
  "destination": "xX3gahy5kjozWwaPKViDDiF4vkGfUhMiyU",
  "amount": 1000000000,
  "txid": "",
  "state": "pending",
  "fee": 100000,
  "spentFee": 0,
  "isStealth": false,
  "subtractFeeFromAmt": false,
  "pending": true,
  "complete": false,
  "retrying": false,
  "failed": false,
  "cancelled": false,
  "retry_count": 0,
  "rejected": false,
  "reject_reason": ""
}
```

### Query withdraw (USER_DATA)
```
GET /api/v1/withdraw  (HMAC SHA256)
```
Look up one withdraw by numeric id.

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
id | LONG | YES | Numeric withdraw id.
timestamp | LONG | NO |
recvWindow | LONG | NO |

**Response:**
```javascript
{
  "withdraw": {
    "id": 28457,
    "user_id": "ca41eaeb-21af-4918-9131-91d19ba2a87a",
    "symbol": "CLAM",
    "destination": "xX3gahy5kjozWwaPKViDDiF4vkGfUhMiyU",
    "amount": "10.00000000",
    "txid": "e3748cbc9bd38edbd1a63d6c2f3f6044e9294bbdebaab2a2d416e1a78abf1030",
    "state": "complete",
    "fee": "0.00100000",
    "complete": true,
    "isStealth": false,
    "failed": false,
    "cancelled": false,
    "rejected": false,
    "reject_reason": "",
    "updated_at": "2019-11-21T18:25:38Z"
  }
}
```

### Account trade list (USER_DATA)
```
GET /api/v1/myTrades  (HMAC SHA256)
```
Get trades for a specific account and market.

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
market | STRING | YES |
startTime | LONG | NO | Unix **seconds** when used with `endTime`.
endTime | LONG | NO | Unix **seconds**. If only one of start/end is sent, the other defaults to now / now minus 24 hours.
limit | INT | NO | Default 500; max 1000.
timestamp | LONG | NO |
recvWindow | LONG | NO |

**Response:**
```javascript
[
  {
    "order_id": "9574c6c5-cd3d-41d3-88b5-abfa836ac111",
    "user_id": "ca41eaeb-21af-4918-9131-91d19ba2a87a",
    "trade_id": 3,
    "market": "BTC_ETH",
    "side": "buy",
    "price": "0.00400000",
    "quantity": "10.00000000",
    "maker": true,
    "total": "0.04000000",
    "commission": "0.05000000",
    "commission_asset": "ETH",
    "date": "2019-11-11T19:35:01.897Z"
  }
]
```

### Deposit history (USER_DATA)
```
GET /api/v1/depositHistory  (HMAC SHA256)
```
Get deposits for an account, optionally filtered by asset.

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
asset | STRING | NO |
startTime | LONG | NO | Milliseconds.
endTime | LONG | NO | Milliseconds.
limit | INT | NO | Default 500; max 1000.
timestamp | LONG | NO |
recvWindow | LONG | NO |

**Response:**
```javascript
[
  {
    "user_id": "ca41eaeb-21af-4918-9131-91d19ba2a87a",
    "symbol": "CLAM",
    "address": "xLpyii7piMV6Y4jx1UKr5QfxyswkALniD3",
    "amount": "0.01",
    "txid": "00bbfec31dc6f2c0ff917524768786258bb057c471a536eaa960999c4e334c26",
    "confirmations": 30,
    "confirmed": true,
    "complete": false,
    "commission": "0.00000000",
    "state": "complete",
    "created_at": 1571159819902,
    "updated_at": 1571161613058,
    "vout": 0
  }
]
```

### Withdraw history (USER_DATA)
```
GET /api/v1/withdrawHistory  (HMAC SHA256)
```
Get withdrawals for an account, optionally filtered by asset.

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
asset | STRING | NO |
startTime | LONG | NO | Milliseconds. Both start and end are required to filter by time.
endTime | LONG | NO | Milliseconds.
limit | INT | NO | Default 500; max 1000.
timestamp | LONG | NO |
recvWindow | LONG | NO |

**Response:**
```javascript
[
  {
    "id": 52,
    "user_id": "ca41eaeb-21af-4918-9131-91d19ba2a87a",
    "symbol": "CLAM",
    "destination": "x9Ph1xyrN7z4vfzYuNCdujgrX2Ly2SQsPs",
    "amount": "10.00100000",
    "txid": "6d5069d30a87bcd4c7631890e20375119444adcb52b97bb77a8183fe960e437a",
    "state": "complete",
    "fee": "0.00100000",
    "spentFee": "0.00100000",
    "complete": true,
    "isStealth": false,
    "failed": false,
    "cancelled": false,
    "rejected": false,
    "reject_reason": "",
    "updated_at": 1572482164000
  }
]
```

### Deposit address (USER_DATA)
```
GET /api/v1/deposit  (HMAC SHA256)
```
Current deposit address(es) for a symbol, or all symbols if omitted.

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
symbol | STRING | NO |
timestamp | LONG | NO |
recvWindow | LONG | NO |

**Response:**
```javascript
[
  {
    "address": "xLpyii7piMV6Y4jx1UKr5QfxyswkALniD3",
    "symbol": "CLAM",
    "legacy": false
  }
]
```

### New deposit address (USER_DATA)
```
POST /api/v1/deposit  (HMAC SHA256)
```
Create a new deposit address for an asset. Limited to one new address per asset per 24 hours (`-2002`).

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
asset | STRING | YES |
timestamp | LONG | NO |
recvWindow | LONG | NO |

**Response:**
```javascript
{
  "address": "xLpyii7piMV6Y4jx1UKr5QfxyswkALniD3",
  "symbol": "CLAM"
}
```

## Api v2
CoinMarketCap-style public market data. These routes do not use HMAC auth.

### Markets
```
GET /api/v2/markets
```
Overview of market data for all pairs.

**Parameters:**
NONE

**Response:**
```javascript
{
  "BTC_CLAM": {
    "trading_pairs": "BTC_CLAM",
    "base_currency": "CLAM",
    "quote_currency": "BTC",
    "last_price": "0.00000106",
    "lowest_ask": "0.0000011",
    "highest_bid": "0.00000106",
    "price_change_percent_24h": "0",
    "base_volume": "2681.22604431",
    "quote_volume": "0.00296892",
    "highest_price_24h": "0.00000111",
    "lowest_price_24h": "0.00000106"
  }
}
```

### Ticker
```
GET /api/v2/ticker
```
24-hour pricing and volume summary for each market pair.

**Parameters:**
NONE

**Response:**
```javascript
{
  "BTC_CLAM": {
    "base_id": 580,
    "quote_id": 1,
    "last_price": "0.00000106",
    "base_volume": "2681.22604431",
    "quote_volume": "0.00296892",
    "isFrozen": "false"
  }
}
```

`base_id` / `quote_id` are CoinMarketCap unified cryptoasset IDs when known.

### Assets
```
GET /api/v2/assets
```
Summary for each currency on the exchange.

**Parameters:**
NONE

**Response:**
```javascript
{
  "CLAM": {
    "name": "Clam",
    "unified_cryptoasset_id": 460,
    "can_withdraw": "true",
    "can_deposit": "true",
    "min_withdraw": "0.00100000",
    "max_withdraw": "1234.00000000",
    "maker_fee": "0.002",
    "taker_fee": "0.002"
  }
}
```

`can_withdraw` / `can_deposit` are strings `"true"` / `"false"`.
`max_withdraw` is the exchange hot-wallet available balance for that asset.

### Orderbook
```
GET /api/v2/orderbook/{MARKET_PAIR}
```
Level 2 order book for a market pair.

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
level | INT | NO | `1` = best bid and ask only. `2` or `3` = arranged by best bids/asks. Default 3.
depth | INT | NO | `0` or omitted = full book. Otherwise total levels split across both sides (`depth / 2` per side). Max 1000.

Unknown markets return HTTP 404 plain text: `the symbol {MARKET_PAIR} is unknown`.

**Response:**
```javascript
{
  "timestamp": "1787851909",
  "bids": [
    ["0.00000106", "4.96874451"]
  ],
  "asks": [
    ["0.0000011", "1.23636364"]
  ]
}
```

`timestamp` is unix seconds as a string.

### Trades
```
GET /api/v2/trades/{MARKET_PAIR}
```
Recently completed **taker** trades for a market pair (up to 500).

**Parameters:**
NONE

**Response:**
```javascript
[
  {
    "trade_id": 40298,
    "price": "0.00000106",
    "base_volume": "0.00000018",
    "quote_volume": "0.17",
    "timestamp": 1787829474547,
    "type": "sell"
  }
]
```

Field names here are inverted from usual exchange usage:

* `quote_volume` is the **base** quantity (e.g. CLAM).
* `base_volume` is price × quantity (BTC).

## Legacy public endpoints
These older routes are still live and used by the exchange website. Prefer `/api/v1` or `/api/v2` for new integrations.

### Market list
```
GET /api/markets
```

```javascript
{
  "result": {
    "markets": ["BTC_CLAM", "BTC_DOGE"],
    "tickers": [ /* same objects as GET /api/v1/ticker/24hr */ ]
  }
}
```

### Market book
```
GET /api/market/{MARKET_PAIR}
```
Full book plus recent trades for the website. Unknown markets return HTTP 404 plain text.

### Ticker
```
GET /api/ticker/{MARKET_PAIR}
```

```javascript
{
  "result": {
    "market": "BTC_CLAM",
    "lastPrice": "0.00000106",
    "bidPrice": "0.00000106",
    "askPrice": "0.0000011"
  }
}
```

# Filters
Filters define trading rules on a market or the exchange.

## Market filters
### PRICE_FILTER
The `PRICE_FILTER` defines the `price` rules for a market:

* `minPrice` defines the minimum `price` allowed; disabled on `minPrice` == 0.
* `maxPrice` defines the maximum `price` allowed; disabled on `maxPrice` == 0.
* `tickSize` defines the intervals that a `price` can be increased/decreased by; disabled on `tickSize` == 0.

To pass the filter:

* `price` >= `minPrice`
* `price` <= `maxPrice`
* (`price`-`minPrice`) % `tickSize` == 0

```javascript
{
  "filterType": "PRICE_FILTER",
  "minPrice": "0.00000010",
  "maxPrice": "10000.00000000",
  "tickSize": "0.00000100"
}
```

### LOT_SIZE
The `LOT_SIZE` filter defines quantity rules for a market:

* `minQty` defines the minimum `quantity` allowed.
* `maxQty` defines the maximum `quantity` allowed.
* `stepSize` defines the intervals that a `quantity` can be increased/decreased by.

```javascript
{
  "filterType": "LOT_SIZE",
  "minQty": "0.01000000",
  "maxQty": "100000.00000000",
  "stepSize": "0.01000000"
}
```

### MARKET_LOT_SIZE
Same shape as `LOT_SIZE`, applied to market orders.

```javascript
{
  "filterType": "MARKET_LOT_SIZE",
  "minQty": "0.00000000",
  "maxQty": "63100.00000000",
  "stepSize": "0.00000000"
}
```

### MARKET_QUANTITY_INPUT
Defines which asset `quantity` is denominated in for market orders.

```javascript
{
  "filterType": "MARKET_QUANTITY_INPUT",
  "buy": "quote",
  "sell": "base"
}
```

A market **buy** `quantity` is BTC. A market **sell** `quantity` is the base coin.

### MAX_NUM_ORDERS
Maximum open orders an account may have on a market. Current limit is 100.

```javascript
{
  "filterType": "MAX_NUM_ORDERS",
  "maxNumOrders": 100
}
```

### MIN_ORDER_SIZE
Minimum order value (`price * quantity`). On AltQuick this is `0.00000010` BTC. The JSON field name is `minOrderSize`.

```javascript
{
  "filterType": "MIN_ORDER_SIZE",
  "minOrderSize": "0.00000010"
}
```

## Exchange Filters
### EXCHANGE_MAX_NUM_ORDERS
Maximum open orders an account may have across the exchange.

```javascript
{
  "filterType": "EXCHANGE_MAX_NUM_ORDERS",
  "maxNumOrdersExchange": 1000
}
```
