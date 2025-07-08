Altquick API Documentation
This document provides detailed information about the Altquick API endpoints for interacting with market and trade functionalities.
Base URL
All API endpoints are relative to the base URL: https://altquick.com/swap/api/v1
Endpoints
Market Endpoints
Get Market Information
GET /market/:market
Retrieves details for a specific market trade pair.
Parameters:



Name
Type
Description
Required



market
String
Trade pair
Yes


Success Response:

Code: 200
Content:{
  "result": {
    "fromCoin": "CLAM",
    "toCoin": "DOGE",
    "fromQuantity": 4.04716283,
    "fromPrice": 0.00217745,
    "fromRate": 0.00053802,
    "toQuantity": 3436.22947254,
    "toPrice": 0.00206174,
    "toRate": 0.00000059
  }
}



Error Responses:

Code: 500
WithdrawCoinInactive:{ "error": "Receiving coin inactive" }


DepositCoinInactive:{ "error": "Deposit coin inactive" }





Sample Request:
curl https://altquick.com/swap/api/v1/market/CLAM-DOGE


Get Active Trade Pairs
GET /markets
Returns a list of currently active trade pairs.
Success Response:

Code: 200
Content:{
  "result": [
    "CLAM-BTC",
    "BTC-CLAM",
    "CLAM-DOGE",
    "DOGE-CLAM",
    "BTC-DOGE",
    "DOGE-BTC"
  ]
}



Sample Request:
curl https://altquick.com/swap/api/v1/markets


Trade Endpoints
Get BTC Withdraw Fees
GET /btcfees
Returns the current Bitcoin network withdrawal fee rates in sats per byte. Fees are sourced from https://bitcoinfees.earn.com/.
Success Response:

Code: 200
Content:{
  "result": {
    "fastestFee": 24,
    "halfHourFee": 24,
    "hourFee": 14
  }
}



Error Response:

Code: 500{ "error": "Unknown Api Error" }



Sample Request:
curl https://altquick.com/swap/api/v1/btcfees


Get Trade History
GET /history/:address
Returns all associated trades for a given address.
Parameters:



Name
Type
Description
Required



address
String
The address to search for
Yes


Success Response:

Code: 200
Content:{
  "result": [{
    "tradeId": "47ebb8d9-76b9-4e5b-85d5-e7352fa51a0c",
    "fromCoin": "BTC",
    "toCoin": "CLAM",
    "fromQuantity": "0.00100000",
    "fromRate": "1.00000000",
    "fromTotal": "0.00100000",
    "toQuantity": "1.73390054",
    "toRate": "0.00050187",
    "toTotal": "0.00086846",
    "deposit": {
      "amount": "0.00100000",
      "address": "3JwQ7ERYQwDjG8ozX1d4nkdEu28pirMhQm",
      "txid": "13e547dc5714c28a4ab5d72624d75f732f28f5412998b10c1641e6d00a1f9dbd",
      "confirmations": 1,
      "confirmed": true
    },
    "withdraw": {
      "amount": "1.73390054",
      "address": "xNPM3fipHRzL2Jm3MUfnFgyqi8NUXMbsJb",
      "txid": "483ac4aa51f5ff5c667221b3656702c0021ca07376e6163f04b584987226c8de"
    },
    "fees": {
      "balance": "0.00010935",
      "withdraw": "0.00000045",
      "tradeCommission": "0.00002000",
      "exchangeCommission": "0.00000174",
      "total": "0.00013109"
    },
    "exchangeOrders": {
      "sell": [],
      "buy": [
        {
          "quantity": "1.73390054",
          "rate": "0.00050187",
          "total": "0.00086846",
          "exchangeCommission": "0.00000174"
        }
      ]
    },
    "quote": {
      "from": {
        "coin": "BTC",
        "quantity": "0.00194369",
        "rate": "1.00000000",
        "total": "0.00194369"
      },
      "to": {
        "coin": "CLAM",
        "quantity": "3.97062238",
        "rate": "0.00045117",
        "total": "0.00179143"
      },
      "fees": {
        "balance": "0.00010935",
        "withdraw": "0.00000045",
        "commission": "0.00003887",
        "exchangeCommission": "0.00000359",
        "total": "0.00015227"
      }
    },
    "status": "tradeComplete",
    "created": "2018-12-11T23:21:45.151Z"
  }]
}



Error Response:

Code: 500{ "error": "Unknown Api Error" }



Sample Request:
curl https://altquick.com/swap/api/v1/history/xD933hvRUFwcUtrrT1rFqEJbNpiGge6JLh


Get Trade Information
GET /trade/:trade
Retrieves details for a specific trade by its UUID.
Parameters:



Name
Type
Description
Required



trade
String
Uuid for trade
Yes


Success Response:

