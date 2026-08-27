# Error codes for AltQuick

Most `/api/v1` errors return **HTTP 409 Conflict** with:

```javascript
{
  "code": -1104,
  "message": "Invalid market"
}
```

Some signed-account failures instead return:

```javascript
{
  "errors": {
    "msg": "INVALID_API_KEY"
  }
}
```

v2 unknown-market errors are **HTTP 404** with a plain-text body, for example:

```
the symbol NOPE is unknown
```

Codes are stable. `message` text can vary.

## 10xx — General / auth / limits

#### -1000 UNKNOWN_ERROR
* An unknown error occurred while processing the request.

#### -1001 UNAUTHORIZED
* You are not authorized to execute this request.

#### -1002 TOO_MANY_REQUESTS_PER_SECOND
* Too many requests per second.

#### -1003 TOO_MANY_REQUESTS_PER_HOUR
* Too many requests per hour.

#### -1004 TOO_MANY_REQUESTS_PER_DAY
* Too many requests per day.

#### -1005 TOO_MANY_ORDERS
* Too many of your open orders in the requested market.
* Current per-market open-order cap is 100.

#### -1006 CONFLICTING_ORDER
* Conflicting order; check open orders.

#### -1008 EXCHANGE_NOT_ACTIVE
* Exchange is currently not accepting orders.

#### -1009 INVALID_SIGNATURE
* Signature for this request is not valid.

#### -1010 INVALID_TIMESTAMP
* Invalid start or end time.

## 11xx — Request / market validation

#### -1100 INVALID_QUANTITY
* Invalid order or withdraw quantity.
* Quantity under the minimum withdraw amount.

#### -1101 INVALID_PRICE
* Invalid order price.

#### -1102 INVALID_ORDER_SIDE
* Invalid order side. Must be `buy` or `sell`.

#### -1103 INVALID_ORDER_TYPE
* Invalid order type. Must be `limit` or `market`.

#### -1104 INVALID_MARKET
* Invalid or missing market.

#### -1105 INVALID_INTERVAL
* Invalid kline interval.

#### -1106 INVALID_LIMIT
* Invalid limit.

#### -1107 MARKET_CLOSED
* Market is not accepting orders.

#### -1108 INVALID_SYMBOL
* Invalid symbol.

#### -1109 ORDER_TOO_SMALL
* Order would execute under the value threshold.

#### -1110 INVALID_PRECISION
* Invalid price or quantity precision.

#### -1111 MISSING
* A required field is missing.

#### -1112 IS_EMPTY
* A required field is empty.

#### -1113 ORDER_DOES_NOT_EXIST
* Order does not exist, or `orderID` / `orderId` is missing.

#### -1114 EMPTY_DATA_SET
* No matching records.

#### -1115 INVALID_ASSET
* Invalid or unsupported asset / symbol.

## 20xx — Account actions

#### -2000 CREATE_ORDER_ERROR
* Could not process the order; it was canceled.

#### -2001 NOT_ENOUGH_BALANCE
* Balance too low for the request.

#### -2002 ADDRESS_TOO_SOON
* Previous deposit address for this asset was created less than 24 hours ago.

#### -2010 INVALID_API_KEY
* Invalid, inactive, or missing API key.

## 90xx — Filters

#### -9000 MIN_NOTIONAL
* Order under the minimum BTC value (`0.00000010` BTC).

## 100xx — Internal transfers

#### -10000 FAILED_FUND_MOVE
* Fund move failed (invalid user, low balance, or banned).

## Errors without a numeric `code`

These are often returned as `{ "errors": { "msg": "..." } }`:

| `msg` | Meaning |
| --- | --- |
| `INVALID_ADDRESS` | Destination or address is not valid for the asset. |
| `CONTRACT_WITHDRAW_UNSUPPORTED` | Withdraw to a contract address is not supported. |
| `USER_FROZEN` | Account is frozen. |
| `FAILED_IS_AFFILIATE` | Invalid affiliate ID. |
| `TRACK_SWAP_ERROR` | Swap tracking failed. |
