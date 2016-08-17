InfluxQL是与InfluxDB数据交互使用的类SQL的查询语言。这节讲会介绍一些查询语法来获取数据。

基本的: 

* [The `SELECT` statement and the `WHERE` clause](#the-select-statement-and-the-where-clause) 
    * [The basic `SELECT` statement](#the-basic-select-statement) 
    * [The `SELECT` statement and arithmetic](#the-select-statement-and-arithmetic)
    * [The `WHERE` clause](#the-where-clause) 
* [The `GROUP BY` clause](#the-group-by-clause) 
    * [`GROUP BY` tag values](#group-by-tag-values)     
    * [`GROUP BY` time intervals](#group-by-time-intervals)       
    * [`GROUP BY` tag values and a time interval](#group-by-tag-values-and-a-time-interval)
    * [The `GROUP BY` clause and `fill()`](#the-group-by-clause-and-fill) 
* [The `INTO` clause](#the-into-clause) 
    * [Relocate data](#relocate-data)
    * [Downsample data](#downsample-data) 
    
Limit and sort your results: 

* [Limit query returns with `LIMIT` and `SLIMIT`](#limit-query-returns-with-limit-and-slimit)
	* [Limit results per series with `LIMIT`](#limit-the-number-of-results-returned-per-series-with-limit) 
	* [Limit the number of series returned with `SLIMIT`](#limit-the-number-of-series-returned-with-slimit) 
	* [Limit the number of points and series returned with `LIMIT` and `SLIMIT`](#limit-the-number-of-points-and-series-returned-with-limit-and-slimit) 
* [Sort query returns with `ORDER BY time DESC`](#sort-query-returns-with-order-by-time-desc)
* [Paginate query returns with `OFFSET` and `SOFFSET`](#paginate-query-returns-with-offset-and-soffset) 

General tips on query syntax: 
	
* [Multiple statements in queries](#multiple-statements-in-queries) 
* [Merge series in queries](#merge-series-in-queries) 
* [Time syntax in queries](#time-syntax-in-queries) 
	* [Relative time](#relative-time) 
	* [Absolute time](#absolute-time) 
* [Regular expressions in queries](#regular-expressions-in-queries) 
	* [Regular expressions and selecting measurements](#regular-expressions-and-selecting-measurements) 
	* [Regular expressions and specifying tags](#regular-expressions-and-specifying-tags) 

示例使用 [InfluxDB's Command Line Interface (CLI)](/influxdb/v0.13/tools/shell/)。同时可以通过 HTTP API 来 [Querying Data](/influxdb/v0.13/guides/querying_data/) 
	
#### Sample data <br> 
	
如果你想尝试这篇文章中的查询，查看[Sample Data](/influxdb/v0.13/sample_data/data_download/) 下载和导入数据至InfluxDB. 这篇文章使用[National Oceanic and Atmospheric Administration's (NOAA) Center for Operational Oceanographic Products and Services](http://tidesandcurrents.noaa.gov/stations.html?type=Water+Levels)的公开可用数据。数据包含了从 August 18, 2015 到 September 18, 2015期间，两个站点(`(Santa Monica, CA (ID 9410840)` 和 `Coyote Creek, CA (ID 9414575))`)，water levels(ft)，每六秒收集1次

 measurement `h2o_feet`一段数据: 
	
``` 
name: h2o_feet 
-------------- 
time                 level description      location     water_level 
2015-08-18T00:00:00Z between 6 and 9 feet   coyote_creek 8.12 
2015-08-18T00:00:00Z below 3 feet           santa_monica 2.064 
2015-08-18T00:06:00Z between 6 and 9 feet   coyote_creek 8.005 
2015-08-18T00:06:00Z below 3 feet           santa_monica 2.116 
2015-08-18T00:12:00Z between 6 and 9 feet   coyote_creek 7.887 
2015-08-18T00:12:00Z below 3 feet           santa_monica 2.028 
2015-08-18T00:18:00Z between 6 and 9 feet   coyote_creek 7.762 
2015-08-18T00:18:00Z below 3 feet           santa_monica 2.126 
2015-08-18T00:24:00Z between 6 and 9 feet   coyote_creek 7.635 
2015-08-18T00:24:00Z below 3 feet           santa_monica 2.041 
``` 
	
这个[series](/influxdb/v0.13/concepts/glossary/#series) 由 [measurement](/influxdb/v0.13/concepts/glossary/#measurement) `h2o_feet` 和 [tag key](/influxdb/v0.13/concepts/glossary/#tag-key) `location` 和 [tag values](/influxdb/v0.13/concepts/glossary/#tag-value) `santa_monica` 和 `coyote_creek`组成。有两个字段： [fields](/influxdb/v0.13/concepts/glossary/#field): `water_level` 存储为floats，`level description` 存储为string。所有的数据在 `NOAA_water_database` database中。
	
> **Disclaimer:** `level description` 字段不是NOAA的数据一部分 - 我们偷偷的放在这里的是为了加一个以sting存储的key
	
## The SELECT statement and the `WHERE` clause 
	
InfluxQL的`SELECT`声明和SQL中的`SELECT`声明一致，`WHERE`：
	
```sql 
SELECT <stuff> FROM <measurement_name> WHERE <some_conditions> 
``` 
	
### The basic `SELECT` statement 
--- 
下面的3个例子，从measurement`h20_feet`返回所有数据。虽然它们返回的结果相同，但是方式有点区别，理解这些有助于理解`SELECT`语法。

通过`*`从`h2o_feet`中查询所有数据：
	
```sql 
> SELECT * FROM h2o_feet 
``` 

从`h2o_feet`中通过指定tag key和field key查询数据：
	
```sql 
> SELECT "level description",location,water_level FROM h2o_feet 
``` 
	
* 通过逗号分隔你所需要的field和tag。注意，在`SELECT`声明中，至少包含一个字段。
* Leave identifiers unquoted unless they start with a digit, contain characters other than `[A-z,0-9,_]`, or if they are an [InfluxQL keyword](https://github.com/influxdb/influxdb/blob/master/influxql/README.md#keywords) - then you need to double quote them. Identifiers are database names, retention policy names, user names, measurement names, tag keys, and field keys. 通过全名查询 `h2o_feet` : 

```sql 
> SELECT * FROM NOAA_water_database."default".h2o_feet 
``` 

* 如果你想从不同的database或者retention policy，需要指定measurement的全名。全名的形式如下：

``` 
"<database>"."<retention policy>"."<measurement>" 
``` 
	
The CLI response for all three queries: 
	
``` 
name: h2o_feet 
-------------- 
time                 level description    location     water_level 
2015-08-18T00:00:00Z between 6 and 9 feet coyote_creek 8.12 
2015-08-18T00:00:00Z below 3 feet         santa_monica 2.064 	
2015-08-18T00:06:00Z between 6 and 9 feet coyote_creek 8.005 
2015-08-18T00:06:00Z below 3 feet         santa_monica 2.116 
[...] 
2015-09-18T21:24:00Z between 3 and 6 feet santa_monica 5.013 
2015-09-18T21:30:00Z between 3 and 6 feet santa_monica 5.01 
2015-09-18T21:36:00Z between 3 and 6 feet santa_monica 5.066 
2015-09-18T21:42:00Z between 3 and 6 feet santa_monica 4.938 
``` 
	
### The `SELECT` statement and arithmetic 
--- 
对于存储为float或者integer的字段做一些基本算法。对`water_level`加2: 
	
```sql 
> SELECT water_level + 2 FROM h2o_feet 
``` 
	
CLI 响应: 
	
```bash 
name: h2o_feet 
-------------- 
time 
2015-08-18T00:00:00Z 10.12 
2015-08-18T00:00:00Z 4.064 
[...] 
2015-09-18T21:36:00Z 7.066 
2015-09-18T21:42:00Z 6.938 
``` 
	
另外一个例子: 
	
```sql 
> SELECT (water_level * 2) + 4 from h2o_feet 
``` 
	
CLI 响应: 
	
```bash 
name: h2o_feet 
-------------- 
time 2015-08-18T00:00:00Z 20.24 
2015-08-18T00:00:00Z 8.128 [...] 
2015-09-18T21:36:00Z 14.132 
2015-09-18T21:42:00Z 13.876 
``` 
	
> **Note:** 在存储为integer的field上做数学计算时，integer类型会被转换为float。对于某些数字可能会导致溢出问题。
	
### The `WHERE` clause 
--- 
使用`WHERE`基于tags、时间范围或者field来过滤数据。

> **Note:** 引号的语法和 [line protocol](/influxdb/v0.13/concepts/glossary/#line-protocol)和有区别。 参见 [rules for single and double-quoting](/influxdb/v0.13/troubleshooting/frequently_encountered_issues/#single-quoting-and-double-quoting-in-queries) 
	
**Tags** 
	
返回tag `location`值为`santa_monica`的数据：

```sql 
> SELECT water_level FROM h2o_feet WHERE location = 'santa_monica' 
``` 

* 在tag查询中，使用单引号，因为它们是string。注意，使用双引号查询tag时不会起任何作用，并且会导致静默的错误。
	
> **Note:** Tags是建立了索引的，所以在上面执行的查询比在field上更高效。

返回`location`不包含值的数据（更多参见正则表达式[later](#regular-expressions-in-queries)）

```sql 
> SELECT * FROM h2o_feet WHERE location !~ /./ 
``` 

返回`location`包含值的数据：

```sql
 > SELECT * FROM h2o_feet WHERE location =~ /./ 
``` 
	
**Time ranges** 

返回过去7天的数据：

```sql 
> SELECT * FROM h2o_feet WHERE time > now() - 7d 
``` 
	
* `now()` 表示server上执行query的Unix。更多关于 `now()` 其他方式的查询参见 [time syntax in queries](#time-syntax-in-queries). 
	
**Field values** 

返回 tag `location`值为`coyote_creek`并且field `water_level`大于8 feet：
	
```sql 
> SELECT * FROM h2o_feet WHERE location = 'coyote_creek' AND water_level > 8 
``` 
	
返回 tag `location`值为`santa_monica`，field `level description` 等于 `'below 3 feet'`: 
	
```sql 
> SELECT * FROM h2o_feet WHERE location = 'santa_monica' AND "level description" = 'below 3 feet' 
```
	
返回`water_level`乘以2大于`11.9`:
	
``` 
> SELECT * FROM h2o_feet WHERE water_level + 2 > 11.9 
``` 
	
* field值为string类型时，永远使用单引号。注意，双引号不会起任何作用，并且会导致静默的错误。
	
> **Note:** Fields上没有构建索引；查询效率不如tags。

InfluxQL的`WHERE`更多用法如下:
 	
* `WHERE`字句中支持对strings、booleans、floats、integers比较，`time`通过timestamp比较。支持郑泽表达式匹配tag，但是field不支持。
* 链式逻辑使用 `AND` 和 `OR`, 分隔符使用 `(` 和 `)`. 
* 支持的比较符: 
  * `=` 等于
  * `<>` 不等于
  * `!=` 不等于
  * `>` 大于
  * `<` 小于
  * `=~` 匹配
  * `!~` 不匹配

## The GROUP BY clause 


使用 `GROUP BY` 子句根据 tag 或者 time intervals对数据分组. 正确使用 `GROUP BY`的姿势为将 `GROUP BY` 添加至 `SELECT` 声明后，并对 `SELECT` 声明使用 InfluxQL的 [functions](/influxdb/v0.13/query_language/functions/): 
	
> **Note:** 如果你的查询同时包含 `WHERE` 和 `GROUP BY` , `GROUP BY` 必须在 `WHERE` 之后。

### GROUP BY tag values 

对于不同的`location` 计算 [`MEAN()`](/influxdb/v0.13/query_language/functions/#mean) `water_level`: 
	
```sql 
> SELECT MEAN(water_level) FROM h2o_feet GROUP BY location 
``` 
	
CLI response: 
	
```bash 
> SELECT MEAN(water_level) FROM h2o_feet GROUP BY location 
name: h2o_feet 
tags: location=coyote_creek 
time mean 
---- ---- 
1970-01-01T00:00:00Z 5.359342451341401 
	
name: h2o_feet 
tags: location=santa_monica 
time mean 
---- ---- 
1970-01-01T00:00:00Z 3.530863470081006 
``` 
	
>**Note:** 在InfluxDB中, [epoch 0](https://en.wikipedia.org/wiki/Unix_time) (`1970-01-01T00:00:00Z`) 通常用来表示空时间戳。如果你请求的查询没有时间戳反悔，比如没有指定时间返回的聚合返回，InfluxDB会返回`epoch 0`作为时间戳。

为`h2o_quality`中的每个tag set计算 [`MEAN()`](/influxdb/v0.13/query_language/functions/#mean) `index` ：

```sql
> SELECT MEAN(index) FROM h2o_quality GROUP BY *
```

CLI response:

```bash
name: h2o_quality
tags: location=coyote_creek, randtag=1
time			               mean
----			               ----
1970-01-01T00:00:00Z	 50.55405446521169


name: h2o_quality
tags: location=coyote_creek, randtag=2
time			               mean
----			               ----
1970-01-01T00:00:00Z	 50.49958856271162


name: h2o_quality
tags: location=coyote_creek, randtag=3
time			               mean
----			               ----
1970-01-01T00:00:00Z	 49.5164137518956


name: h2o_quality
tags: location=santa_monica, randtag=1
time			               mean
----			               ----
1970-01-01T00:00:00Z	 50.43829082296367


name: h2o_quality
tags: location=santa_monica, randtag=2
time			               mean
----			               ----
1970-01-01T00:00:00Z	 52.0688508894012


name: h2o_quality
tags: location=santa_monica, randtag=3
time			               mean
----			               ----
1970-01-01T00:00:00Z	 49.29386362086556
```

### GROUP BY time intervals

用户可以通过给定的time interval 使用 `GROUP BY time()`对数据进行分组:

```
SELECT <function>(<field_key>) FROM <measurement_name> WHERE <time_range> GROUP BY time(<time_interval>[,<offset_interval>])
```

如果使用在InfluxQL中使用 `GROUP BY` 和 `time()`，就需要添加 `WHERE` 字句。注意，除非你指定时间范围的上下界，否则使用`epoch 0`作为下界，`now()`作为上界

`time()` 的有效单位：  

* `u` microseconds    
* `ms` milliseconds  
* `s` seconds  
* `m` minutes  
* `h` hours  
* `d` days  
* `w` weeks  

#### Rounded `GROUP BY time()` boundaries

默认的， `GROUP BY time()` 返回落在时间范围内的结果。

Example:

以3d为间隔，[`COUNT()`](/influxdb/v0.13/query_language/functions/#count) 位于2015-08-19凌晨和2015-08-27下午5点之前的 `water_level` point 数量

```sql
> SELECT COUNT(water_level) FROM h2o_feet WHERE time >= '2015-08-19T00:00:00Z' AND time <= '2015-08-27T17:00:00Z' AND location='coyote_creek' GROUP BY time(3d)
```

CLI response:

```bash
name: h2o_feet
--------------
time			 count
2015-08-18T00:00:00Z	 480
2015-08-21T00:00:00Z	 720
2015-08-24T00:00:00Z	 720
2015-08-27T00:00:00Z	 171
```

每个时间戳表现为3天一个间隔，`count`字段的值表示`water_level`出现在3d间隔中的point数量。
You could get the same results by querying the data four times - that is, one `COUNT()` query for every three days between August 19, 2015 at midnight and August 27, 2015 at 5:00pm - but that could take a while.

注意到，在CLI response (`2015-08-18T00:00:00Z`) 的出现是比时间范围 (`2015-08-19T00:00:00Z`)小的。这是因为默认 `GROUP BY time()` 落在完整的日历区间中。虽然`count`的结果显示的时间是`2015-08-18T00:00:00Z`，但是也仅包含了`2015-08-19T00:00:00Z`范围内的数据。参见 [Frequently Encountered Issues](/influxdb/v0.13/troubleshooting/frequently_encountered_issues/#understanding-the-time-intervals-returned-from-group-by-time-queries) 了解更详细的
`GROUP BY time()` 默认行为。

#### Configured `GROUP BY time()` boundaries

`GROUP BY time()` 同样允许你通过设置offset interval来更改默认的日历区间。

Examples:

以3d为间隔，偏移1d，[`COUNT()`](/influxdb/v0.13/query_language/functions/#count) 位于2015-08-19凌晨和2015-08-27下午5点之前的 `water_level` point 数量

```sql
> SELECT COUNT(water_level) FROM h2o_feet WHERE time >= '2015-08-19T00:00:00Z' AND time <= '2015-08-27T17:00:00Z' AND location='coyote_creek' GROUP BY time(3d,1d)
```

CLI response:

```
name: h2o_feet
--------------
time			               count
2015-08-19T00:00:00Z	 720
2015-08-22T00:00:00Z	 720
2015-08-25T00:00:00Z	 651
```

`1d` 偏移间隔修改了默认3d时间间隔的日历边界为

```
from                          to
August 18 - August 20         August 19 - August 21
August 21 - August 23         August 22 - August 24
August 24 - August 26         August 25 - August 27
August 27 - August 29         
```

以3d为间隔，偏移-2d，[`COUNT()`](/influxdb/v0.13/query_language/functions/#count) 位于2015-08-19凌晨和2015-08-27下午5点之前的 `water_level` point 数量

```
> SELECT COUNT(water_level) FROM h2o_feet WHERE time >= '2015-08-19T00:00:00Z' AND time <= '2015-08-27T17:00:00Z' AND location='coyote_creek' GROUP BY time(3d,-2d)
```

CLI response:

```
name: h2o_feet
--------------
time			               count
2015-08-19T00:00:00Z	 720
2015-08-22T00:00:00Z	 720
2015-08-25T00:00:00Z	 651
```

`-2d` 偏移间隔修改了默认3d时间间隔的日历边界为

```
August 18 - August 20         August 16 - August 18
August 21 - August 23         August 19 - August 21
August 24 - August 26         August 22 - August 24
August 27 - August 29         August 25 - August 27
```

InfluxDB 不返回第一个时间间隔(August 16 - August 18)中的结果，因为它不在`WHERE`子句范围内。

### GROUP BY tag values AND a time interval

Separate multiple `GROUP BY` arguments with a comma.

Calculate the average `water_level` for the different tag values of `location` in the last two weeks at 6 hour intervals:
```sql
> SELECT MEAN(water_level) FROM h2o_feet WHERE time > now() - 2w GROUP BY location,time(6h)
```

### The `GROUP BY` clause and `fill()`
---
默认的， `GROUP BY` interval  没有数据时，使用 `null` z作为在列中的输出值。使用 `fill()` 函数来修改在没有数据时的展示。`fill()` 的选项包括：

* 任何数值
* `null` - 在没有数值时用 `null` 作为它的值
* `previous` - 没有数据时，复制上一个interval的值作为值
* `none` - 跳过这个interval

下面的例子中可以看到`fill()`可以做到的事情

**GROUP BY without fill()**

```sql
> SELECT MEAN(water_level) FROM h2o_feet WHERE time >= '2015-08-18' AND time < '2015-09-24' GROUP BY time(10d)
```

CLI response:

```bash
name: h2o_feet
--------------
time			                 mean
2015-08-13T00:00:00Z	   4.306212083333323
2015-08-23T00:00:00Z	   4.318944629367029
2015-09-02T00:00:00Z	   4.363877681204781
2015-09-12T00:00:00Z   	4.69811470811633
✨2015-09-22T00:00:00Z
```

**GROUP BY with fill()**  

`fill(-10)`:
 
```sql
> SELECT MEAN(water_level) FROM h2o_feet WHERE time >= '2015-08-18' AND time < '2015-09-24' GROUP BY time(10d) fill(-100)
```

CLI response:  

```bash
name: h2o_feet
--------------
time			                 mean
2015-08-13T00:00:00Z	   4.306212083333323
2015-08-23T00:00:00Z	   4.318944629367029
2015-09-02T00:00:00Z	   4.363877681204781
2015-09-12T00:00:00Z	   4.698114708116322
✨2015-09-22T00:00:00Z	 -100
```

`fill(none)`:

```sql
> SELECT MEAN(water_level) FROM h2o_feet WHERE time >= '2015-08-18' AND time < '2015-09-24' GROUP BY time(10d) fill(none)
```

CLI response:  

```bash
name: h2o_feet
--------------
time			               mean
2015-08-13T00:00:00Z	 4.306212083333323
2015-08-23T00:00:00Z	 4.318944629367029
2015-09-02T00:00:00Z	 4.363877681204781
2015-09-12T00:00:00Z	 4.69811470811633
✨
```

> **Note:** 如果 `GROUP(ing) BY` 使用了多个字段(比如包含tags 和 time interval) `fill()` 必须位于 `GROUP BY` 的最后。

## The INTO clause

### Relocate data

使用`INTO`将数据复制至另外的database，retention policy和measurement：

```sql
SELECT <field_key> INTO <different_measurement> FROM <current_measurement> [WHERE <stuff>] [GROUP BY <stuff>]
```

将`h2o_feet`中的`water_level`写入位于相同database的新measurement(`h2o_feet_copy`)中：


```sql
> SELECT water_level INTO h2o_feet_copy FROM h2o_feet WHERE location = 'coyote_creek'
```

The CLI 返回了多少写入至 `h2o_feet_copy`的point的数量:

```
name: result
------------
time			               written
1970-01-01T00:00:00Z	 7604
```

将`h2o_feet`中的`water_level` 写入到已经存在 database `where_else`的`default` retention policy中：

```sql
> SELECT water_level INTO where_else."default".h2o_feet_copy FROM h2o_feet WHERE location = 'coyote_creek'
```

CLI response:

```bash
name: result
------------
time			               written
1970-01-01T00:00:00Z	 7604
```

> **Note**: 如果使用 `SELECT *` with `INTO`, query 会将当前 measurment 中的 tags 转换为新 measurement 中的 fields。这会导致InfludxDB在写入时区和之前的tag值有所不同。可以使用 `GROUP BY <tag_key>` 保留tag。

### Downsample data
Combine the `INTO` clause with an InfluxQL [function](/influxdb/v0.13/query_language/functions/) and a `GROUP BY` clause to write the lower precision query results to a different measurement:
```sql
SELECT <function>(<field_key>) INTO <different_measurement> FROM <current_measurement> WHERE <stuff> GROUP BY <stuff>
```

> **Note:** The `INTO` queries in this section downsample old data, that is, data that have already been written to InfluxDB.
If you want InfluxDB to automatically query and downsample all future data see [Continuous Queries](/influxdb/v0.13/query_language/continuous_queries/).

Calculate the average `water_level` in `santa_monica`, and write the results to a new measurement (`average`) in the same database:
```sql
> SELECT mean(water_level) INTO average FROM h2o_feet WHERE location = 'santa_monica' AND time >= '2015-08-18T00:00:00Z' AND time <= '2015-08-18T00:30:00Z' GROUP BY time(12m)
```

The CLI response shows the number of points that InfluxDB wrote to the new measurement:
```bash
name: result
------------
time			               written
1970-01-01T00:00:00Z	 3
```

To see the query results, select everything from the new measurement `average` in `NOAA_water_database`:
```bash
> SELECT * FROM average
name: average
-------------
time			               mean
2015-08-18T00:00:00Z	 2.09
2015-08-18T00:12:00Z	 2.077
2015-08-18T00:24:00Z	 2.0460000000000003
```

Calculate the average `water_level` and the max `water_level` in `santa_monica`, and write the results to a new measurement (`aggregates`) in a different database (`where_else`):
```sql
> SELECT mean(water_level), max(water_level) INTO where_else."default".aggregates FROM h2o_feet WHERE location = 'santa_monica' AND time >= '2015-08-18T00:00:00Z' AND time <= '2015-08-18T00:30:00Z' GROUP BY time(12m)
```

CLI response:
```bash
name: result
------------
time			               written
1970-01-01T00:00:00Z	 3
```

Select everything from the new measurement `aggregates` in the database `where_else`:
```bash
> SELECT * FROM where_else."default".aggregates
name: aggregates
----------------
time			               max	   mean
2015-08-18T00:00:00Z	 2.116	 2.09
2015-08-18T00:12:00Z	 2.126	 2.077
2015-08-18T00:24:00Z	 2.051	 2.0460000000000003
```

Calculate the average `degrees` for all temperature measurements (`h2o_temperature` and `average_temperature`) in the `NOAA_water_database` and write the results to new measurements with the same names in a different database (`where_else`).
`:MEASUREMENT` tells InfluxDB to write the query results to measurements with the same names as those targeted by the query:
```sql
> SELECT mean(degrees) INTO where_else."default".:MEASUREMENT FROM /temperature/ WHERE time >= '2015-08-18T00:00:00Z' AND time <= '2015-08-18T00:30:00Z' GROUP BY time(12m)
```

CLI response:
```bash
name: result
------------
time			               written
1970-01-01T00:00:00Z	 6
```

Select the `mean` field from all new temperature measurements in the database `where_else`.
```bash
> SELECT mean FROM where_else."default"./temperature/
name: average_temperature
-------------------------
time			               mean
2015-08-18T00:00:00Z	 78.5
2015-08-18T00:12:00Z	 84
2015-08-18T00:24:00Z	 74.75

name: h2o_temperature
---------------------
time			                mean
2015-08-18T00:00:00Z	  63.75
2015-08-18T00:12:00Z	  63.5
2015-08-18T00:24:00Z	  63.5
```

More on downsampling with `INTO`:

* InfluxDB does not store null values.
Depending on the frequency of your data, the query results may be missing time intervals.
Use [fill()](/influxdb/v0.13/query_language/data_exploration/#the-group-by-clause-and-fill) to ensure that every time interval appears in the results.
* The number of writes in the CLI response includes one write for every time interval in the query's time range even if there is no data for some of the time intervals.

## Limit query returns with LIMIT and SLIMIT
InfluxQL supports two different clauses to limit your query results:

* `LIMIT <N>` returns the first \<N> [points](/influxdb/v0.13/concepts/glossary/#point) from each [series](/influxdb/v0.13/concepts/glossary/#series) in the specified measurement.
* `SLIMIT <N>` returns every point from \<N> series in the specified measurement.
* `LIMIT <N>` followed by `SLIMIT <N>` returns the first \<N> points from \<N> series in the specified measurement.

### Limit the number of results returned per series with `LIMIT`
---
Use `LIMIT <N>` with `SELECT` and `GROUP BY *` to return the first \<N> points from each series.

Return the three oldest points from each series associated with the measurement `h2o_feet`:
```sql
> SELECT water_level FROM h2o_feet GROUP BY * LIMIT 3
```

CLI response:
```bash
name: h2o_feet
tags: location=coyote_creek
time			              water_level
----			              -----------
2015-08-18T00:00:00Z	8.12
2015-08-18T00:06:00Z	8.005
2015-08-18T00:12:00Z	7.887

name: h2o_feet
tags: location=santa_monica
time			              water_level
----			              -----------
2015-08-18T00:00:00Z	2.064
2015-08-18T00:06:00Z	2.116
2015-08-18T00:12:00Z	2.028
```

> **Note:** If \<N> is greater than the number of points in the series, InfluxDB returns all points in the series.

### Limit the number of series returned with `SLIMIT`
---
Use `SLIMIT <N>` with `SELECT` and `GROUP BY *` to return every point from \<N> series.

Return everything from one of the series associated with the measurement `h2o_feet`:
```sql
> SELECT water_level FROM h2o_feet GROUP BY * SLIMIT 1
```

CLI response:
```bash
name: h2o_feet
tags: location=coyote_creek
time			              water_level
----			              -----
2015-08-18T00:00:00Z	8.12
2015-08-18T00:06:00Z	8.005
2015-08-18T00:12:00Z	7.887
[...]
2015-09-18T16:12:00Z	3.402
2015-09-18T16:18:00Z	3.314
2015-09-18T16:24:00Z	3.235
```

> **Note:** If \<N> is greater than the number of series associated with the specified measurement, InfluxDB returns all points from every series.

### Limit the number of points and series returned with `LIMIT` and `SLIMIT`
---
Use `LIMIT <N1>` followed by `SLIMIT <N2>` with `GROUP BY *` to return \<N1> points from \<N2> series.

Return the three oldest points from one of the series associated with the measurement `h2o_feet`:
```sql
> SELECT water_level FROM h2o_feet GROUP BY * LIMIT 3 SLIMIT 1
```

CLI response:
```bash
name: h2o_feet
tags: location=coyote_creek
time			               water_level
----			               -----------
2015-08-18T00:00:00Z	 8.12
2015-08-18T00:06:00Z	 8.005
2015-08-18T00:12:00Z	 7.887
```

> **Note:** If \<N1> is greater than the number of points in the series, InfluxDB returns all points in the series.
If \<N2> is greater than the number of series associated with the specified measurement, InfluxDB returns points from every series.

## Sort query returns with ORDER BY time DESC
By default, InfluxDB returns results in ascending time order - so the first points that are returned are the oldest points by timestamp.
Use `ORDER BY time DESC` to see the newest points by timestamp.

Return the oldest five points from one series **without** `ORDER BY time DESC`:  
```sql
> SELECT water_level FROM h2o_feet WHERE location = 'santa_monica' LIMIT 5
```

CLI response:  
```bash
name: h2o_feet
----------
time			water_level
2015-08-18T00:00:00Z	2.064
2015-08-18T00:06:00Z	2.116
2015-08-18T00:12:00Z	2.028
2015-08-18T00:18:00Z	2.126
2015-08-18T00:24:00Z	2.041
```

Now include  `ORDER BY time DESC` to get the newest five points from the same series:  
```sql
> SELECT water_level FROM h2o_feet WHERE location = 'santa_monica' ORDER BY time DESC LIMIT 5
```

CLI response:  
```bash
name: h2o_feet
----------
time			water_level
2015-09-18T21:42:00Z	4.938
2015-09-18T21:36:00Z	5.066
2015-09-18T21:30:00Z	5.01
2015-09-18T21:24:00Z	5.013
2015-09-18T21:18:00Z	5.072
```

Finally, use `GROUP BY` with `ORDER BY time DESC` to return the last five points from each series:  
```sql
> SELECT water_level FROM h2o_feet GROUP BY location ORDER BY time DESC LIMIT 5
```

CLI response:
```bash
name: h2o_feet
tags: location=santa_monica
time			               water_level
----			               -----------
2015-09-18T21:42:00Z	 4.938
2015-09-18T21:36:00Z	 5.066
2015-09-18T21:30:00Z	 5.01
2015-09-18T21:24:00Z	 5.013
2015-09-18T21:18:00Z	 5.072

name: h2o_feet
tags: location=coyote_creek
time			               water_level
----			               -----------
2015-09-18T16:24:00Z	 3.235
2015-09-18T16:18:00Z	 3.314
2015-09-18T16:12:00Z	 3.402
2015-09-18T16:06:00Z	 3.497
2015-09-18T16:00:00Z	 3.599
```

## Paginate query returns with OFFSET and SOFFSET

### Use `OFFSET` to paginate the results returned
---
For example, get the first three points written to a series:

```sql
> SELECT water_level FROM h2o_feet WHERE location = 'coyote_creek' LIMIT 3
```

CLI response:  
```bash
name: h2o_feet
----------
time			               water_level
2015-08-18T00:00:00Z	 8.12
2015-08-18T00:06:00Z	 8.005
2015-08-18T00:12:00Z	 7.887
```

Then get the second three points from that same series:

```sql
> SELECT water_level FROM h2o_feet WHERE location = 'coyote_creek' LIMIT 3 OFFSET 3
```

CLI response:
```bash
name: h2o_feet
----------
time			               water_level
2015-08-18T00:18:00Z	 7.762
2015-08-18T00:24:00Z	 7.635
2015-08-18T00:30:00Z	 7.5
```

### Use `SOFFSET` to paginate the series returned
---

For example, get the first three points from a single series:
```
> SELECT water_level FROM h2o_feet GROUP BY * LIMIT 3 SLIMIT 1
```

CLI response:
```
name: h2o_feet
tags: location=coyote_creek
time			               water_level
----			               -----------
2015-08-18T00:00:00Z	 8.12
2015-08-18T00:06:00Z	 8.005
2015-08-18T00:12:00Z	 7.887
```

Then get the first three points from the next series:
```
> SELECT water_level FROM h2o_feet GROUP BY * LIMIT 3 SLIMIT 1 SOFFSET 1
```

CLI response:
```
name: h2o_feet
tags: location=santa_monica
time			               water_level
----			               -----------
2015-08-18T00:00:00Z	 2.064
2015-08-18T00:06:00Z	 2.116
2015-08-18T00:12:00Z	 2.028
```

## Multiple statements in queries

Separate multiple statements in a query with a semicolon.
For example:
<br>
<br>
```sql
> SELECT mean(water_level) FROM h2o_feet WHERE time > now() - 2w GROUP BY location,time(24h) fill(none); SELECT count(water_level) FROM h2o_feet WHERE time > now() - 2w GROUP BY location,time(24h) fill(80)
```

## Merge series in queries

In InfluxDB, queries merge series automatically.

The `NOAA_water_database` database has two [series](/influxdb/v0.13/concepts/glossary/#series).
The first series is made up of the measurement `h2o_feet` and the tag key `location` with the tag value `coyote_creek`.
The second series is made of up the measurement `h2o_feet` and the tag key `location` with the tag value `santa_monica`.

The following query automatically merges those two series when it calculates the [average](/influxdb/v0.13/query_language/functions/#mean) `water_level`:

```sql
> SELECT MEAN(water_level) FROM h2o_feet
```

CLI response:
```bash
name: h2o_feet
--------------
time			               mean
1970-01-01T00:00:00Z	 4.319097913525821
```

If you only want the `MEAN()` `water_level` for the first series, specify the tag set in the `WHERE` clause:
```sql
> SELECT MEAN(water_level) FROM h2o_feet WHERE location = 'coyote_creek'
```

CLI response:
```bash
name: h2o_feet
--------------
time			               mean
1970-01-01T00:00:00Z	 5.296914449406493
```

> **NOTE:** In InfluxDB, [epoch 0](https://en.wikipedia.org/wiki/Unix_time) (`1970-01-01T00:00:00Z`) is often used as a null timestamp equivalent.
If you request a query that has no timestamp to return, such as an aggregation function with an unbounded time range, InfluxDB returns epoch 0 as the timestamp.

## Time syntax in queries  
InfluxDB is a time series database so, unsurprisingly, InfluxQL has a lot to do with specifying time ranges.
If you do not specify start and end times in your query, they default to epoch 0 (`1970-01-01T00:00:00Z`) and `now()`.
The following sections detail how to specify different start and end times in queries.

### Relative time
---
`now()` is the Unix time of the server at the time the query is executed on that server.
Use `now()` to calculate a timestamp relative to the server's
current timestamp.

Query data starting an hour ago and ending `now()`:
```sql
> SELECT water_level FROM h2o_feet WHERE time > now() - 1h
```

Query data that occur between epoch 0 and 1,000 days from `now()`:  
```sql
> SELECT "level description" FROM h2o_feet WHERE time < now() + 1000d
```

* Note the whitespace between the operator and the time duration.
Leaving that whitespace out can cause InfluxDB to return no results or an `error parsing query` error .

The other options for specifying time durations with `now()` are listed below.  
`u` microseconds  
`ms` milliseconds  
`s` seconds  
`m` minutes  
`h` hours  
`d` days  
`w` weeks   

### Absolute time
---
#### Date time strings
Specify time with date time strings.
Date time strings can take two formats: `YYYY-MM-DD HH:MM:SS.nnnnnnnnn` and  `YYYY-MM-DDTHH:MM:SS.nnnnnnnnnZ`, where the second specification is [RFC3339](https://www.ietf.org/rfc/rfc3339.txt).
Nanoseconds (`nnnnnnnnn`) are optional in both formats.

Examples:

Query data between August 18, 2015 23:00:01.232000000 and September 19, 2015 00:00:00 with the timestamp syntax `YYYY-MM-DD HH:MM:SS.nnnnnnnnn` and  `YYYY-MM-DDTHH:MM:SS.nnnnnnnnnZ`:

```sql
> SELECT water_level FROM h2o_feet WHERE time > '2015-08-18 23:00:01.232000000' AND time < '2015-09-19'
```
```sql
> SELECT water_level FROM h2o_feet WHERE time > '2015-08-18T23:00:01.232000000Z' AND time < '2015-09-19'
```

Query data that occur 6 minutes after September 18, 2015 21:24:00:

```sql
> SELECT water_level FROM h2o_feet WHERE time > '2015-09-18T21:24:00Z' + 6m
```

Things to note about querying with date time strings:

* Single quote the date time string.
InfluxDB returns as error (`ERR: invalid operation: time and *influxql.VarRef are not compatible`) if you double quote the date time string.
* If you only specify the date, InfluxDB sets the time to `00:00:00`.

#### Epoch time
Specify time with timestamps in epoch time.
Epoch time is the number of nanoseconds that have elapsed since 00:00:00 Coordinated Universal Time (UTC), Thursday, 1 January 1970.
Indicate the units of the timestamp at the end of the timestamp (see the section above for a list of acceptable time units).

Examples:

Return all points that occur after  `2014-01-01 00:00:00`:  
```sql
> SELECT * FROM h2o_feet WHERE time > 1388534400s
```

Return all points that occur 6 minutes after `2015-09-18 21:24:00`:

```sql
> SELECT * FROM h2o_feet WHERE time > 24043524m + 6m
```

## Regular expressions in queries

Regular expressions are surrounded by `/` characters and use [Golang's regular expression syntax](http://golang.org/pkg/regexp/syntax/).
Use regular expressions when selecting measurements and tags.

> **Note:** You cannot use regular expressions to match databases, retention policies, or fields.
You can only use regular expressions to match measurements and tags.

In this section we'll be using all of the measurements in the [sample data](/influxdb/v0.13/query_language/data_exploration/#sample-data):
`h2o_feet`, `h2o_quality`, `h2o_pH`, `average_temperature`, and `h2o_temperature`.
Please note that every measurement besides `h2o_feet` is fictional and contains fictional data.

### Regular expressions and selecting measurements
---
Select the oldest point from every measurement in the `NOAA_water_database` database:
```sql
> SELECT * FROM /.*/ LIMIT 1
```

CLI response:
```bash
name: average_temperature
-------------------------
time			               degrees	 index	 level description	    location	     pH	 randtag	 water_level
2015-08-18T00:00:00Z	 82					                               coyote_creek


name: h2o_feet
--------------
time			               degrees	 index	 level description	    location	     pH	 randtag	 water_level
2015-08-18T00:00:00Z			               between 6 and 9 feet	 coyote_creek			            8.12


name: h2o_pH
------------
time			               degrees	 index	 level description	    location	     pH	 randtag	 water_level
2015-08-18T00:00:00Z						                                  coyote_creek	 7


name: h2o_quality
-----------------
time			               degrees	 index	 level description	    location	     pH	 randtag	 water_level
2015-08-18T00:00:00Z		         41				                       coyote_creek		    1


name: h2o_temperature
---------------------
time			               degrees	 index	 level description	    location	     pH	 randtag	 water_level
2015-08-18T00:00:00Z	 60					                               coyote_creek
```

* Alternatively, `SELECT` all of the measurements in `NOAA_water_database` by typing them out and separating each name with a comma , but that could get tedious:

    ```sql
> SELECT * FROM average_temperature,h2o_feet,h2o_pH,h2o_quality,h2o_temperature LIMIT 1
    ```

Select the first three points from every measurement whose name starts with `h2o`:  
```sql
> SELECT * FROM /^h2o/ LIMIT 3
```
CLI response:  
```bash
name: h2o_feet
--------------
time			               degrees	 index	 level description	    location	     pH	randtag	water_level
2015-08-18T00:00:00Z			               between 6 and 9 feet	 coyote_creek			          8.12
2015-08-18T00:00:00Z			               below 3 feet		        santa_monica			          2.064
2015-08-18T00:06:00Z			               between 6 and 9 feet	 coyote_creek			          8.005


name: h2o_pH
------------
time			               degrees	 index	 level description	    location	     pH	randtag	water_level
2015-08-18T00:00:00Z						                                  coyote_creek	 7
2015-08-18T00:00:00Z						                                  santa_monica	 6
2015-08-18T00:06:00Z						                                  coyote_creek	 8


name: h2o_quality
-----------------
time			               degrees	 index	 level description	    location	     pH	randtag	water_level
2015-08-18T00:00:00Z		         99				                       santa_monica		   2
2015-08-18T00:00:00Z		         41				                       coyote_creek		   1
2015-08-18T00:06:00Z		         11				                       coyote_creek		   3


name: h2o_temperature
---------------------
time			               degrees	 index	 level description	    location	     pH	randtag	water_level
2015-08-18T00:00:00Z	 70					                               santa_monica
2015-08-18T00:00:00Z	 60					                               coyote_creek
2015-08-18T00:06:00Z	 60					                               santa_monica

```

Select the first 5 points from every measurement whose name contains `temperature`:

```sql
> SELECT * FROM /.*temperature.*/ LIMIT 5
```

CLI response:
```bash
name: average_temperature
-------------------------
time			              degrees	location
2015-08-18T00:00:00Z	85	     santa_monica
2015-08-18T00:00:00Z	82	     coyote_creek
2015-08-18T00:06:00Z	73	     coyote_creek
2015-08-18T00:06:00Z	74	     santa_monica
2015-08-18T00:12:00Z	86	     coyote_creek

name: h2o_temperature
---------------------
time			              degrees	location
2015-08-18T00:00:00Z	60	     coyote_creek
2015-08-18T00:00:00Z	70	     santa_monica
2015-08-18T00:06:00Z	65	     coyote_creek
2015-08-18T00:06:00Z	60	     santa_monica
2015-08-18T00:12:00Z	68	     coyote_creek
```

### Regular expressions and specifying tags
---
Use regular expressions to specify tags in the `WHERE` clause.
The relevant comparators include:  
`=~` matches against  
`!~` doesn't match against

Select the oldest four points from the measurement `h2o_feet` where the value of the tag `location` does not include an `a`:
```sql
> SELECT * FROM h2o_feet WHERE location !~ /.*a.*/ LIMIT 4
```

CLI response:
```bash
name: h2o_feet
--------------
time			               level description	    location	     water_level
2015-08-18T00:00:00Z	 between 6 and 9 feet 	coyote_creek	 8.12
2015-08-18T00:06:00Z	 between 6 and 9 feet	 coyote_creek	 8.005
2015-08-18T00:12:00Z	 between 6 and 9 feet	 coyote_creek	 7.887
2015-08-18T00:18:00Z	 between 6 and 9 feet	 coyote_creek	 7.762
```

Select the oldest four points from the measurement `h2o_feet` where the value of the tag `location` includes a `y` or an `m` and `water_level` is greater than zero:
```sql
> SELECT * FROM h2o_feet WHERE (location =~ /.*y.*/ OR location =~ /.*m.*/) AND water_level > 0 LIMIT 4
```
or
<br>
<br>
```sql
> SELECT * FROM h2o_feet WHERE location =~ /[ym]/ AND water_level > 0 LIMIT 4
```

CLI response:
```
name: h2o_feet
--------------
time			               level description	    location	     water_level
2015-08-18T00:00:00Z	 between 6 and 9 feet	 coyote_creek	 8.12
2015-08-18T00:00:00Z	 below 3 feet		        santa_monica	 2.064
2015-08-18T00:06:00Z	 between 6 and 9 feet	 coyote_creek	 8.005
2015-08-18T00:06:00Z	 below 3 feet		        santa_monica	 2.116
```

See [the WHERE clause](/influxdb/v0.13/query_language/data_exploration/#the-where-clause) section for an example of how to return data where a tag key has a value and an example of how to return data where a tag key has no value using regular expressions.

