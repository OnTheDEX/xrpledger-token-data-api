# XRP Ledger Token Data API

## About this API
This API offers accurate and up-to-date token trading data for all tokens traded on the XRP Ledger DEX.  It is accessible to anyone.  Particular use cases might include:
* Cryptocurrency Aggregator sites such as CMC, CoinGecko, LiveCoinWatch and others.  Use this API to secure your data feed for any and all tokens traded on the XRP Ledger.
* Projects needing specific token data such as trading volume, current price, or historic OHLC price data.
* Personal projects eg. importation of token data into your portfolio tracking spreadsheets using Excel or Google Sheets, for example.

## Endpoint for access
The endpoint to use is:
`https://api.onthedex.live/public/v1`

## Data Paths
### `GET`: `/daily/tokens`
Returns a **list of tokens traded** on the XRP Ledger DEX at any time within the last rolling 24 hours, together with their key metrics.  Max returned items is 100, which in combination with the `by` parameter, allows getting the top 100 by volume, market cap, or trade count.

#### GET parameters to specify:
Parameter | Specification | Description
--- | --- | ---
`by` | `volume` (traded volume in USD equivalent value), `market_cap` (market capitalization, in USD), or `trades` (number of trades) | Sorts the returned token list by the parameter specified, in descending order.  Default: `volume`
`min_trades` | Integer | Filter the tokens returned to include only those tokens that have traded the specified number of times or more in the last 24 hours.  Use to exclude illiquid tokens particularly when ordering `by`=`market_cap`.  Default: `100`

#### Returned properties:
Property | Type | Description
--- | --- | ---
`tokens` | Array | List of tokens returned
`tokens[].currency` | String | Currency code of the token, eg. `USD`
`tokens[].issuer` | String | r-Address of the token issuer
`tokens[].token_name` | String | Official name of the token, if known
`tokens[].market_cap` | Integer | Market capitalization of the token in USD based on current mid price and issued suppy of the token
`tokens[].volume_token` | Float | The total volume of the token traded across all its pairs
`tokens[].volume_usd` | Float | The USD equivalent volume traded across all its pairs at the time of each trade taking place
`tokens[].price_mid_usd` | Float | The current mid price in USD equivalent based on the most liquid token pair (currently token/XRP)
`tokens[].last_trade_at` | ISO Timestamp | Date and time (in GMT) of the last trade of this token across all its pairs
`tokens[].num_trades` | Integer | Total number of individual trades on this token across all its pairs
`tokens[].suppy` | Float | Total number of tokens under supply obligation by the issuing account


### `GET`: `/daily/pairs`
Returns a **list of pairs (base / quote) traded** on the XRP Ledger DEX at any time within the last rolling 24 hours, together with their key metrics.

#### GET parameters to specify:
Parameter | Specification | Description
--- | --- | ---
`token` | String, optional | When not specified, returns complete list of traded pairs.  Note that this complete list includes inverted pairs (eg. TOKEN/USD.rhub and USD.rhub/TOKEN) with the same overall metrics for easier access to the pair you might want.  Be sure not to double-count such entries where relevant in your use case.  If `token` is specified as a concatenation of currency code and issuing account with a period `.` separator, returns only those pairs traded against the specified token.  Examples: `CSC.rCSCManTZ8ME9EoLrSHHYKW8PPwWMgkwr`, `XRP`
`by` | `volume` (traded volume in USD equivalent value), `trades` (number of trades) | Sorts the returned pairs by the parameter specified, in descending order.  Default: `volume`

