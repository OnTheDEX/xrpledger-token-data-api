# XRP Ledger Token Data API
#### Powered by [`OnTheDEX.live`](https://onthedex.live/)
![OnTheDex Logo](https://onthedex.live/onthedex_logo_simple.svg)

## About this API
This API offers accurate and up-to-date token trading data for all tokens traded on the XRP Ledger DEX.  The data source is the XRP Ledger itself.  This API is accessible to anyone.  Particular use cases might include:
* Cryptocurrency Aggregator sites such as CMC, CoinGecko, LiveCoinWatch and others.  Such sites can use this API to secure a data feed for any and all tokens traded on the XRP Ledger.
* Projects needing specific token data such as trading volume, current price, or historic OHLC price data.
* Personal projects eg. importation of token data into your portfolio tracking spreadsheets using Excel or Google Sheets, for example.

Methods are available for static data retrieval via HTTP `GET` and `POST` methods, and some data is also available streamed via `WEBSOCKET` connections.

## Endpoints for access
The HTTP endpoint to use for `GET` and `POST` requests is:
```
https://api.onthedex.live/public/v1
```

The `WEBSOCKET` endpoint to use for websocket paths is:
```
wss://api.onthedex.live/public/v1
```

## Things to know
* All times in GMT.
* All responses are JSON format unless specified otherwise.


## API Error trapping
Only return status codes of `200` are valid; any other status code returned means the API did not function as intended for your request.  Additionally, having received a valid `200` return status code from your request, 
always check the root JSON return object for the `error` property.  If set, this will be a string error code accompanied by a `message` property string explaining the error in more detail.  Example:
```json
{
    "error": "ERROR_MAINTENANCE", 
    "message": "The system is under maintenance for a short period.  Please try again later."
}
```


## Websocket specifics
When using the websocket versions of some endpoint paths described below, there are 3 possible outcomes:
- an error message with `error` property; or 
- a `ping` property with other details denoting current status at the server level of this channel; or
- a proper data packet as described below for each websocket endpoint path. 

### Websocket ping messages
When there is no `error` and no changed data needing to be sent to you, a `ping` message will be sent to signify that the connection is still alive and the current status of the channel on watch at the server.
(There is no need to send a ping request to the server from your client.)

Property | Type | Description
--- | --- | ---
`ping` | Integer | Timestamp of ping event from server
`channel` | String | Path of the channel this websocket is listening on for data
`status` | String | Current status of this channel (for example: `watching`) means the channel is alive and watching for property changes to transmit to you
`message` | String | Other descriptive data (for example: `no change`) means no change in the dataset was detected since you last received a valid dataset



## Documentation
Path | Methods | Description
--- | --- | ---
[`/ticker/:tokens_or_pairs`](#get-or-websocket-tickertokens_or_pairs) | `GET`, `WEBSOCKET` | Latest ticker information for a given token, pairing or group of pairs.
[`/daily/tokens`](#get-dailytokens) | `GET` | Headline Daily Token Data for top 100 traded tokens by volume, market cap or number of trades.
[`/daily/pairs`](#get-dailypairs) | `GET` | Headline Daily Traded Pair Data by volume or number of trades.
[`/aggregator`](#getpost-aggregator) | `GET`, `POST` | Aggregator data for all tokens traded in the last rolling 24 hours, including token metrics, fiat USD equivalent pricing and volumes, and individual token pairing data.
[`/ohlc`](#get-ohlc) | `GET` | Candlestick chart data including Open/High/Low/Close price data, base and quote volumes, for varying intervals.



# Data Paths

## `GET` or `WEBSOCKET`: `/ticker/:tokens_or_pairs`
Returns data for the specified tokens, pairs or (if `:token_or_pairs` is unspecified) a list of latest traded tokens within the last rolling 24 hours.

If a single token is requested, either as a base (`/ticker/TOKEN.ISSUER`) or as a quote (`/ticker/:TOKEN.ISSUER`), the server will return all pairs traded against the token in the last 24 hours.

If a specific pair (or pairs) is requested, those specific pair(s) are returned irrespective of whether they have traded in the last rolling 24 hours or not.
The difference will be in the `volume_base` and `volume_quote` properties which will be 0 (zero) if not traded in the last 24 hours, together with a `time` property that represents a time earlier than 24 hours ago.

#### Example `GET`:
To return data for all pairs recently traded in the last rolling 24 hours...
```
    // ...with SOLO as the base currency:
    https://api.onthedex.live/public/v1/ticker/SOLO.rsoLo2S1kiGeCcn6hCUXVrCpGMWLrRrLZz

    // ... with SOLO as the quote currency:
    https://api.onthedex.live/public/v1/ticker/:SOLO.rsoLo2S1kiGeCcn6hCUXVrCpGMWLrRrLZz

    // ... for all pairs (no specific token filter):
    https://api.onthedex.live/public/v1/ticker
```

To return a specific pair or pairs (returning most recent trade data even if outside of 24 hours):
```
    // ...for CSC/XRP only:
    https://api.onthedex.live/public/v1/ticker/CSC.rCSCManTZ8ME9EoLrSHHYKW8PPwWMgkwr:XRP
    
    // for XRP/CSC only:
    https://api.onthedex.live/public/v1/ticker/XRP:CSC.rCSCManTZ8ME9EoLrSHHYKW8PPwWMgkwr

    // for SOLO/XRP and CSC/XRP (2 pairs in one request):
    https://api.onthedex.live/public/v1/ticker/CSC.rCSCManTZ8ME9EoLrSHHYKW8PPwWMgkwr:XRP+SOLO.rsoLo2S1kiGeCcn6hCUXVrCpGMWLrRrLZz:XRP
```

Other options are available as specified below to format responses in a preferred way.


#### Example `WEBSOCKET`:
Websocket data is available as above except via the websocket endpoint beginning `wss://`.  Data packets will be returned approximately every 4 seconds.
```
// create websocket instance
var mySocket = new WebSocket("wss://api.onthedex.live/public/v1/ticker/CSC.rCSCManTZ8ME9EoLrSHHYKW8PPwWMgkwr:XRP");

// add event listener for receiving data from the API
mySocket.onmessage = function (event) {
    const token_data = JSON.parse(event.data);
    // always check there is no token_data.error first
    if (!token_data['error']) {
        // all good, so access .pairs array for the data on the token(s) requested
        token_data.pairs && token_data.pairs.forEach(pair => {
            const theBaseCurrency = (pair.b.c ? pair.b.c : pair.b);
            const theQuoteCurrency = (pair.q.c ? pair.q.c : pair.q);
            stringMessage = 
            console.log('Token pair ' + theBaseCurrency + '/' + theQuoteCurrency + ' was last traded at a price of ' + pair.l + ' ' + theQuoteCurrency);
        });
    } else {
        console.log('There was an error returned from the websocket API:', token_data.error, token_data.message);
    }
};
```


#### Parameters to specify:
These parameters are valid for both `GET` and `WEBSOCKET` requests:

Parameter | Specification | Description
--- | --- | ---
`by` | `volume` (traded volume in USD equivalent value, descending), `trades` (number of trades, descending), `base` (base currency code, ascending) | Sorts the returned pairs by the parameter specified.  Default: `volume`
`page` | Integer | Data page number to return.  Default is page `1`.
`per_page` | Integer | Maximum number of results to return for this page.  Default (and max) is `100`.


#### Returned properties:
Remember to always check the `error` property first before attempting to process any of the below returned parameters.

Property (`GET`) | Property (`WEBSOCKET`) | Type | Description
--- | --- | ---
`pairs` | `pairs` | Array | List of pairings returned
`pairs[].base` | `pairs[].b` | Object | If base currency is XRP, this contains the string `XRP`.  In all other cases, an object is returned with `currency` and `issuer` properties
`pairs[].base.currency` | `pairs[].b.c` | String | Currency code of the base token
`pairs[].base.issuer` | `pairs[].b.i` |String | r-Address of the base token issuer
`pairs[].quote` | `pairs[].q` | String **or** Object | If quote currency is XRP, this contains the string `XRP`.  In all other cases, an object is returned with `currency` and `issuer` properties
`pairs[].quote.currency` | `pairs[].q.c` | String | Currency code of the quote token
`pairs[].quote.issuer` | `pairs[].q.i` | String | r-Address of the quote token issuer (not specified if quote = `XRP`)
`pairs[].volume_base` | `pairs[].vb` | Float | Volume of the base token traded this pair in the last rolling 24 hours
`pairs[].volume_quote` | `pairs[].vq` | Float | Volume of the quote token traded this pair in the last rolling 24 hours
`pairs[].volume_usd` | `pairs[].vfx` | Float | Volume of USD equivalent value traded this pair in the last rolling 24 hours
`pairs[].price_mid` | `pairs[].pm` | Float | Current mid price on the order books of this pairing.  If no orders on both sides of the book for this pair, result is `null`
`pairs[].price_hi` | `pairs[].ph` | Float | The highest price achieved across all trades of this pair in the last rolling 24 hours
`pairs[].price_lo` | `pairs[].pl` | Float | The lowest price achieved across all trades of this pair in the last rolling 24 hours
`pairs[].last` | `pairs[].l` | Float | The price achieved on most recent trade of the pair
`pairs[].time` | `pairs[].t` | Integer, UNIX timestamp (seconds) | Time of most recent trade of the pair
`pairs[].ago24` | `pairs[].a24` | Float | The price last traded from (rolling) 24 hours ago
`pairs[].pc24` | `pairs[].pc24` | Float | The percentage change in price from (rolling) 24 hours ago to last traded
`pairs[].trend` | `pairs[].tr` | String | Either `up` or `down` (or unspecified if new), reflecting the last trade direction from the trade immediately prior.  If not determined, this property will be missing.
`pairs[].num_trades` | `pairs[].n` | Integer | Total number of individual trades on this pairing in the last rolling 24 hours

#### Websocket specifics:
Websocket returned property names are shortened from their REST equivalents as detailed above.
Websocket version will return a data packet every ledger close (approx every 4 seconds) and the returned data will vary depending if changes from the initial connecting request were detected.

When connecting to the websocket version of this endpoint, be aware that the server will return a `pairs` array of data for any property that has changed since the last `pairs` packet - this may not necessarily be the `l` (`last`) price that has changed.
It could be that the rolling 24 hours calculation has changed, for example, triggering the sending of a data packet to your connection.
Any property that has changed will trigger data for the whole request to be sent to you (that is, a change in any property of 1 pair will mean data for all your requested pairs will be sent, not just for the changed pair).




## `GET`: `/daily/tokens`
Returns a **list of tokens traded** on the XRP Ledger DEX at any time within the last rolling 24 hours, together with their key metrics.  Max returned items is 100, which in combination with the `by` parameter, allows getting the top 100 by volume, market cap, or trade count.

#### Example:
```
https://api.onthedex.live/public/v1/daily/tokens
```

#### GET parameters to specify:
Parameter | Specification | Description
--- | --- | ---
`by` | `volume` (traded volume in USD equivalent value), `market_cap` (market capitalization, in USD), or `trades` (number of trades) | Sorts the returned token list by the parameter specified, in descending order.  Default: `volume`
`min_trades` | Integer | Filter the tokens returned to include only those tokens that have traded the specified number of times or more in the last 24 hours.  Use to exclude illiquid tokens particularly when ordering `by`=`market_cap`.  Default: `100`.  To include all traded tokens, set to `1`.
`page` | Integer | Data page number to return.  Default is page `1`.
`per_page` | Integer | Maximum number of results to return for this page.  Default (and max) is `100`.



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
`tokens[].supply` | Float | Total number of tokens under supply obligation by the issuing account

#### Example returned data:
```json
{
    "tokens": [
        {
            "currency": "USD",
            "issuer": "rvYAfWj5gh67oV6fW32ZzP3Aw4Eubs59B",
            "last_trade_at": "2022-05-10T15:05:42.000Z",
            "market_cap": 32878521,
            "num_trades": 768,
            "price_mid_usd": 1.0055049538537304,
            "supply": 32698516.82373998,
            "token_name": "Bitstamp USD",
            "volume_token": 774648.164521166,
            "volume_usd": 776031.2081952752
        },
        {
            "currency": "SOLO",
            "issuer": "rsoLo2S1kiGeCcn6hCUXVrCpGMWLrRrLZz",
            "last_trade_at": "2022-05-10T15:14:52.000Z",
            "market_cap": 146978419,
            "num_trades": 2850,
            "price_mid_usd": 0.36781515699146133,
            "supply": 399598591.6977495,
            "token_name": "Sologenic",
            "volume_token": 1487728.1869581435,
            "volume_usd": 544250.4805697609
        },
        ...
    ]
}
```


## `GET`: `/daily/pairs`
Returns a **list of pairs (base / quote) traded** on the XRP Ledger DEX at any time within the last rolling 24 hours, together with their key metrics.

#### Example:
```
https://api.onthedex.live/public/v1/daily/pairs?token=CSC.rCSCManTZ8ME9EoLrSHHYKW8PPwWMgkwr
```

#### GET parameters to specify:
Parameter | Specification | Description
--- | --- | ---
`token` | String, optional | When not specified, returns complete list of traded pairs.  Note that this complete list includes inverted pairs (eg. TOKEN/USD.rhub and USD.rhub/TOKEN) with the same overall metrics for easier access to the pair you might want.  Be sure not to double-count such entries where relevant in your use case.  If `token` is specified as a concatenation of currency code and issuing account with a period `.` separator, returns only those pairs traded against the specified token.  Examples: `CSC.rCSCManTZ8ME9EoLrSHHYKW8PPwWMgkwr`, `XRP`.  If `token` is set to be `XRP`, tokens are always returned as the base currency with XRP as the quote currency.   If `token` is specified in conjuction with `quote` below, `token` is treated as the base currency request (including if set as XRP) and data will be returned in this configuration.
`quote` | String, optional | If specified, must be used with `token` parameter and will limit the return data to be for this pairing only (`token`/`quote`).  Examples: `SOLO.rsoLo2S1kiGeCcn6hCUXVrCpGMWLrRrLZz`, `XRP`.
`by` | `volume` (traded volume in USD equivalent value), `trades` (number of trades) | Sorts the returned pairs by the parameter specified, in descending order.  Default: `volume`
`page` | Integer | Data page number to return.  Default is page `1`.
`per_page` | Integer | Maximum number of results to return for this page.  Default (and max) is `100`.

#### Returned properties:
Property | Type | Description
--- | --- | ---
`pairs` | Array | List of tokens returned
`pairs[].base` | Object | Base currency specified with `currency` and `issuer` properties
`pairs[].base.currency` | String | Currency code of the base token
`pairs[].base.issuer` | String | r-Address of the base token issuer
`pairs[].quote` | String **or** Object | If quote currency is XRP, this contains the string `XRP`.  In all other cases, an object is returned with `currency` and `issuer` properties
`pairs[].quote.currency` | String | Currency code of the quote token
`pairs[].quote.issuer` | String | r-Address of the quote token issuer (not specified if quote = `XRP`)
`pairs[].volume_base` | Float | Volume of the base token traded this pair in the last rolling 24 hours
`pairs[].volume_quote` | Float | Volume of the quote token traded this pair in the last rolling 24 hours
`pairs[].volume_usd` | Float | Volume of USD equivalent value traded this pair in the last rolling 24 hours
`pairs[].price_mid` | Float | Current mid price on the order books of this pairing.  If no orders on both sides of the book for this pair, result is `null`
`pairs[].price_hi` | Float | The highest price achieved across all trades of this pair in the last rolling 24 hours
`pairs[].price_lo` | Float | The lowest price achieved across all trades of this pair in the last rolling 24 hours
`pairs[].price_hi_usd` | Float | The highest USD equivalent price achieved across all trades of this pair in the last rolling 24 hours
`pairs[].price_lo_usd` | Float | The lowest USD equivalent price achieved across all trades of this pair in the last rolling 24 hours
`pairs[].last` | Float | The price achieved on most recent trade of the pair
`pairs[].ago24` | Float | The price last traded from (rolling) 24 hours ago
`pairs[].pc24` | Float | The percentage change in price from (rolling) 24 hours ago to last traded
`pairs[].trend` | String | Either `up` or `down` (or unspecified if new), reflecting the last trade direction from the trade immediately prior




`pairs[].num_trades` | Integer | Total number of individual trades on this pair

#### Example returned data:
```json
{
   "pairs": [
      {
        "base":{
            "currency":"USD",
            "issuer":"rhub8VRN55s94qWKDv6jmDy1pUykJzF3wq"
        },
        "quote": "XRP",
        "price_hi": 3.2871288176530022,
        "price_lo": 2.747280225147826,
        "price_hi_usd": 1.0121361553828967,
        "price_lo_usd": 0.9893297283245092,
        "volume_base": 696691.0645208354,
        "volume_quote": 2032756.865624,
        "volume_usd": 697858.4758794114,
        "num_trades": 524,
        "price_mid": 3.233545692446336,
        "ago24": 2.865329512893983,
        "pc24": 13.27,
        "last": 3.245452115401085,
        "trend": "down"
      },
      ...
   ]
}
```



## `GET`/`POST`: `/aggregator`
Returns a **list of aggregator data** for all tokens traded on the XRP Ledger DEX at any time within the last rolling 24 hours, together with each token's pairings and associated key metrics.

#### GET Example:
```
GET https://api.onthedex.live/public/v1/aggregator
```

#### POST Example:
```
curl -d '{"tokens": [{"currency": "CSC", "issuer": "rCSCManTZ8ME9EoLrSHHYKW8PPwWMgkwr"}, "SOLO.rsoLo2S1kiGeCcn6hCUXVrCpGMWLrRrLZz"]}' -X POST https://api.onthedex.live/public/v1/aggregator
```

#### GET/POST parameters to specify:
Parameter | Specification | Description
--- | --- | ---
`token` | String, `GET` only | If specified, returns the aggregator just for the specified token.  Example: `SOLO.rsoLo2S1kiGeCcn6hCUXVrCpGMWLrRrLZz`
`tokens` | Array of Object or String, `POST` only | If specified, returns the aggregator for the specified token(s) only.  The array can contain tokens in a mixture of formats: objects with `currency` and `issuer` properties, and/or strings with period-separated currency and issuer.  Example: `[{"currency": "CSC", "issuer": "rCSCManTZ8ME9EoLrSHHYKW8PPwWMgkwr"}, "SOLO.rsoLo2S1kiGeCcn6hCUXVrCpGMWLrRrLZz"]`

#### Returned properties:
Property | Type | Description
--- | --- | ---
`tokens` | Array | List of tokens returned
`tokens[].currency` | String | Currency code of the base token
`tokens[].issuer` | String | r-Address of the base token issuer
`tokens[].name` | String | Official name of the token, if known
`tokens[].supply` | Float | Total number of tokens under supply obligation by the issuing account
`tokens[].market_cap` | Integer | Market capitalization of the token in USD based on current mid price and issued suppy of the token
`tokens[].dex` | Object | Information on the DEX specific trades for this token
`tokens[].dex.price` | Float | XRPL DEX current mid price in USD fiat equivalent for the token via most liquid pairing
`tokens[].dex.pc24` | Float | USD fiat equivalent price change in percent from the price 24 hours ago (rolling)
`tokens[].dex.fx24` | Float | USD fiat equivalent sum of all trades across all pairs for this token in the last rolling 24 hours
`tokens[].dex.sum24` | Float | Total volume traded across all pairs for this token in the last rolling 24 hours
`tokens[].dex.count24` | Integer | Number of trades across all pairs for this token in the last rolling 24 hours
`tokens[].dex.pairs` | Array | List of pair data traded against this token in the last rolling 24 hours
`tokens[].dex.pairs[].quote` | String | Currency code of the quote token
`tokens[].dex.pairs[].issuer` | String | r-Address of the quote token issuer (not specified if quote = `XRP`)
`tokens[].dex.pairs[].name` | String | Official name of the quote currency, if known
`tokens[].dex.pairs[].vol_token` | Number | Volume of token currency traded in the period against this quote currency
`tokens[].dex.pairs[].vol_quote` | Number | Volume of this quote currency traded in the period against the token currency
`tokens[].dex.pairs[].last` | Float | Price achieved on most recent trade of the pair
`tokens[].dex.pairs[].time` | Integer, UNIX timestamp (seconds) | Time of most recent trade of the pair
`tokens[].dex.pairs[].ask` | Float | Current ask (sell) price of the pair on the order books
`tokens[].dex.pairs[].bid` | Float | Current bid (buy) price of the pair on the order books
`tokens[].dex.pairs[].mid` | Float | Current mid price of the pair on the order books
`tokens[].dex.pairs[].spr` | Float | Current price spread between bid and ask prices
`tokens[].dex.pairs[].hi24` | Float | Highest traded price of the pair within the period
`tokens[].dex.pairs[].lo24` | Float | Lowest traded price of the pair within the period
`tokens[].dex.pairs[].ago24` | Float | Price last traded 24 hours ago (rolling)
`tokens[].dex.pairs[].pc24` | Float | Price change in percent from the price 24 hours ago (rolling)
`tokens[].dex.pairs[].fx` | Float | USD fiat equivalent sum of all trades of this pair in the last rolling 24 hours
`tokens[].dex.pairs[].count24` | Integer | Number of trades of this pair in the last rolling 24 hours



#### Example returned data:
```json
{
   "tokens": [
      {
         "currency": "USD",
         "issuer": "rvYAfWj5gh67oV6fW32ZzP3Aw4Eubs59B",
         "name": "Bitstamp USD",
         "supply": 66956924,
         "market_cap": 67369988,
         "dex": {
            "sum24": 3697128,
            "fx24": 3710229.4243013225,
            "price": 1.0061690928246487,
            "count24": 1049,
            "pairs": [
               {
                  "quote": "XRP",
                  "vol_token": 3696703,
                  "vol_quote": 9534078,
                  "time": 1653582001,
                  "last": 2.5152037780373253,
                  "fx": 3709805.121005436,
                  "bid": 2.5139844891,
                  "ask": 2.5206369411,
                  "mid": 2.5173107151,
                  "spr": 0.006652452,
                  "hi24": 2.6562030364266627,
                  "lo24": 2.380952380952381,
                  "ago24": 2.4876708046381366,
                  "pc24": 1.11,
                  "count24": 1045
               },
               {
                  "quote": "CNY",
                  "issuer": "rKiCet8SdvWxPXnAgYarFUXMh1zCPz432Y",
                  "name": "RippleFox CNY",
                  "vol_token": 181,
                  "vol_quote": 1205,
                  "time": 1653580582,
                  "last": 6.65999999999999,
                  "fx": 180.91095660379398,
                  "bid": 6.66,
                  "ask": 6.82221,
                  "mid": 6.741105,
                  "spr": 0.16221,
                  "hi24": 6.65999999999999,
                  "lo24": 6.65999999999999,
                  "ago24": 6.659999999999614,
                  "count24": 2
               },
               {
                  "quote": "BTC",
                  "issuer": "rchGBxcD1A1C2tdxF6papQYZ8kjRKMYcL",
                  "name": "GateHub Bitcoin",
                  "vol_token": 181,
                  "vol_quote": 0.00621774,
                  "time": 1653580582,
                  "last": 0.0000343005035829973,
                  "fx": 181.272593767661,
                  "bid": 0.000010011212558065033,
                  "ask": 0.00003398053545634613,
                  "mid": 0.000015465917070579048,
                  "spr": 0.00001419257292890581,
                  "hi24": 0.0000343005035829973,
                  "lo24": 0.0000343005035829973,
                  "ago24": 0.000028184003834661707,
                  "pc24": 17.83,
                  "count24": 1
               },
               {
                  "quote": "USD",
                  "issuer": "rhub8VRN55s94qWKDv6jmDy1pUykJzF3wq",
                  "name": "GateHub USD",
                  "vol_token": 62.12,
                  "vol_quote": 62.12,
                  "time": 1653579980,
                  "last": 1,
                  "fx": 62.119745515061,
                  "bid": 1,
                  "ask": 1.0102235143962908,
                  "mid": 1.0050857600782255,
                  "spr": 98.81372248693705,
                  "hi24": 1,
                  "lo24": 1,
                  "count24": 1
               }
            ],
            "pc24":0.62
         }
      },
      ...
      {
         "currency":"CSC",
         "issuer":"rCSCManTZ8ME9EoLrSHHYKW8PPwWMgkwr",
         "name":"CasinoCoin",
         "supply":64992453166,
         "market_cap":43291147,
         "dex":{
            "sum24":24083073,
            "fx24":15730.828657375625,
            "price":0.0006660949774231682,
            "count24":518,
            "pairs":[
               {
                  "flags":1,
                  "quote":"XRP",
                  "vol_token":24083073,
                  "vol_quote":39472,
                  "time":1653581760,
                  "last":0.0016393442622950817,
                  "fx":15730.828657375625,
                  "bid":0.0016393443,
                  "ask":0.0016936304,
                  "mid":0.0016664873,
                  "spr":0.0000542861,
                  "hi24":0.0017211531849988541,
                  "lo24":0.0015850121596770262,
                  "ago24":0.001585012125342759,
                  "pc24":3.43,
                  "count24":518
               }
            ],
            "pc24":4.34
         }
      },
      ...
   ]
}


```





## `GET`: `/ohlc`
Returns **Open/High/Low/Close price data** for a given token pairing for a given timeframe interval.

#### Example:
```
https://api.onthedex.live/public/v1/ohlc?base=CSC.rCSCManTZ8ME9EoLrSHHYKW8PPwWMgkwr&quote=XRP&bars=100&interval=60
```

#### GET parameters to specify:
Parameter | Specification | Description
--- | --- | ---
`base` | String | Concatenation of base currency code and issuing account with a period `.` separator.  Examples: `CSC.rCSCManTZ8ME9EoLrSHHYKW8PPwWMgkwr`, `XRP`
`quote` | String | Concatenation of quote currency code and issuing account with a period `.` separator.  Examples: `CSC.rCSCManTZ8ME9EoLrSHHYKW8PPwWMgkwr`, `XRP`
`interval` | `5`, `15`, `60`, `240`, `D`, or `W` | Interval timeframe for which OHLC data is required.  Integer values represent timeframe in minutes.  Ignored when `marker` is specified.
`tf` | `ISO` (optional) | Time format to return.  Default is UNIX timestamp in seconds since epoch.  Setting `ISO` returns string of ISO format time per bar instead.
`ending` | String or Integer (optional) | A string timestamp, or UNIX timestamp in seconds, representing the last time bar to return.  If not specified, current time is used.  Ignored when `marker` is specified.  Examples: `2021-12-25T14:00:00Z`, `2022-01-01`, `2022-03-15 12:00:00`, `1652054400`
`bars` | Integer | Number of bars of data to return, starting from the `ending` time and moving backwards for this number of bars.  Default varies depending on specified `tf`.  Maximum returned bars is 5000 per call.  Ignored when `marker` is specified.
`cf` | `yes` (optional) | Flag for requesting the carry-forward open price.  If set to `yes`, modifies the open `o` property returned for each time bar (see below)
`fx` | `USD` (optional) | If set, returns the OHLC data converted to forex equivalent (USD equivalent only presently) to show this pairing relative to USD
`marker` | String (optional) | If set and valid from a previous call, the next set of data backwards in time from the last call is returned.  Mandatory parameters when `marker` is specified are: `base`, `quote`.  Ignored parameters when `marker` is set are: `ending`, `interval`, `bars`.  Other parameters should be specified identical to the original request where relevant.

#### Returned properties:
Property | Type | Description
--- | --- | ---
`spec` | Object | The interpreted specification from the passed parameters; what the API determined to be your request
`spec.base` | String **or** Object | If base currency is XRP, this contains the string `XRP`.  In all other cases, and object is returned with `currency` and `issuer` properties
`spec.base.currency` | String | Currency code of the base token
`spec.base.issuer` | String | r-Address of the base token issuer
`spec.quote` | String **or** Object | If quote currency is XRP, this contains the string `XRP`.  In all other cases, an object is returned with `currency` and `issuer` properties
`spec.quote.currency` | String | Currency code of the quote token
`spec.quote.issuer` | String | r-Address of the quote token issuer
`spec.interval` | String | Interval timeframe
`spec.ending` | Timestamp or String | The passed `ending` parameter
`spec.bars` | Integer | The number of bars queried for.  Should be equal to the number of `data.ohlc` items returned.
`spec.cf` | String | The passed `cf` parameter
`spec.fx` | String | The passed `fx` parameter
`spec.tf` | String | The passed `tf` parameter
`data` | Object | The main returned data object
`data.marker` | String | The response includes a marker field when more data of the type requested in the original call exists. To retrieve the next block of data, call this path again with the `marker` field populated with this value.  If no `marker` property is returned, there is no more data to retrieve.  See `marker` parameter above.
`data.ohlc[]` | Array | List of OHLC price bar data in ascending time order
`data.ohlc[].t` | Integer or ISO Timestamp | UNIX Timestamp (default when `tf` not set) or ISO Timestamp (if `tf`=`ISO`) in GMT of the opening time of the price bar
`data.ohlc[].o` | Float | Opening trading price of the time bar.  If `cf` was set to `yes`, this property contains the closing price of the previous bar, and is distinct from the original pairing `o` price of this bar.  It permits the representation of this time bar opening at the same price the previous bar closed at, effectively ensuring there are no price 'gaps' in data bar-to-bar due to illiquid price jumps.
`data.ohlc[].h` | Float | Highest trading price within the time bar
`data.ohlc[].l` | Float | Lowest trading price within the time bar
`data.ohlc[].c` | Float | Closing trading price of the time bar
`data.ohlc[].vb` | Float | Volume of base currency traded within the time bar
`data.ohlc[].vq` | Float | Volume of quote currency traded within the time bar
`data.ohlc[].vusd` | Float | Volume of USD equivalent traded within the time bar.  Only returned if property `fx`=`USD`

#### Example returned data:
```json
{
    "spec": {
       "base": {
          "currency": "SOLO",
          "issuer": "rsoLo2S1kiGeCcn6hCUXVrCpGMWLrRrLZz"
       },
       "quote": "XRP",
       "interval": "60",
       "ending": "2021-11-30 12:00:00",
       "bars": 100,
       "cf": "yes",
       "tf": "ISO"
    },
    "data": {
       "ohlc": [
          {
             "t": "2021-11-26T09:00:00.000Z",
             "vb": 137921.15374666537,
             "vq": 455238.8346539995,
             "o": 3.2729339999096463,
             "h": 3.4467817896389317,
             "l": 3.200000000379838,
             "c": 3.2729339999096463
          },
          {
             "t": "2021-11-26T10:00:00.000Z",
             "vb": 97760.89569615627,
             "vq": 318993.3424850003,
             "o": 3.2729339999096463,
             "h": 3.390000000000026,
             "l": 3.182000000133979,
             "c": 3.295
          },
          {
             "t": "2021-11-26T11:00:00.000Z",
             "vb": 85772.03223426663,
             "vq": 287311.8863329999,
             "o": 3.295,
             "h": 3.4482824387894633,
             "l": 3.1800000048945862,
             "c": 3.3300000323603123
          },
          ...
       ]
    }
}
```

#### Difference between `cf` open and normal open price:
This first image below shows the effect of `cf` being unset, where the `o`pen price is unaffected by the previous bar values.  Where price gaps exist bar-to-bar, they will be shown as a result:
![Normal Open Prices](/images/example_normal_open_prices.png?raw=true "Normal Open Prices on illiquid token or interval, showing gaps")

This second image below shows the effect of specifying `cf`=`yes`, to close the gap between bars where there would otherwise be a gap.  The previous bar's close price is carried forward to be the current bar's open price:
![Adjusted Open Prices](/images/example_open_prices_with_cf_flag.png?raw=true "Adjusted Open Prices with carried forward open price")