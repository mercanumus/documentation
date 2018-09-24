# MercaNumus Configuration

MercaNumus software is configured using JSON file stored in the same directory with the executable. Default name for the configuration file is _MercaNumus.json_ (another file can be specified using [command line parameters](#command-line-parameters)). Configuration file contains single JSON object with the following fields (sections): [General](#general), [Recorder](#recorder), [Player](#player), [Venues](#venues), [Assets](#assets), [Symbols](#symbols), and [Strategies](#strategies).

## General
Section name: **"general"**.

Section fields:

| Field | Values | Meaning |
| ----- | ------ | ------- |
| "mode" | "RECORDING" | sets recording market data system mode  ( see [Recorder](#recorder) ). Strategies in this mode are disabled |
| | "PLAYBACK" | strategy simulation using prerecorded market data ( see [Player](#player) ) |
| | "SIMULATION" | strategy simulation using live market data |
| | "TRADING" | live trading mode |

Example:
```json
"general" : {
    "mode" : "SIMULATION"		
}
```

## Recorder

Section name: **"recorder"**. This section and all of its fields are _optional_.

Section fields:

| Field | Values | Meaning |
| ----- | ------ | ------- |
| "recordAnyway" | true / **false** | enables/disables recording during SIMULATION and TRADING system modes |
| "fileName" | " " | market data file name, relative to the executable path. Default name is automatically generated using the following template: _"MarketData-YYYY-MM-DD-HH-MM-SS.log"_ |

Example:
```json
"recorder" : {
    "recordAnyway" : true
}
```

## Player

Section name: **"player"**. This section is _required_ in the PLAYBACK mode.

Section fields:

| Field | Values | Meaning |
| ----- | ------ | ------- |
| "sourceFile" | " " | specifies file relative to the executable path where market data to be used for playback is stored |

Example:
```json
"player" : {
    "sourceFile" : "MarketData-2018-09-18-14-47-41.log"
}
``` 

## Venues

Section name: **"venues"**. This section is an _array_ of venue configuration objects. The following fields of the configuration objects are common for all venues:

| Field | Values | Meaning |
| ----- | ------ | ------- |
| "name" | " " | user specified unique name of the trading venue. This name is used for captions and referencing to this venue |
| "type" | " " | specifies system internal type of venue. For example: "binance" for the [Binance](#binance) configuration (see below) |
| "showTiming" | true / **false** | enables/disables reporting of the average market data processing time to console (in microseconds) |
| "timeToMarket" | **1000** | time (in milliseconds) required for order to reach the venue, which is used for simulations. |

### Binance

The following configuration fields are used to configure [Binance](http://www.binance.com) venue:

| Field | Values | Meaning |
| ----- | ------ | ------- |
| "apiKey" | " " | API key, required for live trading (TRADING mode) on Binance |
| "secretKey" | " " | secrect kye used to sign account specific requestes to Binance (TRADING mode) |
| "hostWS" | "wss://stream.binance.com:9443" | base address of the web-socket endpoint. More details in [Binance API documentation](https://github.com/binance-exchange/binance-official-api-docs/blob/master/web-socket-streams.md) |
| "hostREST" | "https://api.binance.com" | base address for the REST API |
| "streams" | [ "trade", "depth", "kline_1m", ... ] | array of names of the market data streams

Example:
```json
"venues"  	: [
    {
        "name"          : "Binance",
        "type"          : "binance",
        "showTiming"    : false,
        "timeToMarket"	: 1000,
        "apiKey"        : "",
        "secretKey"     : "",
        "hostWS"        : "wss://stream.binance.com:9443",
        "hostREST"      : "https://api.binance.com",
        "streams"       : [ "trade", "depth" ]
    }
]
``` 


## Assets

Section name: **"assets"**. This section is an _array_ of the assets used for  trading. Each element of this array is also an array, containing at least one element - asset name. Other elements - are the array duplets of venue name and corresponding available balance:

```json
"assets" : [
    [ "BTC",    ["Binance", 0.01]   ],
    [ "ETH",    ["Binance", 0]      ],
    [ "USDT",   ["Binance", 10000]  ]
]
``` 

Balance values are used in simulations (SIMULATION and PLAYBACK modes) and overriden by actual values in TRADING mode.


## Symbols

Section name: **"symbols"**. This section is an _array_, containing array duplets of assets that comprise the traded symbol. First element is the _base_ asset of the pair, and the second is the _quote_ asset. The following is the example of the symbols definition:

```json
"symbols" : [
    [ "BTC", "USDT" ],
    [ "ETH", "USDT" ]
]
``` 
Here first symbol is "BTC" traded for the "USDT". When symbol is referenced, names of its assets are dash-separated, e.g. "BTC-USDT".

## Strategies

Section name: **"strategies"**. This section is an _array_ of strategy configuration objects, having the following fields:

| Field | Values | Meaning |
| ----- | ------ | ------- |
| "name" | " " | name of the strategy used for captions and referencing |
| "symbols" | [ "BTC-USDT", ... ] | array of symbols that can be observed and traded by the strategy |
| "venues" | [ "Binance", ... ] | array of venues where the strategy will be trading |
| "bookDepth" | **10** | order books depth for each venue that will be accessible by the strategy, as well as visible at GUI |
| "maxLoss" | _number_ | value of combined losses for all symbols in terms of quote asset at which the strategy trading will be stopped. For example: **-10** |
| "guiUpdate" | **500** | delay in millisecnds between GUI updates |
| "script" | "JavaScript" / "Python" | name of the scripting language that is used to implement strategy logic |
| "path" | "" | relative to executable path, where strategy files are stored |
| "srcInit" | " " | name of the source file, containing initialization part of the strategy. This part will be executed only _once_, when strategy is initialized. This field is _optional_ |
| "srcMain" | " " | name of the source file, containing main body of the strategy, which will be executed upon any market event |
| "srcDone" | " " | name of the source file, containing finalization part of the strategy. This part will be executed when the system stopped and is _optional_ |
| "logEnabled" | true / **false** | enalbes/disables writing to strategy logging file |
| "startActive" | true / **false** | specifies whether the strategy starts to trade immediately after system startup |
| "showTiming" | true / **false** | enables/disables reporting of average strategy execution time to console |

Example:

```json
"strategies" : [
    {
        "name"          : "JS Demo",
        "symbols"       : [ "BTC-USDT" ],
        "venues"        : [ "Binance" ],
        "bookDepth"     : 10,
        "maxLoss"       : 100,
        "guiUpdate"     : 500,
        "script"        : "JavaScript",
        "path"      	: "/strats/JSDemo/",
        "srcInit"       : "init.js",
        "srcMain"       : "main.js",
        "srcDone"       : "done.js",
        "logEnabled"    : false,
        "startActive"   : true,
        "showTiming"    : false
    },
    {
        "name"          : "Python Demo",
        "symbols"       : [ "ETH-USDT" ],
        "venues"        : [ "Binance" ],
        "bookDepth"     : 10,
        "maxLoss"       : 150,
        "script"        : "Python",
        "path"          : "/strats/PythonDemo/",
        "srcInit"       : "init.py",
        "srcMain"       : "main.py",
        "srcDone"       : "done.py",
        "logEnabled"    : false,
        "startActive"   : true,
        "showTiming"    : false
    }
]
```

## Command Line Parameters

MercaNumus executable can be launched with command line parameters in the following format:
```sh
java -jar "MercaNumus.jar" [-command  parameter, ...]
```

Possible values for commands and its parameters explained in the following table:

| Command | Parameter |
| ----- | ------ |
| -config | name of the configuration file |
| -c | same as "-config" |
| -play | name of the market data file for playback |
| -p | same as "-play" |