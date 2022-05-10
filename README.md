# XRP Ledger Token Data API

## About this API
This API offers accurate and up-to-date token trading data for all tokens traded on the XRP Ledger DEX.  It is accessible to anyone.  Particular use cases might include:
* Cryptocurrency Aggregator sites such as CMC, CoinGecko, LiveCoinWatch and others.  Use this API to secure your data feed for any and all tokens traded on the XRP Ledger.
* Projects needing specific token data such as trading volume, current price, or historic OHLC price data.
* Personal projects eg. importation of token data into your portfolio tracking spreadsheets using Excel or Google Sheets, for example.

## Endpoint for access
`https://api.onthedex.live/public/v1`

### `GET`: `/daily/tokens`
Returns a list of tokens  traded on the XRP Ledger DEX at any time within the last rolling 24 hours, together with their key metrics.

Property | Specification | Description
--- | --- | ---
`by` | `volume` (traded volume in USD equivalent value, default) `market_cap` (market capitalization, in USD), `trades` (number of trades) | Sorts the returned token list by the parameter specified, in descending order.
`min_trades` | integer, default `100` | Filter the tokens returned to include only those tokens that have traded the specified number of times or more in the last 24 hours.


