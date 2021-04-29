The data structure of item in `klines_btcusdt_1m.json` is:
```json
[
  [
    1499040000000,      // Open time
    "0.01634790",       // Open
    "0.80000000",       // High
    "0.01575800",       // Low
    "0.01577100",       // Close
    "148976.11427815",  // Volume
    1499644799999,      // Close time
    "2434.19055334",    // Quote asset volume
    308,                // Number of trades
    "1756.87402397",    // Taker buy base asset volume
    "28.46694368",      // Taker buy quote asset volume
    "17928899.62484339" // Ignore.
  ]
]
```

Requirement:

- Generate different interval(3m, 5m, 30m, 1h, 2h, 4h, 1d, 3d, 1w) kline history data from `1m` interval data in `klines_btcusdt_1m.json`.
- Insert the kline history data to DB(MongoDB or Postgresql)
- Pls think about how to quickly finish this process.

**Currently you can only use `Open time`, `Open`, `High`, `Low`, `Close` and `Volume`, other fields can be ignored.**

Here is an example of generating `1m` to `3m` kline data:

- choose one item in the list:
```
[1619642640000,"54473.77000000","54622.34000000","54421.76000000","54589.13000000","24.68396300",1619642699999,"1344856.59530477",1835,"12.11248400","660055.96038677","0"]
```
- parse `Open time` and `Close time`
```
  Open time: 1619642640000 (2021-04-29 04:44:00)
  Close time: 1619642699999 (2021-04-29 04:44:59)
  `Close time` - `Open time` ~= 60s, which means this item is the current 1m kline data for 2021-04-29 04:44:00.
```

- If the `Open time` can be divided by the period, then we can generate a new kline data.

```
[1619642640000,"54473.77000000","54622.34000000","54421.76000000","54589.13000000","24.68396300",1619642699999,"1344856.59530477",1835,"12.11248400","660055.96038677","0"],
[1619642700000,"54589.13000000","54589.14000000","54401.00000000","54421.06000000","30.72695400",1619642759999,"1673757.44562674",1803,"15.33394500","835225.62981710","0"],
[1619642760000,"54425.75000000","54449.98000000","54321.74000000","54396.47000000","73.86902600",1619642819999,"4015998.23572650",1943,"37.78616700","2053766.11476467","0"],
[1619642820000,"54396.48000000","54480.56000000","54314.46000000","54318.90000000","36.70866100",1619642879999,"1996685.58852846",2365,"13.68051600","744378.70563816","0"],
[1619642880000,"54317.78000000","54333.42000000","54159.00000000","54304.27000000","159.73543500",1619642939999,"8666351.05074717",4218,"62.80211300","3407430.59759074","0"]
```

The first item's `Open time` is `2021-04-29 04:44:00`, the minute is `44`, and `44 % 3 != 0`, but `42 % 3 == 0` and `45 % 3 == 0`.

So the current `3m` kline data's `Open time` should be `2021-04-29 04:42:00` , and `Close time` should be `2021-04-29 04:44:59`, so you need to collect these items between this period.

and
```
    `Open` is the first item's `Open` of this collection, (54473.77)
    `High` is the highest `High` of this collection,
    `Low` is the lowest `Low` of this collection,
    `Close` is the last `Close` of this collection,
    `Volume` is the sum of `Volume` of this collection
```

- Pls know that the history data maybe not always consecutive.
- Pls verify if each `3m` kline data was created correctly.