#### Returned properties:
Property | Type | Description
--- | --- | ---
`pairs` | Array | List of tokens returned
`pairs[].base` | Object | Base currency specified with `currency` and `issuer` properties
`pairs[].base.currency` | String | Currency code of the base token
`pairs[].base.issuer` | String | r-Address of the base token issuer
`pairs[].quote` | String **or** Object | If quote currency is XRP, this contains the string `XRP`.  In all other cases, an object is returned with `currency` and `issuer` properties
`pairs[].quote.currency` | String | Currency code of the quote token
`pairs[].quote.issuer` | String | r-Address of the quote token issuer
`pairs[].volume_base` | Float | Volume of the base token traded this pair
`pairs[].volume_quote` | Float | Volume of the quote token traded this pair
`pairs[].volume_usd` | Float | Volume of USD equivalent value traded this pair
`pairs[].price_mid` | Float | Current mid price on the order books of this pairing.  If no orders on both sides of the book for this pair, result is `null`
`pairs[].price_hi` | Float | The highest price achieved across all trades of this pair
`pairs[].price_lo` | Float | The lowest price achieved across all trades of this pair
`pairs[].price_hi_usd` | Float | The highest USD equivalent price achieved across all trades of this pair
`pairs[].price_lo_usd` | Float | The lowest USD equivalent price achieved across all trades of this pair
`pairs[].num_trades` | Integer | Total number of individual trades on this pair



### `GET`: `/ohlc`
Returns **Open/High/Low/Close price data** for a given token pairing for a given timeframe interval.

#### GET parameters to specify:
Parameter | Specification | Description
--- | --- | ---
`base` | String | Concatenation of base currency code and issuing account with a period `.` separator.  Examples: `CSC.rCSCManTZ8ME9EoLrSHHYKW8PPwWMgkwr`, `XRP`
`quote` | String | Concatenation of quote currency code and issuing account with a period `.` separator.  Examples: `CSC.rCSCManTZ8ME9EoLrSHHYKW8PPwWMgkwr`, `XRP`
`interval` | `5`, `15`, `60`, `240`, `D`, or `W` | Interval timeframe for which OHLC data is required.  Integer values represent timeframe in minutes.
`ending` | Timestamp (optional) | The timestamp representing the last time bar to return.  If not specified, current time is used.  Examples: `2021-12-25T14:00:00Z`, `2022-01-01`, `2022-03-15 12:00:00`
`bars` | Integer | Number of bars of data to return, starting from the `ending` time and moving backwards for this number of bars.  Default varies depending on specified `tf`.  Maximum returned bars is 1000 per call.
`cf` | `yes` (optional) | Flag for requesting the carry-forward open price.  If set to `yes`, returns an extra property `ocf` for each time bar (see below)

#### Returned properties:
Property | Type | Description
--- | --- | ---
`spec` | Object | The interpreted specification from the passed parameters; what the API determined to be your request
`spec.base` | **or** Object | If base currency is XRP, this contains the string `XRP`.  In all other cases, and object is returned with `currency` and `issuer` properties
`spec.base.currency` | String | Currency code of the base token
`spec.base.issuer` | String | r-Address of the base token issuer
`spec.quote` | String **or** Object | If quote currency is XRP, this contains the string `XRP`.  In all other cases, an object is returned with `currency` and `issuer` properties
`spec.quote.currency` | String | Currency code of the quote token
`spec.quote.issuer` | String | r-Address of the quote token issuer
`spec.interval` | String | Interval timeframe
`spec.ending` | Timestamp | The passed `ending` parameter
`spec.bars` | Integer | The number of bars queried for.  Should be equal to the number of `data.ohlc` items returned.
`spec.cf` | String | The passed `cf` parameter
`data` | Object | The main returned data object
`data.ohlc[]` | Array | List of OHLC price bar data in ascending time order
`data.ohlc[].t` | ISO Timestamp | Timestamp in GMT of the opening time of the price bar
`data.ohlc[].o` | Float | Opening trading price of the time bar
`data.ohlc[].h` | Float | Highest trading price within the time bar
`data.ohlc[].l` | Float | Lowest trading price within the time bar
`data.ohlc[].c` | Float | Closing trading price of the time bar
`data.ohlc[].ocf` | Float | If `cf` was set to `yes`, this property contains the closing price of the previous bar to be used in client applications as the opening price of this bar.  This is distinct from the `o` price of this bar and permits the representation of this time bar opening at the same price the previous bar closed at, effectively ensuring there are no price 'gaps' in data bar-to-bar due to illiquid price jumps.
`data.ohlc[].vb` | Float | Volume of base currency traded within the time bar
`data.ohlc[].vq` | Float | Volume of quote currency traded within the time bar





