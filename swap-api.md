If you want to trade on the AltQuick exchange with an account, please see the https://github.com/AltQuick-com/api/blob/master/altquick-api-offical-docs.md.

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [Public Rest API for AltQuick Swap](#public-rest-api-for-altquick-swap)
  - [General API Information](#general-api-information)
  - [HTTP Return Codes](#http-return-codes)
  - [Error Codes](#error-codes)
  - [General Information on Endpoints](#general-information-on-endpoints)
- [Endpoint security type](#endpoint-security-type)
- [Public API Endpoints](#public-api-endpoints)
  - [Terminology](#terminology)
  - [Market endpoints](#market-endpoints)
    - [All markets](#all-markets)
    - [Market information](#market-information)
  - [Trade endpoints](#trade-endpoints)
    - [Open trade](#open-trade)
    - [Trade information](#trade-information)
    - [Trade history by deposit address](#trade-history-by-deposit-address)
- [Removed / unavailable endpoints](#removed--unavailable-endpoints)
- [Website WebSocket](#website-websocket)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# Public Rest API for AltQuick Swap

Accountless coin-to-coin swaps. Send coin A to a generated deposit address; AltQuick sells it, buys coin B, and withdraws to the address you supply.

## General API Information
* The base endpoint is: **https://altquick.com/swap/api/v1**
* All documented endpoints return JSON unless the body is empty.
* There is **no `{ "result": ... }` wrapper**. The action payload is the HTTP body.
* Pair symbols are `FROM-TO` (deposit coin, then receive coin), for example `CLAM-BTC`. This is **not** the exchange `QUOTE_BASE` underscore format (`BTC_CLAM`).
* Coin amounts in market quotes and in `GET /trade/:uuid` are decimal strings at 8 places (whole coins, not satoshis).
* `POST /trade` returns the raw trade row. Several fee fields on that row are satoshi-scale strings or `null` until the swap finishes.
* CORS: `Access-Control-Allow-Origin: *`, methods `GET, POST`.
* Rate limit: **10 requests per 10 seconds** per client. Limit headers are sent:
  * `X-Rate-Limit-Limit`
  * `X-Rate-Limit-Remaining`
  * `X-Rate-Limit-Reset`
* Gateway action timeout is 3 seconds.

## HTTP Return Codes

* HTTP `200` is used for successful responses. Unknown markets currently also return `200` with an **empty body**.
* HTTP `204` is used for CORS preflight (`OPTIONS`).
* HTTP `404` is used when the path does not match a registered alias.
* HTTP `422` is used for Moleculer parameter validation failures.
* HTTP `429` is used when the rate limit is exceeded.
* HTTP `5XX` return codes are used for internal errors.

## Error Codes
* Any endpoint can return an error.
* There are no numeric AltQuick exchange `code` values (`-1104`, etc.) on this API.

Validation failures (`HTTP 422`):

```javascript
{
  "errors": {
    "toAddress": "The 'toAddress' field is required!"
  }
}
```

Other Moleculer errors:

```javascript
{
  "name": "ServiceNotFoundError",
  "message": "Service 'btcfees' is not found.",
  "code": 404,
  "type": "SERVICE_NOT_FOUND",
  "data": {
    "action": "btcfees"
  }
}
```

Rate limit (`HTTP 429`):

```javascript
{
  "name": "RateLimitExceeded",
  "message": "Rate limit exceeded",
  "code": 429
}
```

## General Information on Endpoints
* `GET` parameters are sent in the path (and optionally the query string).
* `POST /trade` parameters are sent as a **JSON body** (`Content-Type: application/json`). They are **not** read from a query string.
* No API key, HMAC signature, or `X-MBX-APIKEY` header is used.

# Endpoint security type
* Every swap REST endpoint is `NONE` (public).
* Do not send exchange API keys to these routes.

# Public API Endpoints
## Terminology
* `fromCoin` / `FROM` is the **deposit** asset you send in.
* `toCoin` / `TO` is the **withdraw** asset you receive.
* `toAddress` is the destination address for `toCoin`.
* `fromAddress` is the generated (or reused) deposit address for `fromCoin`.
* `legacy` requests a legacy-format BTC deposit address. It is ignored unless `fromCoin` is `BTC`.

**Trade `state` values:**

* `awaitingDeposit` — waiting for an inbound deposit
* `refunded` — deposit returned (or marked refunded)
* `tradeComplete` — withdraw broadcast / swap finished

Related nested states, when present:

* Deposit: `depositConfirmed`
* Sell: `Pending`, `sellingComplete`
* Buy: `Pending`, `buyingComplete`
* Withdraw: `Pending`, `withdrawQueued`, `Complete`

**Listed coins** (from live `GET /markets`; the object is the source of truth):

`42`, `AVAX`, `BCH`, `BTC`, `CLAM`, `CURE`, `DASH`, `DGB`, `DOGE`, `FLO`, `GAP`, `LTC`, `MAZA`, `NMC`, `PART`, `PPC`, `QTUM`, `RHOM`, `SBTC`, `SOL`, `TBTC`, `TBTC4`, `WOW`, `XMR`, `ZEC`

Pairs that cannot currently fill are still listed with `"closed": true`, `min`/`max` of `0`, and zeroed `rates`.

## Market endpoints
### All markets
```
GET /markets
```
Current quotes, min/max deposit size, and open/closed flag for every `FROM-TO` pair.

**Parameters:**
NONE

**Response:** an object keyed by pair. Shape of each value is the same as [Market information](#market-information).

```javascript
{
  "CLAM-BTC": {
    "rates": { /* ... */ },
    "ratesWithFees": { /* ... */ },
    "max": "56900.43946089",
    "min": "129.72798113",
    "closed": false
  },
  "BTC-CLAM": {
    "rates": { /* ... */ },
    "ratesWithFees": { /* ... */ },
    "max": "0.09428366",
    "min": "0.00021003",
    "closed": false
  }
}
```

```
curl https://altquick.com/swap/api/v1/markets
```

### Market information
```
GET /market/:market
```
Quote for one pair.

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
market | STRING | YES | `FROM-TO`, for example `BTC-DOGE`.

* Unknown pairs return HTTP `200` with an empty body (`X-Response-Type: undefined`).
* Closed pairs still return JSON with `"closed": true`.

**Response:**
```javascript
{
  "rates": {
    "from": {
      "coin": "BTC",
      "avgrate": "1.00000000",
      "amount": "0.03561553",
      "lowestrate": "1.00000000",
      "firstrate": "1.00000000",
      "btctotal": "0.03561553"
    },
    "to": {
      "coin": "DOGE",
      "avgrate": "0.00000148",
      "amount": "23999.34652576",
      "highestrate": "0.00000158",
      "firstrate": "0.00000131",
      "btctotal": "0.03561553"
    },
    "min": "0.00021164",
    "max": "0.03561553"
  },
  "ratesWithFees": {
    "from": {
      "coin": "BTC",
      "avgrate": "1.00000000",
      "amount": "0.03561553",
      "lowestrate": "1.00000000",
      "firstrate": "1.00000000",
      "btctotal": "0.03561553"
    },
    "to": {
      "coin": "DOGE",
      "avgrate": "0.00000148",
      "amount": "23823.90182432",
      "highestrate": "0.00000158",
      "firstrate": "0.00000131",
      "btctotal": "0.03525937"
    },
    "fees": {
      "exchangeCommission": "0.00000000",
      "total": "0.00035616",
      "commission": "0.00035616"
    }
  },
  "max": "0.03561553",
  "min": "0.00021164",
  "closed": false
}
```

Field notes:

* `rates.from.amount` / `rates.to.amount` — indicative size at the current book.
* `rates.from.firstrate` — top-of-book rate (BTC per coin for alts; `1` when the side is BTC).
* `rates.from.avgrate` — size-weighted average rate for the quoted amount.
* `min` / `max` — deposit (`fromCoin`) bounds, including deposit-fee padding on `min`. Do not deposit more than `max`.
* `ratesWithFees.to.amount` — estimated receive amount after swap commission. `fees.commission` is in BTC.

```
curl https://altquick.com/swap/api/v1/market/BTC-DOGE
```

## Trade endpoints
### Open trade
```
POST /trade
```
Create a swap and allocate a deposit address.

This does **not** take an `amount` and does **not** return a locked quote. Size is determined by what you actually deposit (within the pair's `min` / `max` at fill time).

**Parameters:** JSON body.

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
fromCoin | STRING | YES | Deposit coin. Minimum length 2.
toCoin | STRING | YES | Receive coin. Minimum length 3.
toAddress | STRING | YES | Withdraw address for `toCoin`. Minimum length 30.
emergencyAddress | STRING | NO | Refund address for `fromCoin` if the swap cannot complete.
legacy | BOOLEAN | NO | Request a legacy BTC deposit address. Only applies when `fromCoin` is `BTC`.
affiliateId | STRING | NO | Affiliate id. Invalid ids are stored as `null`.
fromAddress | STRING | NO | Reuse an existing deposit address instead of generating one.
withdrawFee | NUMBER | NO | Accepted by validation. Currently **not applied**; the created row stores `"0"`.

The old field names `withdrawAddress`, `amount`, `withdrawRate`, `withdrawSpeed`, and `reuse` are **not** read. Sending `withdrawAddress` without `toAddress` returns `422`.

**Response:** the created trade row (Sequelize JSON). `fromAddress` is the deposit address to pay.

```javascript
{
  "id": 12345,
  "uuid": "669f8ef7-aa19-4b6d-b3cc-3e9c26119332",
  "fromCoin": "CLAM",
  "toCoin": "BTC",
  "fromAddress": "xUGTiLBBcbJHaygFHF7u3jq3eN5UgikGQT",
  "toAddress": "1GixdsuVAsQjo7rMNUWubbZnsr3bNBrVct",
  "emergencyAddress": null,
  "balanceFee": null,
  "exchangeFee": null,
  "exchangeFeeB": null,
  "underage": null,
  "overage": null,
  "withdrawFee": "0",
  "commission": null,
  "state": "awaitingDeposit",
  "working": false,
  "refunded": false,
  "flagged": false,
  "affiliateId": null,
  "fakeTrade": false,
  "complete": false,
  "createdAt": "2026-08-27T18:10:37.112Z",
  "updatedAt": "2026-08-27T18:10:37.112Z"
}
```

Poll `GET /trade/:uuid` (the path param is `uuid`, not `tradeId`) until `state` is `tradeComplete`. Send only `fromCoin` to `fromAddress`.

**Validation errors (`HTTP 422`):**

```javascript
{
  "errors": {
    "toCoin": "The 'toCoin' field is required!",
    "fromCoin": "The 'fromCoin' field is required!",
    "toAddress": "The 'toAddress' field is required!"
  }
}
```

```javascript
{
  "errors": {
    "toAddress": "The 'toAddress' field length must be greater than or equal to 30 characters long!"
  }
}
```

```
curl -X POST https://altquick.com/swap/api/v1/trade \
  -H "Content-Type: application/json" \
  -d '{
    "fromCoin": "CLAM",
    "toCoin": "BTC",
    "toAddress": "1GixdsuVAsQjo7rMNUWubbZnsr3bNBrVct"
  }'
```

### Trade information
```
GET /trade/:uuid
```
Formatted details for one swap.

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
uuid | STRING | YES | Trade UUID from `POST /trade`.

Amounts on this endpoint are converted from satoshis to 8-decimal coin strings. Nested objects are omitted until that step exists.

**Response:**
```javascript
{
  "uuid": "47ebb8d9-76b9-4e5b-85d5-e7352fa51a0c",
  "fromCoin": "BTC",
  "toCoin": "CLAM",
  "fromAddress": "3JwQ7ERYQwDjG8ozX1d4nkdEu28pirMhQm",
  "toAddress": "xNPM3fipHRzL2Jm3MUfnFgyqi8NUXMbsJb",
  "emergencyAddress": null,
  "state": "tradeComplete",
  "fees": {
    "exchangeFee": "0.00000174",
    "exchangeFeeB": "0.00000000",
    "commission": "0.00002000"
  },
  "Deposit": {
    "amount": "0.00100000",
    "confirmations": 1,
    "state": "depositConfirmed",
    "txid": "13e547dc5714c28a4ab5d72624d75f732f28f5412998b10c1641e6d00a1f9dbd",
    "fee": "0.00000000"
  },
  "Sell": {
    "amount": "0.00100000",
    "average": "1.00000000",
    "btc": "0.00100000",
    "state": "sellingComplete"
  },
  "Buy": {
    "amount": "1.73390054",
    "average": "0.00050187",
    "btc": "0.00086846",
    "state": "buyingComplete"
  },
  "Withdraw": {
    "amount": "1.73390054",
    "state": "Complete",
    "txid": "483ac4aa51f5ff5c667221b3656702c0021ca07376e6163f04b584987226c8de"
  }
}
```

This payload is **not** the old `{ tradeId, quote, exchangeOrders, status, created }` document. Use `uuid` and `state`.

* Invalid or unknown UUIDs are intended to return `422`. The current gateway error handler can fail to serialize those errors, so the HTTP request may hang instead of returning JSON. Successful lookups return the object above.

```
curl https://altquick.com/swap/api/v1/trade/47ebb8d9-76b9-4e5b-85d5-e7352fa51a0c
```

### Trade history by deposit address
```
GET /history/:address
```
Trades whose **deposit** address (`fromAddress`) equals `address`.

**Parameters:**

Name | Type | Mandatory | Description
------------ | ------------ | ------------ | ------------
address | STRING | YES | Deposit address, not the withdraw address.

**Intended response:** an array of full trade rows, including nested `Deposit`, `Sell`, `Buy`, and `Withdraw` records (raw database objects, not the `GET /trade/:uuid` formatter). Empty results are intended to return `422` (`No trades found for address`).

**Live status:** the public gateway alias is currently registered as `history:/address` (colon in the wrong place). `GET /history/:address` therefore returns `404`:

```javascript
{
  "name": "ServiceNotFoundError",
  "message": "Service 'history.xD933hvRUFwcUtrrT1rFqEJbNpiGge6JLh' is not found.",
  "code": 404,
  "type": "SERVICE_NOT_FOUND",
  "data": {
    "action": "history.xD933hvRUFwcUtrrT1rFqEJbNpiGge6JLh"
  }
}
```

Treat this route as **unavailable** until that alias is `GET history/:address`.

# Removed / unavailable endpoints

These exist only in older swap docs and are **not** served:

### BTC withdraw fees
```
GET /btcfees
```

```javascript
{
  "name": "ServiceNotFoundError",
  "message": "Service 'btcfees' is not found.",
  "code": 404,
  "type": "SERVICE_NOT_FOUND",
  "data": {
    "action": "btcfees"
  }
}
```

Network fees are no longer quoted from bitcoinfees.earn.com on this API. Pair `min` already includes withdraw- and deposit-fee padding.

### Breaking changes vs older `swap-api.md`

Old field / behavior | Current
------------ | ------------
`{ "result": ... }` wrapper | Unwrapped JSON
`GET /markets` array of pair names | Object keyed by `FROM-TO`
`fromQuantity` / `fromRate` / `fromPrice` | `rates.from.amount`, `avgrate`, `btctotal`
`withdrawAddress` | `toAddress`
`amount` on open (locked quote) | Not accepted; deposit size is the fill size
`withdrawRate` / `withdrawSpeed` | Not accepted
`reuse` | Optional `fromAddress`
`tradeId` / `status` / `created` | `uuid` / `state` / `createdAt`
`GET /btcfees` | Removed (`404`)

# Website WebSocket

The swap website uses a WebSocket, not these REST routes, to open and watch trades.

* URL: **wss://altquick.com/wss/swap**
* Messages are JSON `{ "event": "...", "payload": ... }`.

Client → server events:

Event | Payload
------------ | ------------
`ping` | none (server replies `pong`)
`open` | `{ from, to, address, emergencyAddress, affiliateId, legacy }`
`loadTrade` | `{ tradeId }`
`filterMarket` | `{ market, filterBy, amount }`

Server → client events include `init`, `tradeOpened`, `tradeLoaded`, `tradeComplete`, `marketFilter`, `recentTradeUpdate`, and `error`.

`open` calls the same `trade.handler.open` action as `POST /trade`. REST integrators do not need the WebSocket.
