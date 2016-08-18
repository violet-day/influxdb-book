使用InfluxQL functions 来聚合、查询、转化数据。

| **Aggregations** | **Selectors** | **Transformations** |
| --- | --- | --- |
| [COUNT\(\)](#count) | [BOTTOM\(\)](#bottom) | [CEILING\(\)](#ceiling) |
| [DISTINCT\(\)](#distinct) | [FIRST\(\)](#first) | [DERIVATIVE\(\)](#derivative) |
| [INTEGRAL\(\)](#integral) | [LAST\(\)](#last) | [DIF](https://github.com/influxdata/docs.influxdata.com/blob/master/influxdb/v0.13/query_language/functions/#difference)[FERENC](#difference)[E\(\)](https://github.com/influxdata/docs.influxdata.com/blob/master/influxdb/v0.13/query_language/functions/#difference) |
| [MEAN\(\)](#mean) | [MAX\(\)](#max) | [ELAPSED\(\)](#elapsed) |
| [MEDIAN\(\)](#median) | [MIN\(\)](#min) | [FLOOR\(\)](#floor) |
| [SPREAD\(\)](#spread) | [PERCENTILE\(\)](#percentile) | [HISTOGRAM\(\)](#histogram) |
| [SUM\(\)](#sum) | [TOP\(\)](#top) | [MOVING\_AVERAGE\(\)](#movingaverage) |
|  |  | [NON\_NEGATIVE\_DERIVATIVE\(\)](#nonnegativederivative) |
|  |  | [STDDEV\(\)](#stddev) |

Useful InfluxQL for functions:

* [Include multiple functions in a single query](#include-multiple-functions-in-a-single-query)
* [Change the value reported for intervals with no data with ](#change-the-value-reported-for-intervals-with-no-data-with-fill)`fill()`
* [Rename the output column's title with ](#rename-the-output-columns-title-with-as)`AS`

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

## **INTEGRAL\(\)** {#integral}

`INTEGRAL()` is not yet functional.

See GitHub Issue [\#5930](https://github.com/influxdata/influxdb/issues/5930) for more information.

## **MEAN\(\)** {#mean}

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

* 查询每个 `location`tag中最大的`water_level`value：

```
> SELECT TOP(water_level,location,2) FROM h2o_feet
```

CLI response:

```
name: h2o_feet
--------------
time                      top     location
2015-08-29T03:54:00Z     7.205   santa_monica
2015-08-29T07:24:00Z     9.964   coyote_creek
```

结果展示了每个`location`tag（`santa_monica` and `coyote_creek`）中的`water_level`最大值。

> **Note:** 使用 `SELECT TOP(<field_key>,<tag_key>,<N>)`时, 假设tag有 `X` 个不一样的值，结果则返回 `N` 或 `X` 个值。基本上是，每个点都有唯一的tag value。为了演示，看下面 `N` ＝ `3` 和 `N` ＝ `1`和时的查询例子：
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
> InfluxDB 仅返回了2个值而不是3个，是因为`location` tag  \(`santa_monica` and`coyote_creek`\)仅有2个值
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
> InfluxDB 比较了每个`location` tag中的`water_level` 的最大值，返回了其中更大的结果。

* 查询`2015-08-18T04:00:00Z`和`2015-08-18T04:24:00Z`时间段内每个tag `location`中`water_level`最大的两个值：

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

* 查询`2015-08-18T04:00:00Z`和`2015-08-18T04:24:00Z`时间段内 tag `santa_monica` 内`water_level` 最大的两个值：

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

注意，在原本的数据中， `water_level`  在 `2015-08-18T04:06:00Z` 和 `2015-08-18T04:12:00Z`都等于 `4.055`。相同情况下，InfluxDB返回更早的时间戳。

# **Transformations**

## **CEILING\(\)**

`CEILING()` is not yet functional.

See GitHub Issue [\#5930](https://github.com/influxdata/influxdb/issues/5930) for more information.

## **DERIVATIVE\(\)**

返回series中的某个field的变化率。InfluxDB每`unit`根据时间顺序计算field value之间的变化率。 `unit` 参数为可选的，默认值为`1s`

基本 `DERIVATIVE()` 查询：

```
SELECT DERIVATIVE(<field_key>, [<unit>]) FROM <measurement_name> [WHERE <stuff>]
```

可用的 `unit` ：

`u` microseconds

`s` seconds

`m` minutes

`h` hours

`d` days

`w` weeks

`DERIVATIVE()` 也可以配合 `GROUP BY time()` 子句使用前套函数。这种情况下，InfluxDB 首先根据指定的`GROUP BY time()`执行 aggregation、selection或者transformation 函数，然后以`unit`为单位计算根据时间计算之前结果的变化率。`unit` 参数可选，如果没有指定，则和 `GROUP BY time()` 中指定的invterval一致。

`DERIVATIVE()` 、 aggregation 函数 和 `GROUP BY time()` 一起使用：

```
SELECT DERIVATIVE(AGGREGATION_FUNCTION(<field_key>),[<unit>]) FROM <measurement_name> WHERE <stuff> GROUP BY time(<aggregation_interval>)
```

Examples:

下面的例子查询了`location = santa_monica`时measurement `h2o_feet`中`water_level` field 变化率：

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

* `DERIVATIVE()` 仅使用一个参数：

  计算单个指标每秒的变化率


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

注意，`derivative`列中的第一个值 \(`0.00014`\) **不是** `0.052` \(原始数据中两个值的差为: `2.116` - `2.604` = `0.052`\)。之所以这样的原因是查询中没有指定 `unit` , InfluxDB 自动计算了每秒的变化，而不是每6分钟。`derivative`列的计算公式为：

```
(2.116 - 2.064) / (360s / 1s)
```

分子是按照时间顺序field value之间的差值，分母是时间差值 \(`2015-08-18T00:06:00Z` - `2015-08-18T00:00:00Z` = `360s`\) 除以 `unit` \(`1s`\)，也就是 `2015-08-18T00:00:00Z` 到 `2015-08-18T00:06:00Z`每秒的变化率

* `DERIVATIVE()` 中使用2个参数：

  计算每6分钟的变化率：


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

`derivative` 列的计算公式为：

```
(2.116 - 2.064) / (6m / 6m)
```

分子是按照时间顺序field value之间的差值，分母是时间差 \(`2015-08-18T00:06:00Z` - `2015-08-18T00:00:00Z` = `6m`\) 除以 `unit` \(`6m`\)，也就是 `2015-08-18T00:00:00Z` 到 `2015-08-18T00:06:00Z`每6min的变化率

* `DERIVATIVE()` 中使用2个参数：

  计算每12分钟的变化率


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

`derivative` 的计算公式为：

```
(2.116 - 2.064 / (6m / 12m)
```

分子是按照时间顺序field value之间的差值， 分母是时间差 \(`2015-08-18T00:06:00Z` - `2015-08-18T00:00:00Z` = `6m`\) 除以 `unit` \(`12m`\)，也就是 `2015-08-18T00:00:00Z` 到 `2015-08-18T00:06:00Z`每12分钟的变化率

> **Note: ** `12m` 作为单位 `unit` 并不 意味着 InfluxDB 每12分钟计算一次变化率，而是对于每个时间间隔之间的数据，计算每12分钟对应的变化。

* `DERIVATIVE()` 使用一个参数并和 `GROUP BY time()` clause：

  以12分钟作为interval，得到 `MAX()` 后计算每12分钟的变化率。


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

InfluxDB 首先使用`GROUP BY time()` clause \(`12m`\) 分组计算`MAX()` `water_level`：

```
name: h2o_feet
--------------
time                      max
2015-08-18T00:00:00Z     2.116
2015-08-18T00:12:00Z     2.126
2015-08-18T00:24:00Z     2.051
```

然后, InfluxDB 计算每 `12m` \(和 `GROUP BY time()` interval相同\) 的变化率作为`derivative` 列。 第一个`derivative`的计算为：

```
(2.126 - 2.116) / (12m / 12m)
```

分子是按照时间顺序field value之间的差值，分母是时间差\(`2015-08-18T00:12:00Z` - `2015-08-18T00:00:00Z` = `12m`\) 除以 `unit` \(`12m`\)，也就是`2015-08-18T00:00:00Z` 到 `2015-08-18T00:12:00Z`之间聚合之后的数据每12分钟的变化率。

* `DERIVATIVE()` 使用2个参数，并和 `GROUP BY time()` clause一起使用：

  以18min作为interval分组，计算每6分钟的变化率


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

InfluxDB 首先使用`GROUP BY time()` clause \(`18m`\) 通过`SUM()` 聚合`water_level`：

```
name: h2o_feet
--------------
time                           sum
2015-08-18T00:00:00Z     6.208
2015-08-18T00:18:00Z     6.218
```

然后， InfluxDB 每 `unit` \(`6m`\) 计算 `derivative` 。计算如下：

```
(6.218 - 6.208) / (18m / 6m) 
```

分子是按照时间顺序field value之间的差值，分母是时间差 \(`2015-08-18T00:18:00Z` - `2015-08-18T00:00:00Z` = `18m`\) 除以 `unit` \(`6m`\)，也就是`2015-08-18T00:00:00Z` 到 `2015-08-18T00:18:00Z`之间聚合之后的数据，每6分钟的变化率。

## **DIFFERENCE\(\)**

按照时间顺序，就计算单个字段的差值，字段类型必须是int64 或 float64。

基本的 `DIFFERENCE()` 查询：

```
SELECT DIFFERENCE(<field_key>) FROM <measurement_name> [WHERE <stuff>]
```

使用`GROUP BY time()`，`DIFFERENCE()` 内部嵌套函数：

```
SELECT DIFFERENCE(<function>(<field_key>)) FROM <measurement_name> WHERE <stuff> GROUP BY time(<time_interval>) 
```

可以和 `DIFFERENCE()` 一起使用的有： `COUNT()`, `MEAN()`, `MEDIAN()`, `SUM()`, `FIRST()`, `LAST()`, `MIN()`,`MAX()`, 和`PERCENTILE()`.

Examples:

下面的例子主要关注`2015-08-18T00:00:00Z` 到 `2015-08-18T00:36:00Z`之间，`santa_monica`的`water_level`字段：

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

* 计算`water_level`间的差值：

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

`difference` 列的第一个值是 `2.116 - 2.064，`第二个是`2.028 - 2.116`。请注意，额外的小数位是因为浮点性计算导致的。

* 每12分钟计算`water_level`最小值，并计算最小值之间的差值 ：

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

为了得到 `difference` ，InfluxDB 首先根据12分钟分组，查询 `MIN()` values：

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

在这这结果中，根据时间序列计算差值。第一个值为 `difference` 为 `2.028 - 2.064`.

## **ELAPSED\(\)**

返回时间戳的差值，`unit` 参数可选，默认值为纳秒。

```
SELECT ELAPSED(<field_key>, <unit>) FROM <measurement_name> [WHERE <stuff>]
```

Examples:

* 以纳秒形式计算 field `h2o_feet`的差值：

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

* 以1分钟为interval，计算 field `h2o_feet`的时间戳差：

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

> **Note:** 如果 `unit` 大于 timestamps查值时，InfluxDB 返回 `0`  。比如，在 `h2o_feet` 按照6分钟为interval出现，如果你查询参数中为`1h`，InfluxDB returns `0`:
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

返回连续一段`window`中的某个字段的均线，字段类型必须是int64 或 float64。

基本的 `MOVING_AVERAGE()` 查询：

```
SELECT MOVING_AVERAGE(<field_key>,<window>) FROM <measurement_name> [WHERE <stuff>]
```

配合`GROUP BY time()` ，`MOVING_AVERAGE()` 可以嵌套函数使用

```
SELECT MOVING_AVERAGE(<function>(<field_key>),<window>) FROM <measurement_name> WHERE <stuff> GROUP BY time(<time_interval>) 
```

可以和 `MOVING_AVERAGE()` 和一起使用有： `COUNT()`, `MEAN()`, `MEDIAN()`, `SUM()`, `FIRST()`, `LAST()`,`MIN()`, `MAX()`, 和 `PERCENTILE()`

Examples:

下面的例子主要`2015-08-18T00:00:00Z` 到 `2015-08-18T00:36:00Z`范围内关注 `santa_monica`中的field `water_level`：

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

* 每2个值计算均线：

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

在`moving_average` 列中的第一个值是 `2.064` 和 `2.116`的平均值，第二个值是 `2.116` 和 `2.028`的平均值

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

InfluxDB 首先根据12分钟的interval计算`water_level`的`MIN()` ：

```
name: h2o_feet
--------------
time                            min
2015-08-18T00:00:00Z      2.064
2015-08-18T00:12:00Z      2.028
2015-08-18T00:24:00Z      2.041
2015-08-18T00:36:00Z      2.067

```

然后用这些数据，每2个值计算平均值。`moving_average` 第一个值是 `2.064` 和 `2.028`的平均值，第二个是 `2.028` 和`2.041。`

## **NON\_NEGATIVE\_DERIVATIVE\(\)**

返回series中某个field的非负变化率。InfluxDB每`unit`根据时间顺序计算field value之间的变化率。 `unit` 参数为可选的，默认值为`1s`

基本查询语法 `NON_NEGATIVE_DERIVATIVE()` ：

```
SELECT NON_NEGATIVE_DERIVATIVE(<field_key>, [<unit>]) FROM <measurement_name> [WHERE <stuff>]
```

用法同`DERIVATIVE`

## **STDDEV\(\)**

返回某个field的标准差，字段类型必须是int64 或 float64.

```
SELECT STDDEV(<field_key>) FROM <measurement_name> [WHERE <stuff>] [GROUP BY <stuff>]
```

Examples:

* 计算measurement `h2o_feet` 中的`water_level` field：

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

* 根据1周和`location` 分组，计算`2015-08-18T00:00:00Z`和 `2015-09-18T12:06:00Z`之间`water_level`的标准差：

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

查询中的多个function通过 `,`分隔

在一个查询中计算`water_level`的 [minimum](#min) 和  [maximum ：](#max)

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

在interval中，没有数据的情况下，默认返回 `null` 。可以通过在查询最后添加 `fill()` 来改变。完整的讲解件： [Data Exploration](/data-exploration.md).

> **Note:** `fill()` 在`COUNT()`的处理有一些不一致。见 [the documentation on ](#count)`COUNT()` 对`fill()`的用法

## **Rename the output column's title with **`AS`

查询结果默认使用function name作为列名，如果你想使用其他名称，可以使用 `AS` clause。

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

