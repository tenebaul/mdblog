# metrics

## MetricsType

- ``Gauge``:  [geɪdʒ] 仪表盘，具有刷新性质。时序数据库收到``Gauge``类型的数据，会做出``刷新/覆盖``的行为。

- ``Counter``: 计数器，具有累加性质。时序数据库收到``Counter``类型的数据，会做出``累加``的行为。

- ``Histogram``: 分布。前面两个（``Gauge``和``Counter``）都是考量一个数据，而``Histogram``是考量一组数据，只不过不是挨个地展示出来，而是用数学的方式，以分布的方式展示它们。

- ``Meter``：单位时间的数据状况。前面都是仅从数据层面考量，而没有考虑时间因素，而``Meter``纳入了时间。比如我们的``TPS``系统吞吐量。

- ``Timer``: A timer is basically a combination of Histogram + Meter.

### Gauge

``` json
{
  "type"  : "gauge",
  "value" : value // any json value
}
```

### Counter

``` json
{
  "type"  : "counter",
  "count" : 1 // number
}
```

### Histogram

``` json
{
  "type"   : "histogram",
  "count"  : 1 // long
  "min"    : 1 // long
  "max"    : 1 // long
  "mean"   : 1.0 // double
  "stddev" : 1.0 // double
  "median" : 1.0 // double
  "75%"    : 1.0 // double
  "95%"    : 1.0 // double
  "98%"    : 1.0 // double
  "99%"    : 1.0 // double
  "99.9%"  : 1.0 // double
}
```

### Meter

``` json
{
  "type"              : "meter",
  "count"             : 1 // long
  "meanRate"          : 1.0 // double
  "oneMinuteRate"     : 1.0 // double
  "fiveMinuteRate"    : 1.0 // double
  "fifteenMinuteRate" : 1.0 // double
  "rate"              : "events/second" // string representing rate
}

```

### Timer

``` json
{
  "type": "timer",

  // histogram data
  "count"  : 1 // long
  "min"    : 1 // long
  "max"    : 1 // long
  "mean"   : 1.0 // double
  "stddev" : 1.0 // double
  "median" : 1.0 // double
  "75%"    : 1.0 // double
  "95%"    : 1.0 // double
  "98%"    : 1.0 // double
  "99%"    : 1.0 // double
  "99.9%"  : 1.0 // double

  // meter data
  "meanRate"          : 1.0 // double
  "oneMinuteRate"     : 1.0 // double
  "fiveMinuteRate"    : 1.0 // double
  "fifteenMinuteRate" : 1.0 // double
  "rate"              : "events/second" // string representing rate
}

```


## 工具链

- 三组合：``influxdb`` + ``grafana``
