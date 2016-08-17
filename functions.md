使用InfluxQL functions 来聚合、查询、转化数据。

| **AggregationsSelectorsTransformations** |  |  |
| --- | --- | --- |
| [COUNT\(\)](https://github.com/influxdata/docs.influxdata.com/blob/master/influxdb/v0.13/query_language/functions/#count) | [BOTTOM\(\)](https://github.com/influxdata/docs.influxdata.com/blob/master/influxdb/v0.13/query_language/functions/#bottom) | [CEILING\(\)](https://github.com/influxdata/docs.influxdata.com/blob/master/influxdb/v0.13/query_language/functions/#ceiling) |
| [DISTINCT\(\)](https://github.com/influxdata/docs.influxdata.com/blob/master/influxdb/v0.13/query_language/functions/#distinct) | [FIRST\(\)](https://github.com/influxdata/docs.influxdata.com/blob/master/influxdb/v0.13/query_language/functions/#first) | [DERIVATIVE\(\)](https://github.com/influxdata/docs.influxdata.com/blob/master/influxdb/v0.13/query_language/functions/#derivative) |
| [INTEGRAL\(\)](https://github.com/influxdata/docs.influxdata.com/blob/master/influxdb/v0.13/query_language/functions/#integral) | [LAST\(\)](https://github.com/influxdata/docs.influxdata.com/blob/master/influxdb/v0.13/query_language/functions/#last) | [DIFFERENCE\(\)](https://github.com/influxdata/docs.influxdata.com/blob/master/influxdb/v0.13/query_language/functions/#difference) |
| [MEAN\(\)](https://github.com/influxdata/docs.influxdata.com/blob/master/influxdb/v0.13/query_language/functions/#mean) | [MAX\(\)](https://github.com/influxdata/docs.influxdata.com/blob/master/influxdb/v0.13/query_language/functions/#max) | [ELAPSED\(\)](https://github.com/influxdata/docs.influxdata.com/blob/master/influxdb/v0.13/query_language/functions/#elapsed) |
| [MEDIAN\(\)](https://github.com/influxdata/docs.influxdata.com/blob/master/influxdb/v0.13/query_language/functions/#median) | [MIN\(\)](https://github.com/influxdata/docs.influxdata.com/blob/master/influxdb/v0.13/query_language/functions/#min) | [FLOOR\(\)](https://github.com/influxdata/docs.influxdata.com/blob/master/influxdb/v0.13/query_language/functions/#floor) |
| [SPREAD\(\)](https://github.com/influxdata/docs.influxdata.com/blob/master/influxdb/v0.13/query_language/functions/#spread) | [PERCENTILE\(\)](https://github.com/influxdata/docs.influxdata.com/blob/master/influxdb/v0.13/query_language/functions/#percentile) | [HISTOGRAM\(\)](https://github.com/influxdata/docs.influxdata.com/blob/master/influxdb/v0.13/query_language/functions/#histogram) |
| [SUM\(\)](https://github.com/influxdata/docs.influxdata.com/blob/master/influxdb/v0.13/query_language/functions/#sum) | [TOP\(\)](https://github.com/influxdata/docs.influxdata.com/blob/master/influxdb/v0.13/query_language/functions/#top) | [MOVING\_AVERAGE\(\)](https://github.com/influxdata/docs.influxdata.com/blob/master/influxdb/v0.13/query_language/functions/#moving-average) |
|  |  | [NON\_NEGATIVE\_DERIVATIVE\(\)](https://github.com/influxdata/docs.influxdata.com/blob/master/influxdb/v0.13/query_language/functions/#non-negative-derivative) |
|  |  | [STDDEV\(\)](https://github.com/influxdata/docs.influxdata.com/blob/master/influxdb/v0.13/query_language/functions/#stddev) |

Useful InfluxQL for functions:

* [Include multiple functions in a single query](https://github.com/influxdata/docs.influxdata.com/blob/master/influxdb/v0.13/query_language/functions/#include-multiple-functions-in-a-single-query)
* [Change the value reported for intervals with no data with ](https://github.com/influxdata/docs.influxdata.com/blob/master/influxdb/v0.13/query_language/functions/#change-the-value-reported-for-intervals-with-no-data-with-fill)`fill()`
* [Rename the output column's title with ](https://github.com/influxdata/docs.influxdata.com/blob/master/influxdb/v0.13/query_language/functions/#rename-the-output-column-s-title-with-as)`AS`

The examples below query data using [InfluxDB's Command Line Interface \(CLI\)](https://github.com/influxdata/docs.influxdata.com/blob/master/influxdb/v0.13/tools/shell). See the [Querying Data](https://github.com/influxdata/docs.influxdata.com/blob/master/influxdb/v0.13/guides/querying_data) guide for how to query data directly using the HTTP API.

**Sample data**

这篇文档中使用的数据和 [Data Exploration](/data-exploration.md) 一样。可以在 [Sample Data](https://github.com/influxdata/docs.influxdata.com/blob/master/influxdb/v0.13/sample_data/data_download) 页面下载到。

# **Aggregations**

## **COUNT\(\)**

返回单个field中不为null的值的数量

```
SELECT COUNT(<field_key>) FROM <measurement_name> [WHERE <stuff>] [GROUP BY <stuff>]
```

Examples:

* 查询 `water_level` 不为null的数量

```
> SELECT COUNT(water_level) FROM h2o_feet
```

CLI response:

```
name: h2o_feet
--------------
time                           count
1970-01-01T00:00:00Z     15258
```

> **Note:** 在没有指定查询时间范围下限的情况下，聚合函数返回的时间戳为 epoch 0 \(`1970-01-01T00:00:00Z`\) 。如果指定，则为时间范围下限。

* 以4d为时间间隔，查询`water_level` 字段的非null数量：

```
> SELECT COUNT(water_level) FROM h2o_feet WHERE time >= '2015-08-18T00:00:00Z' AND time < '2015-09-18T17:00:00Z' GROUP BY time(4d)
```

CLI response:

```
name: h2o_feet
--------------
time                           count
2015-08-17T00:00:00Z     1440
2015-08-21T00:00:00Z     1920
2015-08-25T00:00:00Z     1920
2015-08-29T00:00:00Z     1920
2015-09-02T00:00:00Z     1915
2015-09-06T00:00:00Z     1920
2015-09-10T00:00:00Z     1920
2015-09-14T00:00:00Z     1920
2015-09-18T00:00:00Z     335
```

> #### `COUNT()`** and controlling the values reported for intervals with no data**
> 
> 其他的 InfluxQL functions 在interval内没有数据时返回 `null` ，并且通过追加 `fill(<stuff>)` 并用 `<stuff>` 替换`null`  。然而 `COUNT()`在interval内没有数据时，返回为 `0` ，所以使用在 `COUNT()` 时使用 `fill(<stuff>)` 来讲 `0` 替换 为`<stuff>。`
> 
> Example: 使用 `fill(none)` 取消 intervals 包含 `0` 的数据
> 
> `COUNT()` without `fill(none)`:
> 
> ```
> SELECT COUNT(water_level) FROM h2o_feet WHERE location = 'santa_monica' AND time >= '2015-09-18T21:41:00Z' AND time <= '2015-09-18T22:41:00Z' GROUP BY time(30m)
> name: h2o_feet
> --------------
> time                           count
> 2015-09-18T21:30:00Z     1
> 2015-09-18T22:00:00Z     0
> 2015-09-18T22:30:00Z     0
> ```
> 
> `COUNT()` with `fill(none)`:
> 
> ```
> SELECT COUNT(water_level) FROM h2o_feet WHERE location = 'santa_monica' AND time >= '2015-09-18T21:41:00Z' AND time <= '2015-09-18T22:41:00Z' GROUP BY time(30m) fill(none)
> name: h2o_feet
> --------------
> time                           count
> 2015-09-18T21:30:00Z     1
> ```
> 
> 更多关于 `fill()`的内容，见 [Data Exploration](/data-exploration.md).

## **DISTINCT\(\)**

返回单个field的唯一值 [field](https://github.com/influxdata/docs.influxdata.com/blob/master/influxdb/v0.13/concepts/glossary/#field)

```
SELECT DISTINCT(<field_key>) FROM <measurement_name> [WHERE <stuff>] [GROUP BY <stuff>]
```

Examples:

* 查询 `level description` field的唯一值：

```
> SELECT DISTINCT("level description") FROM h2o_feet
```

CLI response:

```
name: h2o_feet
--------------
time                           distinct
1970-01-01T00:00:00Z     between 6 and 9 feet
1970-01-01T00:00:00Z     below 3 feet
1970-01-01T00:00:00Z     between 3 and 6 feet
1970-01-01T00:00:00Z     at or greater than 9 feet
```

返回结果表示了 `level description` 有4个不一样的值。

> **Note:** 聚合函数的时间戳返回为 epoch 0 \(`1970-01-01T00:00:00Z`\) ，仅在指定时间范围下限的返回下限。

* 根据`location` tag分组，查询`level description` field 的唯一值：

```
> SELECT DISTINCT("level description") FROM h2o_feet GROUP BY location
```

CLI response:

```
name: h2o_feet
tags: location=coyote_creek
time                            distinct
----                            --------
1970-01-01T00:00:00Z      between 6 and 9 feet
1970-01-01T00:00:00Z      between 3 and 6 feet
1970-01-01T00:00:00Z      below 3 feet
1970-01-01T00:00:00Z      at or greater than 9 feet


name: h2o_feet
tags: location=santa_monica
time                            distinct
----                            --------
1970-01-01T00:00:00Z      below 3 feet
1970-01-01T00:00:00Z      between 3 and 6 feet
1970-01-01T00:00:00Z      between 6 and 9 feet
```

* 在`COUNT()` 中嵌套 `DISTINCT()` 来查询`level description` 根据`location` tag分组后的数量：

```
> SELECT COUNT(DISTINCT("level description")) FROM h2o_feet GROUP BY location
```

CLI response:

```
name: h2o_feet
tags: location = coyote_creek
time                           count
----                           -----
1970-01-01T00:00:00Z     4

name: h2o_feet
tags: location = santa_monica
time                           count
----                           -----
1970-01-01T00:00:00Z     3
```

## **INTEGRAL\(\)**

`INTEGRAL()` is not yet functional.

See GitHub Issue [\#5930](https://github.com/influxdata/influxdb/issues/5930) for more information.

## **MEAN\(\)**

返回单个field的平均值，该字段必须是int64 或 float64.

```
SELECT MEAN(<field_key>) FROM <measurement_name> [WHERE <stuff>] [GROUP BY <stuff>]
```

Examples:

* 计算 `water_level` field的平均值：

```
> SELECT MEAN(water_level) FROM h2o_feet
```

CLI response:

```
name: h2o_feet
--------------
time                           mean
1970-01-01T00:00:00Z     4.286791371454075
```

> **Notes:**
> 
> * Executing `mean()` on the same set of float64 points may yield slightly different results. InfluxDB does not sort points before it applies the function which results in those small discrepancies.

* 4d为interval，计算 `water_level` 的平均值：

```
> SELECT MEAN(water_level) FROM h2o_feet WHERE time >= '2015-08-18T00:00:00Z' AND time < '2015-09-18T17:00:00Z' GROUP BY time(4d)
```

CLI response:

```
name: h2o_feet
--------------
time                           mean
2015-08-17T00:00:00Z     4.322029861111125
2015-08-21T00:00:00Z     4.251395512375667
2015-08-25T00:00:00Z     4.285036458333324
2015-08-29T00:00:00Z     4.469495801899061
2015-09-02T00:00:00Z     4.382785378590083
2015-09-06T00:00:00Z     4.28849666349042
2015-09-10T00:00:00Z     4.658127604166656
2015-09-14T00:00:00Z     4.763504687500006
2015-09-18T00:00:00Z     4.232829850746268
```

## **MEDIAN\(\)**

返回单个字段的中间值，字段类型必须为 int64 或 float64。

```
SELECT MEDIAN(<field_key>) FROM <measurement_name> [WHERE <stuff>] [GROUP BY <stuff>]
```

> **Note:** `MEDIAN()` 基本和 `PERCENTILE(field_key, 50)` 一致，除了存在两个中间值时 `MEDIAN()` 返回它们的平均值。

Examples:

* 查询 `water_level`字段的中间值：

```
> SELECT MEDIAN(water_level) from h2o_feet
```

CLI response:

```
name: h2o_feet
--------------
time                           median
1970-01-01T00:00:00Z     4.124
```

* Select the median value of `water_level` 查询  2015-08-18T00:00:00Z 和 2015-08-18T00:36:00Z时间范围内，根据 `location` tag 分组后`water_level`的中间值

```
> SELECT MEDIAN(water_level) FROM h2o_feet WHERE time >= '2015-08-18T00:00:00Z' AND time < '2015-08-18T00:36:00Z' GROUP BY location
```

CLI response:

```
name: h2o_feet
tags: location = coyote_creek
time                           median
----                           ------
2015-08-18T00:00:00Z     7.8245

name: h2o_feet
tags: location = santa_monica
time                           median
----                           ------
2015-08-18T00:00:00Z     2.0575
```

## **SPREAD\(\)**

返回字段的最大值和最小值的差值，字段必须是 int64 或 float64.

```
SELECT SPREAD(<field_key>) FROM <measurement_name> [WHERE <stuff>] [GROUP BY <stuff>]
```

Examples:

* 计算 `water_level` field 最大最小的差值：

```
> SELECT SPREAD(water_level) FROM h2o_feet
```

CLI response:

```
name: h2o_feet
--------------
time                            spread
1970-01-01T00:00:00Z      10.574

```

> **Notes:**
> 
> * Executing `spread()` on the same set of float64 points may yield slightly different results. InfluxDB does not sort points before it applies the function which results in those small discrepancies.

* 以30min作为interval，计算指定时间区间内每个时间段`water_level`  的最大最小差值：

```
> SELECT SPREAD(water_level) FROM h2o_feet WHERE location = 'santa_monica' AND time >= '2015-09-18T17:00:00Z' AND time < '2015-09-18T20:30:00Z' GROUP BY time(30m)
```

CLI response:

```
name: h2o_feet
--------------
time                            spread
2015-09-18T17:00:00Z      0.16699999999999982
2015-09-18T17:30:00Z      0.5469999999999997
2015-09-18T18:00:00Z      0.47499999999999964
2015-09-18T18:30:00Z      0.2560000000000002
2015-09-18T19:00:00Z      0.23899999999999988
2015-09-18T19:30:00Z      0.1609999999999996
2015-09-18T20:00:00Z      0.16800000000000015

```

## **SUM\(\)**

返回字段的和，字段类型必须是 int64 或 float64.

```
SELECT SUM(<field_key>) FROM <measurement_name> [WHERE <stuff>] [GROUP BY <stuff>]
```

Examples:

* 计算 `water_level` 的总和：

```
> SELECT SUM(water_level) FROM h2o_feet
```

CLI response:

```
name: h2o_feet
--------------
time                           sum
1970-01-01T00:00:00Z     67777.66900000002
```

* 以5d为interval，计算 `water_level` field 的总和：

```
> SELECT SUM(water_level) FROM h2o_feet WHERE time >= '2015-08-18T00:00:00Z' AND time < '2015-09-18T17:00:00Z' GROUP BY time(5d)
```

CLI response:

```
--------------
time                           sum
2015-08-18T00:00:00Z     10334.908999999983
2015-08-23T00:00:00Z     10113.356999999995
2015-08-28T00:00:00Z     10663.683000000006
2015-09-02T00:00:00Z     10451.321
2015-09-07T00:00:00Z     10871.817999999994
2015-09-12T00:00:00Z     11459.00099999999
2015-09-17T00:00:00Z     3627.762000000003
```

# **Selectors**

## **BOTTOM\(\)**

返回单个field的最小的 `N` ，字段类型必须是 int64 或 float64.

```
SELECT BOTTOM(<field_key>[,<tag_keys>],<N>)[,<tag_keys>] FROM <measurement_name> [WHERE <stuff>] [GROUP BY <stuff>]
```

Examples:

* 查询 `water_level`最小的3个值：

```
> SELECT BOTTOM(water_level,3) FROM h2o_feet
```

CLI response:

```
name: h2o_feet
--------------
time                           bottom
2015-08-29T14:30:00Z     -0.61
2015-08-29T14:36:00Z     -0.591
2015-08-30T15:18:00Z     -0.594
```

* 查询`water_level` 最小的3个值，并且返回相关的 `location` tag ：

```
> SELECT BOTTOM(water_level,3),location FROM h2o_feet
```

```
name: h2o_feet
--------------
time                     bottom    location
2015-08-29T14:30:00Z     -0.61    coyote_creek
2015-08-29T14:36:00Z     -0.591  coyote_creek
2015-08-30T15:18:00Z     -0.594  coyote_creek
```

* 在每个 `location`中查询最小的`water_level`值

```
> SELECT BOTTOM(water_level,location,2) FROM h2o_feet
```

CLI response:

```
name: h2o_feet
--------------
time                           bottom    location
2015-08-29T10:36:00Z     -0.243  santa_monica
2015-08-29T14:30:00Z     -0.61    coyote_creek
```

返回结果展示了在每个`location` \(`santa_monica` and`coyote_creek`\)中的最小值。

> **Note:** 使用 `SELECT BOTTOM(<field_key>,<tag_key>,<N>)`,查询语法时，如果 tag 有 `X` 个不同 values，则返回 `N` 或者 `X` 个point，看起来就像每个point都有唯一的tag值。为了演示，看下面的例子， `N` = `3` 和 `N` = `1`.
> 
> * `N` = `3`
> 
> ```
> SELECT BOTTOM(water_level,location,3) FROM h2o_feet
> ```
> 
> CLI response:
> 
> ```
> name: h2o_feet
> --------------
> time                           bottom    location
> 2015-08-29T10:36:00Z     -0.243  santa_monica
> 2015-08-29T14:30:00Z     -0.61    coyote_creek
> 
> ```
> 
> 因为 `location` tag 仅有2个值 \(`santa_monica` and`coyote_creek`\)，InfluxDB 返回了两个值而不是3个
> 
> * `N` = `1`
> 
> ```
> SELECT BOTTOM(water_level,location,1) FROM h2o_feet
> ```
> 
> CLI response:
> 
> ```
> name: h2o_feet
> --------------
> time                           bottom    location
> 2015-08-29T14:30:00Z     -0.61    coyote_creek
> ```
> 
> InfluxDB 比较了`location` tag中的每一个最小值，通过比较，返回了更小的`water_level`

* 查询2015-08-18T04:00:00Z和2015-08-18T04:24:00Z之前每个`location`的最小的两个值：

```
> SELECT BOTTOM(water_level,2) FROM h2o_feet WHERE time >= '2015-08-18T04:00:00Z' AND time < '2015-08-18T04:24:00Z' GROUP BY location
```

CLI response:

```
name: h2o_feet
tags: location=coyote_creek
time                           bottom
----                           ------
2015-08-18T04:12:00Z     2.717
2015-08-18T04:18:00Z     2.625


name: h2o_feet
tags: location=santa_monica
time                           bottom
----                           ------
2015-08-18T04:00:00Z     3.911
2015-08-18T04:06:00Z     4.055
```

* 查询`2015-08-18T04:00:00Z`和`2015-08-18T04:24:00Z` `water_level`上最小的两个值

```
> SELECT BOTTOM(water_level,2) FROM h2o_feet WHERE time >= '2015-08-18T04:00:00Z' AND time < '2015-08-18T04:24:00Z' AND location = 'santa_monica'
```

CLI response:

```
name: h2o_feet
--------------
time                           bottom
2015-08-18T04:00:00Z     3.911
2015-08-18T04:06:00Z     4.055
```

注意，在返回结果中，`water_level＝4.005`在`2015-08-18T04:06:00Z` 和 `2015-08-18T04:12:00Z`存在，相同情况下，InfluxDB返回更小的时间戳

## **FIRST\(\)**

根据时间戳，返回单个字段最早的值。

```
SELECT FIRST(<field_key>)[,<tag_key(s)>] FROM <measurement_name> [WHERE <stuff>] [GROUP BY <stuff>]
```

Examples:

* 查询`location` = `santa_monica`中 field `water_level` 最早的值：

```
> SELECT FIRST(water_level) FROM h2o_feet WHERE location = 'santa_monica'
```

CLI response:

```
name: h2o_feet
--------------
time                     first
2015-08-18T00:00:00Z     2.064
```

* 查询`2015-08-18T00:42:00Z` 和 `2015-08-18T00:54:00Z` 之间 field `water_level` 最早的值，同时显示`location` tag:

```
> SELECT FIRST(water_level),location FROM h2o_feet WHERE time >= '2015-08-18T00:42:00Z' and time <= '2015-08-18T00:54:00Z'
```

CLI response:

```
name: h2o_feet
--------------
time                           first     location
2015-08-18T00:42:00Z     7.234   coyote_creek
```

* 根据`location` tag分组，查询field `water_level` 最早的值：

```
> SELECT FIRST(water_level) FROM h2o_feet GROUP BY location
```

CLI response:

```
name: h2o_feet
tags: location = coyote_creek
time                           first
----                           -----
2015-08-18T00:00:00Z     8.12

name: h2o_feet
tags: location = santa_monica
time                           first
----                           -----
2015-08-18T00:00:00Z     2.064
```

## **LAST\(\)**

根据时间戳，返回单个字段最近的值。

```
SELECT LAST(<field_key>)[,<tag_key(s)>] FROM <measurement_name> [WHERE <stuff>] [GROUP BY <stuff>]
```

Examples:

* 查询 `location` ＝ `santa_monica`中field `water_level`最新的值：

```
> SELECT LAST(water_level) FROM h2o_feet WHERE location = 'santa_monica'
```

CLI response:

```
name: h2o_feet
--------------
time                           last
2015-09-18T21:42:00Z     4.938
```

* 查询 `2015-08-18T00:42:00Z` 和 `2015-08-18T00:54:00Z`之间最新的field `water_level`值，并且显示 `location` tag:

```
> SELECT LAST(water_level),location FROM h2o_feet WHERE time >= '2015-08-18T00:42:00Z' and time <= '2015-08-18T00:54:00Z'
```

CLI response:

```
name: h2o_feet
--------------
time                           last   location
2015-08-18T00:54:00Z     6.982   coyote_creek

```

* 根据`location` tag查询分组，查询最新的field `water_level` 值：

```
> SELECT LAST(water_level) FROM h2o_feet GROUP BY location
```

CLI response:

```
name: h2o_feet
tags: location = coyote_creek
time                           last
----                           ----
2015-09-18T16:24:00Z     3.235

name: h2o_feet
tags: location = santa_monica
time                           last
----                           ----
2015-09-18T21:42:00Z     4.938
```

> **Note:** `LAST()` 不会返回 `now()` 之后的point，除非 `WHERE` 字句指定了时间范围。参见 [Frequently Encountered Issues](https://github.com/influxdata/docs.influxdata.com/blob/master/influxdb/v0.13/troubleshooting/frequently_encountered_issues/#querying-after-now) 如何查询 `now()`之后的数据。

## **MAX\(\)**

返回单个字段的最大值，字段类型必须是int64、 float64 或 boolean。

```
SELECT MAX(<field_key>)[,<tag_key(s)>] FROM <measurement_name> [WHERE <stuff>] [GROUP BY <stuff>]
```

Examples:

* 查询 measurement `h2o_feet`中的最大 `water_level` ：

```
> SELECT MAX(water_level) FROM h2o_feet
```

CLI response:

```
name: h2o_feet
--------------
time                      max
2015-08-29T07:24:00Z     9.964
```

* 查询 measurement `h2o_feet` 中最大`water_level`并显示相关的`location` tag：

```
> SELECT MAX(water_level),location FROM h2o_feet
```

CLI response:

```
name: h2o_feet
--------------
time                           max     location
2015-08-29T07:24:00Z     9.964   coyote_creek

```

* 根据12min interval和`location` tag 分组，查询 measurement `h2o_feet` 中的`water_level`在`2015-08-18T00:00:00Z`和`2015-08-18T00:54:00Z`之间的最大值：

```
> SELECT MAX(water_level) FROM h2o_feet WHERE time >= '2015-08-18T00:00:00Z' AND time < '2015-08-18T00:54:00Z' GROUP BY time(12m), location
```

CLI response:

```
name: h2o_feet
tags: location = coyote_creek
time                            max
----                          ---
2015-08-18T00:00:00Z      8.12
2015-08-18T00:12:00Z      7.887
2015-08-18T00:24:00Z      7.635
2015-08-18T00:36:00Z      7.372
2015-08-18T00:48:00Z      7.11

name: h2o_feet
tags: location = santa_monica
time                            max
----                          ---
2015-08-18T00:00:00Z      2.116
2015-08-18T00:12:00Z      2.126
2015-08-18T00:24:00Z      2.051
2015-08-18T00:36:00Z      2.067
2015-08-18T00:48:00Z      1.991
```

## **MIN\(\)**

返回字段的最小值，字段类型必须是 int64, float64 或 boolean.

```
SELECT MIN(<field_key>)[,<tag_key(s)>] FROM <measurement_name> [WHERE <stuff>] [GROUP BY <stuff>]
```

Examples:

* 查询  measurement `h2o_feet`中`water_level`最小值

```
> SELECT MIN(water_level) FROM h2o_feet
```

CLI response:

```
name: h2o_feet
--------------
time                           min
2015-08-29T14:30:00Z     -0.61
```

* 查询 measurement `h2o_feet`中`water_level`最小值，同时显示`location` tag：

```
> SELECT MIN(water_level),location FROM h2o_feet
```

CLI response:

```
name: h2o_feet
--------------
time                          min      location
2015-08-29T14:30:00Z    -0.61    coyote_creek
```

* 根据12min interval和`location` tag 分组，查询 measurement `h2o_feet` 中的`water_level`在`2015-08-18T00:00:00Z`和`2015-08-18T00:54:00Z`之间的最小值：

```
> SELECT MIN(water_level) FROM h2o_feet WHERE time >= '2015-08-18T00:00:00Z' AND time < '2015-08-18T00:54:00Z' GROUP BY time(12m), location
```

CLI response:

```
name: h2o_feet
tags: location = coyote_creek
time                             min
----                             ---
2015-08-18T00:00:00Z       8.005
2015-08-18T00:12:00Z       7.762
2015-08-18T00:24:00Z       7.5
2015-08-18T00:36:00Z       7.234
2015-08-18T00:48:00Z       7.11

name: h2o_feet
tags: location = santa_monica
time                             min
----                             ---
2015-08-18T00:00:00Z       2.064
2015-08-18T00:12:00Z       2.028
2015-08-18T00:24:00Z       2.041
2015-08-18T00:36:00Z       2.057
2015-08-18T00:48:00Z       1.991
```

## **PERCENTILE\(\)**

返回指定字段排序后的百分之`N` ，字段类型必须为 int64 或者 float64。百分比 `N` 必须是 integer 或 floating，介于0和100之间，包含0和100。

```
SELECT PERCENTILE(<field_key>, <N>)[,<tag_key(s)>] FROM <measurement_name> [WHERE <stuff>] [GROUP BY <stuff>]
```

Examples:

* 查询`location`  等于 `coyote_creek` field `water_level` 前5%的值：

```
> SELECT PERCENTILE(water_level,5) FROM h2o_feet WHERE location = 'coyote_creek'
```

CLI response:

```
name: h2o_feet
--------------
time                           percentile
2015-09-09T11:42:00Z     1.148
```

`1.148` 大于 `water_level`  `location` = `coyote_creek`条件下中5%数值

* 查询前5% `water_level` 和 `location` tag：

```
> SELECT PERCENTILE(water_level,5),location FROM h2o_feet
```

CLI response:

```
name: h2o_feet
--------------
time                      percentile     location
2015-08-28T12:06:00Z      1.122          santa_monica

```

* 根据`location` tag分组，计算前100% field `water_level` ：

```
> SELECT PERCENTILE(water_level, 100) FROM h2o_feet GROUP BY location
```

CLI response:

```
name: h2o_feet
tags: location = coyote_creek
time                           percentile
----                           ----------
2015-08-29T07:24:00Z     9.964

name: h2o_feet
tags: location = santa_monica
time                           percentile
----                           ----------
2015-08-29T03:54:00Z     7.205
```

注意 `PERCENTILE(<field_key>,100)` 和 `MAX(<field_key>)相同`

当前, `PERCENTILE(<field_key>,0)` 和 `MIN(<field_key>)`不相等，见 GitHub Issue [\#4418](https://github.com/influxdata/influxdb/issues/4418) 。

## **TOP\(\)**

返回字段中最大的 `N` 个值，字段类型必须为int64和float64

```
SELECT TOP(<field_key>[,<tag_keys>],<N>)[,<tag_keys>] FROM <measurement_name> [WHERE <stuff>] [GROUP BY <stuff>]
```

Examples:

* 查询 `water_level`最大的3个值

```
> SELECT TOP(water_level,3) FROM h2o_feet
```

CLI response:

```
name: h2o_feet
--------------
time                           top
2015-08-29T07:18:00Z     9.957
2015-08-29T07:24:00Z     9.964
2015-08-29T07:30:00Z     9.954
```

* 查询 `water_level` 最大的3个值，并且返回相应的 `location` tag ：

```
> SELECT TOP(water_level,3),location FROM h2o_feet
```

```
name: h2o_feet
--------------
time                           top     location
2015-08-29T07:18:00Z     9.957   coyote_creek
2015-08-29T07:24:00Z     9.964   coyote_creek
2015-08-29T07:30:00Z     9.954   coyote_creek
```

* Select the largest value of `water_level` within each tag value of `location`:

```
> SELECT TOP(water_level,location,2) FROM h2o_feet
```

CLI response:

```
name: h2o_feet
--------------
time                           top     location
2015-08-29T03:54:00Z     7.205   santa_monica
2015-08-29T07:24:00Z     9.964   coyote_creek
```

The output shows the top values of `water_level` for each tag value of `location` \(`santa_monica` and `coyote_creek`\).

> **Note:** Queries with the syntax `SELECT TOP(<field_key>,<tag_key>,<N>)`, where the tag has `X` distinct values, return `N` or `X` field values, whichever is smaller, and each returned point has a unique tag value. To demonstrate this behavior, see the results of the above example query where `N` equals `3` and `N` equals `1`.
> 
> * `N` = `3`
> 
> ```
> SELECT TOP(water_level,location,3) FROM h2o_feet
> ```
> 
> CLI response:
> 
> ```
> name: h2o_feet
> --------------
> time                           top     location
> 2015-08-29T03:54:00Z     7.205   santa_monica
> 2015-08-29T07:24:00Z     9.964   coyote_creek
> ```
> 
> InfluxDB returns two values instead of three because the `location` tag has only two values \(`santa_monica` and`coyote_creek`\).
> 
> * `N` = `1`
> 
> ```
> SELECT TOP(water_level,location,1) FROM h2o_feet
> ```
> 
> CLI response:
> 
> ```
> name: h2o_feet
> --------------
> time                           top     location
> 2015-08-29T07:24:00Z     9.964   coyote_creek
> ```
> 
> InfluxDB compares the top values of `water_level` within each tag value of `location` and returns the larger value of `water_level`.

* Select the largest two values of `water_level` between August 18, 2015 at 4:00:00 and August 18, 2015 at 4:18:00 for every tag value of `location`:

```
> SELECT TOP(water_level,2) FROM h2o_feet WHERE time >= '2015-08-18T04:00:00Z' AND time < '2015-08-18T04:24:00Z' GROUP BY location
```

CLI response:

```
name: h2o_feet
tags: location=coyote_creek
time                           top
----                           ---
2015-08-18T04:00:00Z     2.943
2015-08-18T04:06:00Z     2.831


name: h2o_feet
tags: location=santa_monica
time                           top
----                           ---
2015-08-18T04:06:00Z     4.055
2015-08-18T04:18:00Z     4.124
```

* Select the largest two values of `water_level` between August 18, 2015 at 4:00:00 and August 18, 2015 at 4:18:00 in `santa_monica`:

```
> SELECT TOP(water_level,2) FROM h2o_feet WHERE time >= '2015-08-18T04:00:00Z' AND time < '2015-08-18T04:24:00Z' AND location = 'santa_monica'
```

CLI response:

```
name: h2o_feet
--------------
time                           top
2015-08-18T04:06:00Z     4.055
2015-08-18T04:18:00Z     4.124
```

Note that in the raw data, `water_level` equals `4.055` at `2015-08-18T04:06:00Z` and at `2015-08-18T04:12:00Z`. In the case of a tie, InfluxDB returns the value with the earlier timestamp.

# **Transformations**

## **CEILING\(\)**

`CEILING()` is not yet functional.

See GitHub Issue [\#5930](https://github.com/influxdata/influxdb/issues/5930) for more information.

## **DERIVATIVE\(\)**

Returns the rate of change for the values in a single [field](https://github.com/influxdata/docs.influxdata.com/blob/master/influxdb/v0.13/concepts/glossary/#field) in a [series](https://github.com/influxdata/docs.influxdata.com/blob/master/influxdb/v0.13/concepts/glossary/#series). InfluxDB calculates the difference between chronological field values and converts those results into the rate of change per `unit`. The `unit` argument is optional and, if not specified, defaults to one second \(`1s`\).

The basic `DERIVATIVE()` query:

```
SELECT DERIVATIVE(<field_key>, [<unit>]) FROM <measurement_name> [WHERE <stuff>]
```

Valid time specifications for `unit` are:

`u` microseconds

`s` seconds

`m` minutes

`h` hours

`d` days

`w` weeks

`DERIVATIVE()` also works with a nested function coupled with a `GROUP BY time()` clause. For queries that include those options, InfluxDB first performs the aggregation, selection, or transformation across the time interval specified in the `GROUP BY time()` clause. It then calculates the difference between chronological field values and converts those results into the rate of change per `unit`. The `unit` argument is optional and, if not specified, defaults to the same interval as the `GROUP BY time()` interval.

The `DERIVATIVE()` query with an aggregation function and `GROUP BY time()` clause:

```
SELECT DERIVATIVE(AGGREGATION_FUNCTION(<field_key>),[<unit>]) FROM <measurement_name> WHERE <stuff> GROUP BY time(<aggregation_interval>)
```

Examples:

The following examples work with the first six observations of the `water_level` field in the measurement `h2o_feet`with the tag set `location = santa_monica`:

```
name: h2o_feet
--------------
time                           water_level
2015-08-18T00:00:00Z     2.064
2015-08-18T00:06:00Z     2.116
2015-08-18T00:12:00Z     2.028
2015-08-18T00:18:00Z     2.126
2015-08-18T00:24:00Z     2.041
2015-08-18T00:30:00Z     2.051
```

* `DERIVATIVE()` with a single argument:

  Calculate the rate of change per one second


```
> SELECT DERIVATIVE(water_level) FROM h2o_feet WHERE location = 'santa_monica' LIMIT 5
```

CLI response:

```
name: h2o_feet
--------------
time                           derivative
2015-08-18T00:06:00Z     0.00014444444444444457
2015-08-18T00:12:00Z     -0.00024444444444444465
2015-08-18T00:18:00Z     0.0002722222222222218
2015-08-18T00:24:00Z     -0.000236111111111111
2015-08-18T00:30:00Z     2.777777777777842e-05
```

Notice that the first field value \(`0.00014`\) in the `derivative` column is **not** `0.052` \(the difference between the first two field values in the raw data: `2.116` - `2.604` = `0.052`\). Because the query does not specify the `unit` option, InfluxDB automatically calculates the rate of change per one second, not the rate of change per six minutes. The calculation of the first value in the `derivative` column looks like this:

```
(2.116 - 2.064) / (360s / 1s)

```

The numerator is the difference between chronological field values. The denominator is the difference between the relevant timestamps in seconds \(`2015-08-18T00:06:00Z` - `2015-08-18T00:00:00Z` = `360s`\) divided by `unit` \(`1s`\). This returns the rate of change per second from `2015-08-18T00:00:00Z` to `2015-08-18T00:06:00Z`.

* `DERIVATIVE()` with two arguments:

  Calculate the rate of change per six minutes


```
> SELECT DERIVATIVE(water_level,6m) FROM h2o_feet WHERE location = 'santa_monica' LIMIT 5
```

CLI response:

```
name: h2o_feet
--------------
time                           derivative
2015-08-18T00:06:00Z     0.052000000000000046
2015-08-18T00:12:00Z     -0.08800000000000008
2015-08-18T00:18:00Z     0.09799999999999986
2015-08-18T00:24:00Z     -0.08499999999999996
2015-08-18T00:30:00Z     0.010000000000000231
```

The calculation of the first value in the `derivative` column looks like this:

```
(2.116 - 2.064) / (6m / 6m)

```

The numerator is the difference between chronological field values. The denominator is the difference between the relevant timestamps in minutes \(`2015-08-18T00:06:00Z` - `2015-08-18T00:00:00Z` = `6m`\) divided by `unit` \(`6m`\). This returns the rate of change per six minutes from `2015-08-18T00:00:00Z` to `2015-08-18T00:06:00Z`.

* `DERIVATIVE()` with two arguments:

  Calculate the rate of change per 12 minutes


```
> SELECT DERIVATIVE(water_level,12m) FROM h2o_feet WHERE location = 'santa_monica' LIMIT 5
```

CLI response:

```
name: h2o_feet
--------------
time                           derivative
2015-08-18T00:06:00Z     0.10400000000000009
2015-08-18T00:12:00Z     -0.17600000000000016
2015-08-18T00:18:00Z     0.19599999999999973
2015-08-18T00:24:00Z     -0.16999999999999993
2015-08-18T00:30:00Z     0.020000000000000462
```

The calculation of the first value in the `derivative` column looks like this:

```
(2.116 - 2.064 / (6m / 12m)

```

The numerator is the difference between chronological field values. The denominator is the difference between the relevant timestamps in minutes \(`2015-08-18T00:06:00Z` - `2015-08-18T00:00:00Z` = `6m`\) divided by `unit` \(`12m`\). This returns the rate of change per 12 minutes from `2015-08-18T00:00:00Z` to `2015-08-18T00:06:00Z`.

> **Note:** Specifying `12m` as the `unit` **does not** mean that InfluxDB calculates the rate of change for every 12 minute interval of data. Instead, InfluxDB calculates the rate of change per 12 minutes for each interval of valid data.

* `DERIVATIVE()` with one argument, a function, and a `GROUP BY time()` clause:

  Select the `MAX()` value at 12 minute intervals and calculate the rate of change per 12 minutes


```
> SELECT DERIVATIVE(MAX(water_level)) FROM h2o_feet WHERE location = 'santa_monica' AND time >= '2015-08-18T00:00:00Z' AND time < '2015-08-18T00:36:00Z' GROUP BY time(12m)
```

CLI response:

```
name: h2o_feet
--------------
time                           derivative
2015-08-18T00:12:00Z     0.009999999999999787
2015-08-18T00:24:00Z     -0.07499999999999973
```

To get those results, InfluxDB first aggregates the data by calculating the `MAX()` `water_level` at the time interval specified in the `GROUP BY time()` clause \(`12m`\). Those results look like this:

```
name: h2o_feet
--------------
time                           max
2015-08-18T00:00:00Z     2.116
2015-08-18T00:12:00Z     2.126
2015-08-18T00:24:00Z     2.051
```

Second, InfluxDB calculates the rate of change per `12m` \(the same interval as the `GROUP BY time()` interval\) to get the results in the `derivative` column above. The calculation of the first value in the `derivative` column looks like this:

```
(2.126 - 2.116) / (12m / 12m)

```

The numerator is the difference between chronological field values. The denominator is the difference between the relevant timestamps in minutes \(`2015-08-18T00:12:00Z` - `2015-08-18T00:00:00Z` = `12m`\) divided by `unit` \(`12m`\). This returns rate of change per 12 minutes for the aggregated data from `2015-08-18T00:00:00Z` to `2015-08-18T00:12:00Z`.

* `DERIVATIVE()` with two arguments, a function, and a `GROUP BY time()` clause:

  Aggregate the data to 18 minute intervals and calculate the rate of change per six minutes


```
> SELECT DERIVATIVE(SUM(water_level),6m) FROM h2o_feet WHERE location = 'santa_monica' AND time >= '2015-08-18T00:00:00Z' AND time < '2015-08-18T00:36:00Z' GROUP BY time(18m)
```

CLI response:

```
name: h2o_feet
--------------
time                           derivative
2015-08-18T00:18:00Z     0.0033333333333332624
```

To get those results, InfluxDB first aggregates the data by calculating the `SUM()` of `water_level` at the time interval specified in the `GROUP BY time()` clause \(`18m`\). The aggregated results look like this:

```
name: h2o_feet
--------------
time                           sum
2015-08-18T00:00:00Z     6.208
2015-08-18T00:18:00Z     6.218
```

Second, InfluxDB calculates the rate of change per `unit` \(`6m`\) to get the results in the `derivative` column above. The calculation of the first value in the `derivative` column looks like this:

```
(6.218 - 6.208) / (18m / 6m)

```

The numerator is the difference between chronological field values. The denominator is the difference between the relevant timestamps in minutes \(`2015-08-18T00:18:00Z` - `2015-08-18T00:00:00Z` = `18m`\) divided by `unit` \(`6m`\). This returns the rate of change per six minutes for the aggregated data from `2015-08-18T00:00:00Z` to `2015-08-18T00:18:00Z`.

## **DIFFERENCE\(\)**

Returns the difference between consecutive chronological values in a single [field](https://github.com/influxdata/docs.influxdata.com/blob/master/influxdb/v0.13/concepts/glossary/#field). The field type must be int64 or float64.

The basic `DIFFERENCE()` query:

```
SELECT DIFFERENCE(<field_key>) FROM <measurement_name> [WHERE <stuff>]

```

The `DIFFERENCE()` query with a nested function and a `GROUP BY time()` clause:

```
SELECT DIFFERENCE(<function>(<field_key>)) FROM <measurement_name> WHERE <stuff> GROUP BY time(<time_interval>)

```

Functions that work with `DIFFERENCE()` include `COUNT()`, `MEAN()`, `MEDIAN()`, `SUM()`, `FIRST()`, `LAST()`, `MIN()`,`MAX()`, and `PERCENTILE()`.

Examples:

The following examples focus on the field `water_level` in `santa_monica` between `2015-08-18T00:00:00Z` and `2015-08-18T00:36:00Z`:

```
> SELECT water_level FROM h2o_feet WHERE location='santa_monica' AND time >= '2015-08-18T00:00:00Z' and time <= '2015-08-18T00:36:00Z'
name: h2o_feet
--------------
time                            water_level
2015-08-18T00:00:00Z      2.064
2015-08-18T00:06:00Z      2.116
2015-08-18T00:12:00Z      2.028
2015-08-18T00:18:00Z      2.126
2015-08-18T00:24:00Z      2.041
2015-08-18T00:30:00Z      2.051
2015-08-18T00:36:00Z      2.067

```

* Calculate the difference between `water_level` values:

```
> SELECT DIFFERENCE(water_level) FROM h2o_feet WHERE location='santa_monica' AND time >= '2015-08-18T00:00:00Z' and time <= '2015-08-18T00:36:00Z'

```

CLI response:

```
name: h2o_feet
--------------
time                            difference
2015-08-18T00:06:00Z      0.052000000000000046
2015-08-18T00:12:00Z      -0.08800000000000008
2015-08-18T00:18:00Z      0.09799999999999986
2015-08-18T00:24:00Z      -0.08499999999999996
2015-08-18T00:30:00Z      0.010000000000000231
2015-08-18T00:36:00Z      0.016000000000000014

```

The first value in the `difference` column is `2.116 - 2.064`, and the second value in the `difference` column is`2.028 - 2.116`. Please note that the extra decimal places are the result of floating point inaccuracies.

* Select the minimum `water_level` values at 12 minute intervals and calculate the difference between those values:

```
> SELECT DIFFERENCE(MIN(water_level)) FROM h2o_feet WHERE location='santa_monica' AND time >= '2015-08-18T00:00:00Z' and time <= '2015-08-18T00:36:00Z' GROUP BY time(12m)

```

CLI response:

```
name: h2o_feet
--------------
time                            difference
2015-08-18T00:12:00Z      -0.03600000000000003
2015-08-18T00:24:00Z      0.0129999999999999
2015-08-18T00:36:00Z      0.026000000000000245

```

To get the values in the `difference` column, InfluxDB first selects the `MIN()` values at 12 minute intervals:

```
> SELECT MIN(water_level) FROM h2o_feet WHERE location='santa_monica' AND time >= '2015-08-18T00:00:00Z' and time <= '2015-08-18T00:36:00Z' GROUP BY time(12m)
name: h2o_feet
--------------
time                            min
2015-08-18T00:00:00Z    2.064
2015-08-18T00:12:00Z    2.028
2015-08-18T00:24:00Z    2.041
2015-08-18T00:36:00Z    2.067

```

It then uses those values to calculate the difference between chronological values; the first value in the `difference`column is `2.028 - 2.064`.

## **ELAPSED\(\)**

Returns the difference between subsequent timestamps in a single [field](https://github.com/influxdata/docs.influxdata.com/blob/master/influxdb/v0.13/concepts/glossary/#field). The `unit` argument is an optional [duration literal](https://github.com/influxdata/docs.influxdata.com/blob/master/influxdb/v0.13/query_language/spec/#durations) and, if not specified, defaults to one nanosecond.

```
SELECT ELAPSED(<field_key>, <unit>) FROM <measurement_name> [WHERE <stuff>]

```

Examples:

* Calculate the difference \(in nanoseconds\) between the timestamps in the field `h2o_feet`:

```
> SELECT ELAPSED(water_level) FROM h2o_feet WHERE location = 'santa_monica' AND time >= '2015-08-18T00:00:00Z' and time <= '2015-08-18T00:24:00Z'

```

CLI Response:

```
name: h2o_feet
--------------
time                            elapsed
2015-08-18T00:06:00Z      360000000000
2015-08-18T00:12:00Z      360000000000
2015-08-18T00:18:00Z      360000000000
2015-08-18T00:24:00Z      360000000000

```

* Calculate the number of one minute intervals between the timestamps in the field `h2o_feet`:

```
> SELECT ELAPSED(water_level,1m) FROM h2o_feet WHERE location = 'santa_monica' AND time >= '2015-08-18T00:00:00Z' and time <= '2015-08-18T00:24:00Z'

```

CLI Response:

```
name: h2o_feet
--------------
time                            elapsed
2015-08-18T00:06:00Z      6
2015-08-18T00:12:00Z      6
2015-08-18T00:18:00Z      6
2015-08-18T00:24:00Z      6

```

> **Note:** InfluxDB returns `0` if `unit` is greater than the difference between the timestamps. For example, the timestamps in `h2o_feet` occur at six minute intervals. If the query asks for the number of one hour intervals between the timestamps, InfluxDB returns `0`:
> 
> ```
> SELECT ELAPSED(water_level,1h) FROM h2o_feet WHERE location = 'santa_monica' AND time >= '2015-08-18T00:00:00Z' and time <= '2015-08-18T00:24:00Z'
> name: h2o_feet
> --------------
> time                            elapsed
> 2015-08-18T00:06:00Z      0
> 2015-08-18T00:12:00Z      0
> 2015-08-18T00:18:00Z      0
> 2015-08-18T00:24:00Z      0
> 
> ```

## **FLOOR\(\)**

`FLOOR()` is not yet functional.

See GitHub Issue [\#5930](https://github.com/influxdb/influxdb/issues/5930) for more information.

## **HISTOGRAM\(\)**

`HISTOGRAM()` is not yet functional.

See GitHub Issue [\#5930](https://github.com/influxdb/influxdb/issues/5930) for more information.

## **MOVING\_AVERAGE\(\)**

Returns the moving average across a `window` of consecutive chronological field values for a single [field](https://github.com/influxdata/docs.influxdata.com/blob/master/influxdb/v0.13/concepts/glossary/#field). The field type must be int64 or float64.

The basic `MOVING_AVERAGE()` query:

```
SELECT MOVING_AVERAGE(<field_key>,<window>) FROM <measurement_name> [WHERE <stuff>]

```

The `MOVING_AVERAGE()` query with a nested function and a `GROUP BY time()` clause:

```
SELECT MOVING_AVERAGE(<function>(<field_key>),<window>) FROM <measurement_name> WHERE <stuff> GROUP BY time(<time_interval>)

```

Functions that work with `MOVING_AVERAGE()` include `COUNT()`, `MEAN()`, `MEDIAN()`, `SUM()`, `FIRST()`, `LAST()`,`MIN()`, `MAX()`, and `PERCENTILE()`.

Examples:

The following examples focus on the field `water_level` in `santa_monica` between `2015-08-18T00:00:00Z` and `2015-08-18T00:36:00Z`:

```
> SELECT water_level FROM h2o_feet WHERE location = 'santa_monica' AND time >= '2015-08-18T00:00:00Z' and time <= '2015-08-18T00:36:00Z'
name: h2o_feet
--------------
time                            water_level
2015-08-18T00:00:00Z      2.064
2015-08-18T00:06:00Z      2.116
2015-08-18T00:12:00Z      2.028
2015-08-18T00:18:00Z      2.126
2015-08-18T00:24:00Z      2.041
2015-08-18T00:30:00Z      2.051
2015-08-18T00:36:00Z      2.067

```

* Calculate the moving average across every 2 field values:

```
> SELECT MOVING_AVERAGE(water_level,2) FROM h2o_feet WHERE location = 'santa_monica' AND time >= '2015-08-18T00:00:00Z' and time <= '2015-08-18T00:36:00Z'

```

CLI response:

```
name: h2o_feet
--------------
time                            moving_average
2015-08-18T00:06:00Z      2.09
2015-08-18T00:12:00Z      2.072
2015-08-18T00:18:00Z      2.077
2015-08-18T00:24:00Z      2.0835
2015-08-18T00:30:00Z      2.0460000000000003
2015-08-18T00:36:00Z      2.059

```

The first value in the `moving_average` column is the average of `2.064` and `2.116`, the second value in the`moving_average` column is the average of `2.116` and `2.028`.

* Select the minimum `water_level` at 12 minute intervals and calculate the moving average across every 2 field values:

```
> SELECT MOVING_AVERAGE(MIN(water_level),2) FROM h2o_feet WHERE location = 'santa_monica' AND time >= '2015-08-18T00:00:00Z' and time <= '2015-08-18T00:36:00Z' GROUP BY time(12m)

```

CLI response:

```
name: h2o_feet
--------------
time                            moving_average
2015-08-18T00:12:00Z      2.0460000000000003
2015-08-18T00:24:00Z      2.0345000000000004
2015-08-18T00:36:00Z      2.0540000000000003

```

To get those results, InfluxDB first selects the `MIN()` `water_level` for every 12 minute interval:

```
name: h2o_feet
--------------
time                            min
2015-08-18T00:00:00Z      2.064
2015-08-18T00:12:00Z      2.028
2015-08-18T00:24:00Z      2.041
2015-08-18T00:36:00Z      2.067

```

It then uses those values to calculate the moving average across every 2 field values; the first result in the`moving_average` column the average of `2.064` and `2.028`, and the second result is the average of `2.028` and`2.041`.

## **NON\_NEGATIVE\_DERIVATIVE\(\)**

Returns the non-negative rate of change for the values in a single [field](https://github.com/influxdata/docs.influxdata.com/blob/master/influxdb/v0.13/concepts/glossary/#field) in a [series](https://github.com/influxdata/docs.influxdata.com/blob/master/influxdb/v0.13/concepts/glossary/#series). InfluxDB calculates the difference between chronological field values and converts those results into the rate of change per `unit`. The `unit` argument is optional and, if not specified, defaults to one second \(`1s`\).

The basic `NON_NEGATIVE_DERIVATIVE()` query:

```
SELECT NON_NEGATIVE_DERIVATIVE(<field_key>, [<unit>]) FROM <measurement_name> [WHERE <stuff>]
```

Valid time specifications for `unit` are:

`u` microseconds

`s` seconds

`m` minutes

`h` hours

`d` days

`w` weeks

`NON_NEGATIVE_DERIVATIVE()` also works with a nested function coupled with a `GROUP BY time()` clause. For queries that include those options, InfluxDB first performs the aggregation, selection, or transformation across the time interval specified in the `GROUP BY time()` clause. It then calculates the difference between chronological field values and converts those results into the rate of change per `unit`. The `unit` argument is optional and, if not specified, defaults to the same interval as the `GROUP BY time()` interval.

The `NON_NEGATIVE_DERIVATIVE()` query with an aggregation function and `GROUP BY time()` clause:

```
SELECT NON_NEGATIVE_DERIVATIVE(AGGREGATION_FUNCTION(<field_key>),[<unit>]) FROM <measurement_name> WHERE <stuff> GROUP BY time(<aggregation_interval>)
```

See `DERIVATIVE()` for example queries. All query results are the same for `DERIVATIVE()` and`NON_NEGATIVE_DERIVATIVE` except that `NON_NEGATIVE_DERIVATIVE()` returns only the positive values.

## **STDDEV\(\)**

Returns the standard deviation of the values in a single [field](https://github.com/influxdata/docs.influxdata.com/blob/master/influxdb/v0.13/concepts/glossary/#field). The field must be of type int64 or float64.

```
SELECT STDDEV(<field_key>) FROM <measurement_name> [WHERE <stuff>] [GROUP BY <stuff>]
```

Examples:

* Calculate the standard deviation for the `water_level` field in the measurement `h2o_feet`:

```
> SELECT STDDEV(water_level) FROM h2o_feet
```

CLI response:

```
name: h2o_feet
--------------
time                           stddev
1970-01-01T00:00:00Z     2.279144584196145
```

> **Notes:**
> 
> * `stddev()` returns epoch 0 \(`1970-01-01T00:00:00Z`\) as the timestamp unless you specify a lower bound on the time range. Then it returns the lower bound as the timestamp.
> * Executing `stddev()` on the same set of float64 points may yield slightly different results. InfluxDB does not sort points before it applies the function which results in those small discrepancies.

* Calculate the standard deviation for the `water_level` field between August 18, 2015 at midnight and September 18, 2015 at noon grouped at one week intervals and by the `location` tag:

```
> SELECT STDDEV(water_level) FROM h2o_feet WHERE time >= '2015-08-18T00:00:00Z' and time < '2015-09-18T12:06:00Z' GROUP BY time(1w), location
```

CLI response:

```
name: h2o_feet
tags: location = coyote_creek
time                           stddev
----                           ------
2015-08-13T00:00:00Z     2.2437263080193985
2015-08-20T00:00:00Z     2.121276150144719
2015-08-27T00:00:00Z     3.0416122170786215
2015-09-03T00:00:00Z     2.5348065025435207
2015-09-10T00:00:00Z     2.584003954882673
2015-09-17T00:00:00Z     2.2587514836274414

name: h2o_feet
tags: location = santa_monica
time                           stddev
----                           ------
2015-08-13T00:00:00Z     1.11156344587553
2015-08-20T00:00:00Z     1.0909849279082366
2015-08-27T00:00:00Z     1.9870116180096962
2015-09-03T00:00:00Z     1.3516778450902067
2015-09-10T00:00:00Z     1.4960573811500588
2015-09-17T00:00:00Z     1.075701669442093
```

## **Include multiple functions in a single query**

Separate multiple functions in one query with a `,`.

Calculate the [minimum](https://github.com/influxdata/docs.influxdata.com/blob/master/influxdb/v0.13/query_language/functions/#min) `water_level` and the [maximum](https://github.com/influxdata/docs.influxdata.com/blob/master/influxdb/v0.13/query_language/functions/#max) `water_level` with a single query:

```
> SELECT MIN(water_level), MAX(water_level) FROM h2o_feet
```

CLI response:

```
name: h2o_feet
--------------
time                           min     max
1970-01-01T00:00:00Z     -0.61   9.964
```

## **Change the value reported for intervals with no data with **`fill()`

By default, queries with an InfluxQL function report `null` values for intervals with no data. Append `fill()` to the end of your query to alter that value. For a complete discussion of `fill()`, see [Data Exploration](https://github.com/influxdata/docs.influxdata.com/blob/master/influxdb/v0.13/query_language/data_exploration/#the-group-by-clause-and-fill).

> **Note:** `fill()` works differently with `COUNT()`. See [the documentation on ](https://github.com/influxdata/docs.influxdata.com/blob/master/influxdb/v0.13/query_language/functions/#count-and-controlling-the-values-reported-for-intervals-with-no-data)`COUNT()` for a function-specific use of`fill()`.

## **Rename the output column's title with **`AS`

By default, queries that include a function output a column that has the same name as that function. If you'd like a different column name change it with an `AS` clause.

Before:

```
> SELECT MEAN(water_level) FROM h2o_feet
```

CLI response:

```
name: h2o_feet
--------------
time                           mean
1970-01-01T00:00:00Z     4.442107025822521
```

After:

```
> SELECT MEAN(water_level) AS dream_name FROM h2o_feet
```

CLI response:

```
name: h2o_feet
--------------
time                           dream_name
1970-01-01T00:00:00Z     4.442107025822521
```