Code: 200
Content:{
  "result": {
    "tradeId": "47ebb8d9-76b9-4e5b-85d5-e7352fa51a0c",
    "fromCoin": "BTC",
    "toCoin": "CLAM",
    "fromQuantity": "0.00100000",
    "fromRate": "1.00000000",
    "fromTotal": "0.00100000",
    "toQuantity": "1.73390054",
    " toRate": "0.00050187",
    "toTotal": "0.00086846",
    "deposit": {
      "amount": "0.00100000",
      "address": "3JwQ7ERYQwDjG8ozX1d4nkdEu28pirMhQm",
      "txid": "13e547dc5714c28a4ab5d72624d75f732f28f5412998b10c1641e6d00a1f9dbd",
      "confirmations": 1,
      "confirmed": true
    },
    "withdraw": {
      "amount": "1.73390054",
      "address": "xNPM3fipHRzL2Jm3MUfnFgyqi8NUXMbsJb",
      "txid": "483ac4aa51f5ff5c667221b3656702c0021ca07376e6163f04b584987226c8de"
    },
    "fees": {
      "balance": "0.00010935",
      "withdraw": "0.00000045",
      "tradeCommission": "0.00002000",
      "exchangeCommission": "0.00000174",
      "total": "0.00013109"
    },
    "exchangeOrders": {
      "sell": [],
      "buy": [
        {
          "quantity": "1.73390054",
          "rate": "0.00050187",
          "total": "0.00086846",
          "exchangeCommission": "0.00000174"
        }
      ]
    },
    "quote": {
      "from": {
        "coin": "BTC",
        "quantity": "0.00194369",
        "rate": "1.00000000",
        "total": "0.00194369"
      },
      "to": {
        "coin": "CLAM",
        "quantity": "3.97062238",
        "rate": "0.00045117",
        "total": "0.00179143"
      },
      "fees": {
        "balance": "0.00010935",
        "withdraw": "0.00000045",
        "commission": "0.00003887",
        "exchangeCommission": "0.00000359",
        "total": "0.00015227"
      }
    },
    "status": "tradeComplete",
    "created": "2018-12-11T23:21:45.151Z"
  }
}



Error Responses:

Code: 500
Unknown:{ "error": "Unknown Api Error" }


TradeFlagged:{ "error": "Trade Flagged" }





Sample Request:
curl https://altquick.com/swap/api/v1/trade/47ebb8d9-76b9-4e5b-85d5-e7352fa51a0c


Open Trade
POST /trade
Initiates a new trade.
Parameters:



Name
Type
Description
Required
Default



fromCoin
String
Deposit coin
Yes



toCoin
String
Withdraw coin
Yes



withdrawAddress
String
Address to send coins to
Yes



amount
Amount
Receiving coin amount to get quote for
No



withdrawRate
Number
Custom withdraw rate for BTC
No



withdrawSpeed
String
Custom withdraw rate for BTC
No
fastest


emergencyAddress
String
Emergency address to return coins if trade fails
No



reuse
Boolean
Creates a reusable deposit address
No



Success Response:

Code: 200
Content:{
  "result": {
    "tradeId": "669f8ef7-aa19-4b6d-b3cc-3e9c26119332",
    "quote": {
      "from": {
        "coin": "CLAM",
        "quantity": "49.18320490",
        "rate": "0.00047230",
        "total": "0.02322900"
      },
      "to": {
        "coin": "BTC",
        "quantity": "0.02216869",
        "rate": "1.00000000",
        "total": "0.02216869"
      },
      "fees": {
        "balance": "0.00037016",
        "withdraw": "0.00006500",
        "tradeCommission": "0.00058073",
        "exchangeCommission": "0.00004443",
        "total": "0.00106031"
      }
    },
    "deposit": {
      "address": "xUGTiLBBcbJHaygFHF7u3jq3eN5UgikGQT"
    },
    "withdraw": {
      "address": "1GixdsuVAsQjo7rMNUWubbZnsr3bNBrVct"
    },
    "created": "2018-12-13T02:10:37.112Z",
    "status": "awaitingDeposit"
  }
}



Error Responses:

Code: 500
Unknown:{ "error": "Unknown Api Error" }


InvalidWithdrawAddress:{ "error": "Invalid Withdraw Address" }


InvalidEmergencyAddress:{ "error": "Invalid Emergency Address" }


WithdrawRateOnlyBtc:{ "error": "Withdraw rate only applys to bitcoin withdraws" }


WithdrawFeeTooHigh:{ "error": "BTC withdraw fee rate too high" }


WithdrawFeeToLow:{ "error": "BTC withdraw fee too low" }


DepositCoinInactive:{ "error": "Deposit coin inactive" }


WithdrawCoinInactive:{ "error": "Receiving coin inactive" }


GereateAddressError:{ "error": "Could not generate deposit address" }


OpenTradeFailed:{ "error": "Open trade failed" }


NoWithdrawAddress:{ "error": "No withdraw address submitted" }


NoDepositCoin:{ "error": "No deposit coin submitted" }


NoWithdrawCoin:{ "error": "No withdraw coin submitted" }





Sample Request:
curl -X POST https://altquick.com/swap/api/v1/trade \
  -H "Content-Type: application/json" \
  -d '{
    "fromCoin": "CLAM",
    "toCoin": "BTC",
    "withdrawAddress": "1GixdsuVAsQjo7rMNUWubbZnsr3bNBrVct",
    "amount": "49.18320490",
    "withdrawSpeed": "fastest"
  }'

Notes

All endpoints return JSON responses.
Ensure proper authentication and authorization headers are included where required.
Error responses follow a consistent format with an error field describing the issue.
