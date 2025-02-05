# CryptoCurrency eXchange WebSockets

[![CircleCI](https://circleci.com/gh/altangent/ccxws/tree/master.svg?style=shield)](https://circleci.com/gh/altangent/ccxws/tree/master)
[![Coverage Status](https://coveralls.io/repos/github/altangent/ccxws/badge.svg?branch=master)](https://coveralls.io/github/altangent/ccxws?branch=master)

A JavaScript library for connecting to realtime public APIs on all cryptocurrency exchanges.

[Blocktap.io](https://www.blocktap.io), a free data analytics platform, uses CCXWS to provide real time data.

CCXWS provides a standardized eventing interface for connection to public APIs. Currently CCXWS support ticker, trade and orderbook events.

The CCXWS socket client performs automatic reconnection when there are disconnections. It also has silent reconnection logic to assist when no data has been seen by the client but the socket remains open.

CCXWS uses similar market structures to those generated by the CCXT library. This allows interoperability between the RESTful interfaces provided by CCXT and the realtime interfaces provided by CCXWS.

## Getting Started

Install dknafo-ccxws

```bash
npm install dknafo-ccxws
```

Create a new client for an exchange. Subscribe to the events that you want to listen to by supplying a market.

```javascript
const ccxws = require("dknafo-ccxws");
const binance = new ccxws.binance();

// market could be from CCXT or genearted by the user
const market = {
  id: "BTCUSDT", // remote_id used by the exchange
  base: "BTC", // standardized base symbol for Cardano
  quote: "USDT", // standardized quote symbol for Bitcoin
};

// handle trade events
binance.on("trade", trade => console.log(trade));

// handle level2 orderbook snapshots
binance.on("l2snapshot", snapshot => console.log(snapshot));

// subscribe to trades
binance.subscribeTrades(market);

// subscribe to level2 orderbook snapshots
binance.subscribeLevel2Snapshots(market);
```

## Exchanges

| Exchange     | Class       | Ticker  | Trades  | OB-L2 Snapshot | OB-L2 Updates | OB-L3 Snapshot | OB-L3 Updates |
| ------------ | ----------- | ------- | ------- | -------------- | ------------- | -------------- | ------------- |
| Bibox        | bibox       | Support | Support | Support        |               | -              | -             |
| Binance      | binance     | Support | Support | Support        | Support\*\*   | -              | -             |
| Bitfinex     | bitfinex    | Support | Support | -              | Support\*     | -              | Support\*     |
| bitFlyer     | bitflyer    | Support | Support | -              | Support\*\*   | -              | -             |
| BitMEX       | bitmex      |         | Support | -              | Support\*     | -              | -             |
| Bitstamp     | bitstamp    | -       | Support | Support        | Support\*\*   | -              | Support       |
| Bittrex      | bittrex     | Support | Support | -              | Support\*     | -              | -             |
| Cex.io       | cex         | Support | Support | -              | Support\*     | -              | -             |
| Coinex       | coinex      | Support | Support | -              | Support\*     | -              | -             |
| Coinbase Pro | coinbasepro | Support | Support | -              | Support\*     | -              | Support       |
| Ethfinex     | ethfinex    | Support | Support | -              | Support\*     | -              | Support\*     |
| Gate.io      | gateio      | Support | Support | -              | Support\*     | -              | -             |
| Gemini       | gemini      | -       | Support | -              | Support\*     | -              | -             |
| HitBTC       | hitbtc      | Support | Support | -              | Support\*     | -              | -             |
| Huobi        | huobi       | Support | Support | Support        | -             | -              | -             |
| Kraken       | kraken      | Support | Support | -              | Support\*     | -              | -             |
| OKEx         | okex        | Support | Support | Support        | Support\*     | -              | -             |
| Poloniex     | poloniex    | Support | Support | -              | Support\*     | -              | -             |
| Upbit        | upbit       | Support | Support | Support        | -             | -              | -             |
| ZB           | zb          | Support | Support | Support        | -             | -              | -             |

Notes:

- Support\*: Broadcasts a snapshot event at startup
- Support\*\*: Broadcasts a snapshot by using the REST API

## Definitions

Trades - A maker/taker match has been made. Broadcast as an aggregated event.

Orderbook level 2 - has aggregated price points for bids/asks that include the price and total volume at that point. Some exchange may include the number of orders making up the volume at that price point.

Orderbook level 3 - this is the most granual order book information. It has raw order information for bids/asks that can be used to build aggregated volume information for the price points.

## API

### `Market`

Markets are used as input to many of the client functions. Markets can be generated and stored by you the developer or loaded from the CCXT library.

These properties are required by CCXWS.

- `id: string` - the identifier used by the remote exchange
- `base: string` - the normalized base symbol for the market
- `quote: string` - the normalized quote symbol for the market

### `Client`

A websocket client that connects to a specific exchange. There is an implementation of this class for each exchange that governs the specific rules for managing the realtime connections to the exchange. You must instantiate the specific exchanges client to conncet to the exchange.

```javascript
const binance = new ccxws.Binance();
const gdax = new ccxws.GDAX();
```

#### Properties

##### `reconnectIntervalMs: int` - default 90000

Property that controls silent socket drop checking. This will enable a check at the reconnection interval that looks for ANY broadcast message from the server. If there has not been a message since the last check a reconnection (close, connect) operation is performed. This property must be set before the first subscription (and subsequent connection).

#### Events

Subscribe to events by addding an event handler to the client `.on(<event>)` method of the client. Multiple event handlers can be added for the same event.

Once an event handler is attached you can start the stream using the `subscribe<X>` methods.

All events emit the market used to subscribe to the event as a second property of the event handler.

```javascript
binance.on("trades", (trade, market) => console.log(trade, market));
binance.on("l2snapshot", (snapshot, market) => console.log(snapshot, market));
```

##### `ticker` emits `Ticker`, `Market`

Fired when a ticker update is received. Returns an instance of `Ticker` and the `Market` used to subscribe to the event.

##### `trade` emits `Trade`, `Market`

Fired when a trade is received. Returns an instance of `Trade` and the `Market` used to subscribe to the event.

##### `l2snapshot` emits `Level2Snapshot`, `Market`

Fired when a orderbook level 2 snapshot is received. Returns an instance of `Level2Snapshot` and the `Market` used to subscribe to the event.

The level of detail will depend on the specific exchange and may include 5/10/20/50/100/1000 bids and asks.

This event is also fired when subscribing to the `l2update` event on many exchanges.

##### `l2update` emits `Level2Update`, `Market`

Fired when a orderbook level 2 update is recieved. Returns an instance of `Level2Update` and the `Market` used to subscribe to the event.

Subscribing to this event may trigger an initial `l2snapshot` event for many exchanges.

##### `l3snapshot` emits `Level3Snapshot`, `Market`

Fired when a orderbook level 3 snapshot is received. Returns an instance of `Level3Snapshot` and the `Market` used to subscribe to the event.

##### `l3update` emits `Level3Update`, `Market`

Fired when a level 3 update is recieved. Returns an instance of `Level3Update` and the `Market` used to subscribe to the event.

#### Connection Events

##### `connected`

Fires when a socket has connected. This event will fire when for reconnections.

##### `closed`

Fires when the client has closed its connection(s). This event is not fired during disconnections, it is fired when the `close` method is called and the connection(s) are successfully closed.

##### `disconnected`

Fires when a socket has been disconnected. Automatic reconnection should will be performed and the next event will be `reconnected` followed by a `connected` event when the reconnection is successful.

##### `reconnected`

Fires when a socket has initiated the reconnection process. This is fired at the start of thee reconnection process and is more aptly named `reconnecting`.

#### Methods

##### `subscribeTicker(market): void`

Subscribes to a ticker feed for a market. This method will cause the client to emit `ticker` events that have a payload of the `Ticker` object.

##### `unsubscribeTicker(market): void`

Unsubscribes from a ticker feed for a market.

##### `subscribeTrades(market): void`

Subscribes to a trade feed for a market. This method will cause the client to emit `trade` events that have a payload of the `Trade` object.

##### `unsubscribeTrades(market): void`

Unsubscribes from a trade feed for a market.

\*For some exchanges, calling unsubscribe may cause a temporary disruption in all feeds.

##### `subscribeLevel2Snapshots(market): void`

Subscribes to the orderbook level 2 snapshot feed for a market. This method will cause the client to emit `l2snapshot` events that have a payload of the `Level2Snaphot` object.

This method is a no-op for exchanges that do not support level 2 snapshot subscriptions.

##### `unsubscribeLevel2Snapshots(market): void`

Unbusbscribes from the orderbook level 2 snapshot for a market.

\*For some exchanges, calling unsubscribe may cause a temporary disruption in all feeds.

##### `subscribeLevel2Updates(market): void`

Subscribes to the orderbook level 2 update feed for a market. This method will cause the client to emit `l2update` events that have a payload of the `Level2Update` object.

This method is a no-op for exchanges that do not support level 2 snapshot subscriptions.

##### `unsubscribeLevel2Updates(market): void`

Unbusbscribes from the orderbook level 2 updates for a market.

\*For some exchanges, calling unsubscribe may cause a temporary disruption in all feeds.

##### `subscribeLevel3Snapshots(market): void`

Subscribes to the orderbook level 3 snapshot feed for a market. This method will cause the client to emit `l3snapshot` events that have a payload of the `Level3Snaphot` object.

This method is a no-op for exchanges that do not support level 2 snapshot subscriptions.

##### `unsubscribeLevel3Snapshots(market): void`

Unbusbscribes from the orderbook level 3 snapshot for a market.

\*For some exchanges, calling unsubscribe may cause a temporary disruption in all feeds.

##### `subscribeLevel3Updates(market): void`

Subscribes to the orderbook level 3 update feed for a market. This method will cause the client to emit `l3update` events that have a payload of the `Level3Update` object.

This method is a no-op for exchanges that do not support level 3 snapshot subscriptions.

##### `unsubscribeLevel3Updates(market): void`

Unbusbscribes from the orderbook level 3 updates for a market.

\*For some exchanges, calling unsubscribe may cause a temporary disruption in all feeds.

### `Ticker`

The ticker class is the result of a `ticker` event.

#### Properties

- `fullId: string` - the normalized market id prefixed with the exchange, ie: `Binance:LTC/BTC`
- `exchange: string` - the name of the exchange
- `base: string` - the normalized base symbol for the market
- `quote: string` - the normalized quote symbol for the market
- `timestamp: int` - the unix timestamp in milliseconds
- `last: string` - the last price of a match that caused a tick
- `open: string` - the price 24 hours ago
- `low: string` - the highest price in the last 24 hours
- `high: string` - the lowest price in the last 24 hours
- `volume: string` - the base volume traded in the last 24 hours
- `quoteVolume: string` - the quote volume traded in the last 24 hours
- `change: string` - the price change (last - open)
- `changePercent: string` - the price change in percent (last - open) / open \* 100
- `bid: string` - the best bid price
- `bidVolume: string` - the volume at the best bid price
- `ask: string` - the best ask price
- `askVolume: string` - the volume at the best ask price

### `Trade`

The trade class is the result of a `trade` event emitted from a client.

#### Properties

- `fullId: string` - the normalized market id prefixed with the exchange, ie: `Binance:LTC/BTC`
- `exchange: string` - the name of the exchange
- `base: string` - the normalized base symbol for the market
- `quote: string` - the normalized quote symbol for the market
- `tradeId: string` - the unique trade identifer from the exchanges feed
- `unix: int` - the unix timestamp in milliseconds for when the trade executed
- `side: string` - whether the buyer `buy` or seller `sell` was the maker for the match
- `price: string` - the price at which the match executed
- `amount: string` - the amount executed in the match
- `buyOrderId: string` - the order id of the buy side
- `sellOrderId: string` - the order id of the sell side

### `Level2Point`

Represents a price point in a level 2 orderbook

#### Properties

- `price: string` - price
- `size: string` - aggregated volume for all orders at this price point
- `count: int` - optional number of orders aggregated into the price point

### `Level2Snapshot`

The level 2 snapshot class is the result of a `l2snapshot` or `l2update` event emitted from the client.

#### Properties

- `fullId: string` - the normalized market id prefixed with the exchange, ie: `Binance:LTC/BTC`
- `exchange: string` - the name of the exchange
- `base: string` - the normalized base symbol for the market
- `quote: string` - the normalized quote symbol for the market
- `timestampMs: int` - optional timestamp in milliseconds for the snapshot
- `sequenceId: int` - optional sequence identifier for the snapshot
- `asks: [Level2Point]` - the ask (seller side) price points
- `bids: [Level2Point]` - the bid (buyer side) price points

### `Level2Update`

The level 2 update class is a result of a `l2update` event emitted from the client. It consists of a collection of bids/asks even exchanges broadcast single events at a time.

#### Properties

- `fullId: string` - the normalized market id prefixed with the exchange, ie: `Binance:LTC/BTC`
- `exchange: string` - the name of the exchange
- `base: string` - the normalized base symbol for the market
- `quote: string` - the normalized quote symbol for the market
- `timestampMs: int` - optional timestamp in milliseconds for the snapshot
- `sequenceId: int` - optional sequence identifier for the snapshot
- `asks: [Level2Point]` - the ask (seller side) price points
- `bids: [Level2Point]` - the bid (buyer side) price points

### `Level3Point`

Represents a price point in a level 3 orderbook

#### Properties

- `orderId: string` - identifier for the order
- `price: string` - price
- `size: string` - volume of the order
- `meta: object` - optional exchange specific metadata with additional information about the update.

### `Level3Snapshot`

The level 3 snapshot class is the result of a `l3snapshot` or `l3update` event emitted from the client.

#### Properties

- `fullId: string` - the normalized market id prefixed with the exchange, ie: `Binance:LTC/BTC`
- `exchange: string` - the name of the exchange
- `base: string` - the normalized base symbol for the market
- `quote: string` - the normalized quote symbol for the market
- `timestampMs: int` - optional timestamp in milliseconds for the snapshot
- `sequenceId: int` - optional sequence identifier for the snapshot
- `asks: [Level3Point]` - the ask (seller side) price points
- `bids: [Level3Point]` - the bid (buyer side) price points

### `Level3Update`

The level 3 update class is a result of a `l3update` event emitted from the client. It consists of a collection of bids/asks even exchanges broadcast single events at a time.

Additional metadata is often provided in the `meta` property that has more detailed information that is often required to propertly manage a level 3 orderbook.

#### Properties

- `fullId: string` - the normalized market id prefixed with the exchange, ie: `Binance:LTC/BTC`
- `exchange: string` - the name of the exchange
- `base: string` - the normalized base symbol for the market
- `quote: string` - the normalized quote symbol for the market
- `timestampMs: int` - optional timestamp in milliseconds for the snapshot
- `sequenceId: int` - optional sequence identifier for the snapshot
- `asks: [Level3Point]` - the ask (seller side) price points
- `bids: [Level3Point]` - the bid (buyer side) price points
