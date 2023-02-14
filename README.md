# dexbot

This is code for a basic market making trading bot against the Proton DEX, https://protondex.com/.

[![ESLint SAST scan workflow](https://github.com/squdgy/dexbot/actions/workflows/eslint.yml/badge.svg?event=push)](https://github.com/squdgy/dexbot/security/code-scanning)

![Tests](https://github.com/squdgy/dexbot/actions/workflows/test.js.yml/badge.svg?event=push)

This bot works against a single market. The purpose of the market making strategy is to put buy and sell orders on the DEX' order books. This strategy is probably a money losing strategy with fees taken into account. Use this as a reference and implement yor own trading algorithm.

This has been tested on testnet with XPR_XUSDC, XPR_XMD, and XBTC_XUSDT markets. It has been tested on mainnet with XPR_XUSDC, XPR_XMD, and XBTC_XUSDC markets.

## Getting Started

### prerequisites
- a proton account (https://www.proton.org/wallet/)
- enough funds in your account to buy and/or sell in the market that you want to trade in

### run the code
1. `npm install`
1. Add your account name and private key to environment variables, eg
```
export PROTON_USERNAME=user1
export PROTON_PRIVATE_KEY=PVT_K1_7yLfEMQXtFmCA3beLg6PSyiSp8paRBK2rdpLZ791XNAvRggXu
```
1. edit config/default.json to use the market you would like to trade in (symbol value)
1. `npm run bot`
1. To run on testnet: `npm run bot:test`

## config params
config/default.json has other config values you can change
```
{
  "bot" : {
    "api": {

      // api for readonly dex api
      "apiRoot": "https://metal-dexdb.global.binfra.one/dex",

      // api for readonly proton api
      "lightApiRoot": "https://lightapi.eosamsterdam.net/api"
    },

    // set to true in order to cancel all open orders when the bot shuts down
    "cancelOpenOrdersOnExit": false,

    "marketmaker": {
      // minimum spread to capture 0.001 = 0.1%
      "minSpread": 0.001,

      // how many buy and how many sell orders to put on the books
      "numPairs": 2,

      // market to trade in
      "symbol": "XPR_XUSDC"
    },
    "riskstrategy": {
      "symbol": "XPR_XUSDC",
      "riskManagementFactor": 0.05,
      "minimumTradingVolumePercentage": 0.2,
      "maximumVolatility": 0.5,
      "historicalDataCount": 100
    },

    "rpc": {

      // endpoints for RPC API
      "endpoints" : [
        "https://proton.greymass.com",
        "https://proton.eoscafeblock.com"
      ],

      // private key associated with username
      "privateKey": "PVT_K1_7yLfEMQXtFmCA3beLg6PSyiSp8paRBK2rdpLZ791XNAvRggXu",

      // permissions on the key ex. active or owner
      "privateKeyPermission": "active"
    },

    // how often to attempt trade
    "tradeIntervalMS": "5000",

    // username of trader AND API account
    "username": "user1"
  }
}
```

## Strategy
- **riskstrategy** - Modified strategy that uses a Reversion to the Mean algorithm to determine ideal price point and when to buy or sell.  Risk management represents the level of risk that a trader or an investor is willing to take on in a trade. It is a percentage value that determines the maximum amount of capital that an investor is willing to risk in a single trade.  Limits are placed on this for maximum volatility of the market as well as minimum trading volumn.  These values can be changed in the configuration.

## Actions available in this bot code base

### Markets
- **fetchLatestPrice** - retrieves the latest price for a given symbol
```
    const price = await fetchLatestPrice('XPR_XUSDC');
    logger.info(price);
```
- **fetchMarkets** - retrieves all markets that exist on the proton dex
```
    const response = await fetchMarkets();
    logger.info(response);
```
- **fetchOrderBook** - retrieves order book data for a single market
```
    const response = await fetchOrderBook('XBTC_XUSDC', 100, 0.01);
    logger.info(response);
```
- **fetchTrades** - retrieves trades on the given market
```
    const response = await fetchTrades('XPR_XUSDC', 100, 0);
    logger.info(response);
```

### Orders
- **cancelOrder** - cancel a single order
```
    const orderId = 966550;
    cancelOrder(orderId);
```
- **cancelAllOrders** - cancel all orders for a given user
```
    cancelAllOrders();
```
- **fetchOpenOrders** - retrieve all open orders for a given user
```
    const response = await fetchOpenOrders(username);
    logger.info(response);
```
- **fetchOrderHistory** - retrieves order history for a given user
```
    const response = await fetchOrderHistory('metallicus', 20, 0);
    logger.info(response);
```
- **submitLimitOrder** - submit a buy or sell limit order to the dex in postonly mode (ensure it is a maker trade)
```
    // place an order to sell XPR into USDC
    const quantity = 570;
    const price = 0.002020;
    submitLimitOrder('XPR_XUSDC', ORDERSIDES.SELL, quantity, price);
```

### Accounts
- **fetchBalances** - retrieves all balances for a given user
```    
    const response = await fetchBalances('metallicus');
    logger.info(response);
```

### coding references
- basics for a simple limit order placement, including signing: https://www.docs.protondex.com/developers-dex/submit-dex-order
- instructions on finding your private key: https://help.proton.org/hc/en-us/articles/4410313687703-How-do-I-backup-my-private-key-in-the-WebAuth-Wallet-
- actions available on the DEX contract: https://www.docs.protondex.com/developers-dex/actions
- general documentation on interacting with proton contracts: https://docs.protonchain.com/built-with-proton.html#sdks

