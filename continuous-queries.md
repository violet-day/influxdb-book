当InfluxDB中写入了大量的数据之后，你可能会对原始数据做一些采样。通过`GROUP BY time()` 和 InfluxQL [function](https://github.com/influxdata/docs.influxdata.com/blob/master/influxdb/v0.13/query_language/functions) 讲一些高频的数据转化为低频的数据。一直重复的通过query执行这些操作略蠢。InfluxDB 的 continuous queries \(CQ\) 很容易处理这样的事情，CQs会定期自动将查询结果写至另外一个measurement。

* [CQ definition](#cq-definition)
* [InfluxQL for creating a CQ](#influxql-for-creating-a-cq)

  ◦ [The ](#the-create-continuous-query-statement)`CREATE CONTINUOUS QUERY`[ statement](https://github.com/influxdata/docs.influxdata.com/blob/master/influxdb/v0.13/query_language/continuous_queries/#the-create-continuous-query-statement)

  ◦ [CQs with backreferencing](#cqs-with-backreferencing)


* [List CQs with ](#list-cqs-with-show)`SHOW`
* [Delete CQs with ](#delete-cqs-with-drop)`DROP`
* [Backfilling](#backfilling)
* [Further reading](#further-reading)

## **CQ definition**

CQ是在database中定期自动执行的InfluxQL query。 InfluxDB 将查询结果存储至指定的 [measurement](https://github.com/influxdata/docs.influxdata.com/blob/master/influxdb/v0.13/concepts/glossary/#measurement). CQs的 `SELECT` clause需要包含函数，并且必须包含 `GROUP BY time()` clause

CQs 并不维持状态。每个CQ的执行都是独立的，会对匹配的点做采样。

CQ的执行由database内部调度，目前么有办法让用户自己控制开始或者结束的时间。

仅 admin users 允许使用 continuous queries。更多关于用户权限的信息，见 [Authentication and Authorization](https://github.com/influxdata/docs.influxdata.com/blob/master/influxdb/v0.13/administration/authentication_and_authorization/#user-types-and-their-privileges).

> **Note:** CQs 仅仅对CQ创建之后的data做处理。如果你想对在CQ创建之前的数据做采样，见 [Data Exploration](https://github.com/influxdata/docs.influxdata.com/blob/master/influxdb/v0.13/query_language/data_exploration/#downsample-data)

## **InfluxQL for creating a CQ**

### **The **`CREATE CONTINUOUS QUERY`** statement**

```
CREATE CONTINUOUS QUERY <cq_name> ON <database_name> [RESAMPLE [EVERY <interval>] [FOR <interval>]] BEGIN SELECT <function>(<stuff>)[,<function>(<stuff>)] INTO <different_measurement> FROM <current_measurement> [WHERE <stuff>] GROUP BY time(<interval>)[,<stuff>] END
```

`CREATE CONTINUOUS QUERY` statement 本质上是被`CREATE CONTINUOUS QUERY [...] BEGIN` 和 `END` 包裹的 InfluxQL query 。下面会将CQ statement 分成 meta \(`CREATE` 和 `BEGIN`之间\) 和 query \(`BEGIN` 和 `END`\)。

`CREATE CONTINUOUS QUERY` 执行成功之后，返回为空。如果你想创建一个已经存在的CQ，InfluxDB并不会返回任何错误

#### **Meta syntax:**

```
CREATE CONTINUOUS QUERY ON <database_name> [RESAMPLE [EVERY <interval>] [FOR <interval>]] 
```

CQ 属于某个database。通过`<database_name>` 指定属于的database

The optional `RESAMPLE` clause determines how often InfluxDB runs the CQ \(`EVERY <interval>`\) and the time range over which InfluxDB runs the CQ \(`FOR <interval>`\). If included, the `RESAMPLE` clause must specify either `EVERY`, or`FOR`, or both. Without the `RESAMPLE` clause, InfluxDB runs the CQ at the same interval as the `GROUP BY time()`interval and it calculates the query for the most recent `GROUP BY time()` interval \(that is, where time is between`now()` and `now()` minus the `GROUP BY time()` interval\).

#### **Query syntax:**

```
BEGIN SELECT <function>(<stuff>)[,<function>(<stuff>)] INTO <different_measurement> FROM <current_measurement> [WHERE <stuff>] GROUP BY time(<interval>)[,<stuff>] END 
```

上述的query部分和一般的 `SELECT [...] GROUP BY (time)` 有下面两个区别：

1. `INTO` clause：这里指定了查询结果所存放的measurement

2. `WHERE` clause: 因为 CQs 按照时间递增的规律运行，所以你不需要也不应该在 `WHERE` clause中指定时间范围。如果使用 `WHERE` clause，可以对 tags 进行一些过滤。


> **Note:** 如果在 CQ's `SELECT` clause使用了tag，InfluxDB 将`<current_measurement>` 中的tag转换为`<different_measurement>`中的filed。如果想在`<different_measurement>`保留tag，在CQ中的 `GROUP BY`添加 tag key即可。
> 
> 如果同时在 `SELECT` clause 和`GROUP BY` clause中使用相同的tag，你将不能查询`<different_measurement>`. 更多见GitHub Issue [\#4630](https://github.com/influxdata/influxdb/issues/4630)

#### **CQ examples:**

* 创建CQ使用一个function：

  ```
  > CREATE CONTINUOUS QUERY minnie ON world BEGIN SELECT min(mouse) INTO min_mouse FROM zoo GROUP BY time(30m) END
  ```

  Once executed, InfluxDB automatically calculates the 30 minute minimum of the field `mouse` in the measurement`zoo`, and it writes the results to the measurement `min_mouse`. Note that the CQ `minnie` only exists in the database `world`.

* Create a CQ with one function and write the results to another [retention policy](https://github.com/influxdata/docs.influxdata.com/blob/master/influxdb/v0.13/concepts/glossary/#retention-policy-rp):

  ```
  > CREATE CONTINUOUS QUERY minnie_jr ON world BEGIN SELECT min(mouse) INTO world."7days".min_mouse FROM world."1day".zoo GROUP BY time(30m) END
  ```

  The CQ `minnie_jr` acts in the same way as the CQ `minnie`, however, InfluxDB calculates the 30 minute minimum of the field `mouse` in the measurement `zoo` and under the retention policy `1day`, and it automatically writes the results of the query to the measurement `min_mouse` under the retention policy `7days`.

  Combining CQs and retention policies provides a useful way to automatically downsample data and expire the unnecessary raw data. For a complete discussion on this topic, see [Downsampling and Data Retention](https://github.com/influxdata/docs.influxdata.com/blob/master/influxdb/v0.13/guides/downsampling_and_retention).

* Create a CQ with two functions:

  ```
  > CREATE CONTINUOUS QUERY minnie_maximus ON world BEGIN SELECT min(mouse),max(imus) INTO min_max_mouse FROM zoo GROUP BY time(30m) END
  ```

  The CQ `minnie_maximus` automatically calculates the 30 minute minimum of the field `mouse` and the 30 minute maximum of the field `imus` \(both fields are in the measurement `zoo`\), and it writes the results to the measurement `min_max_mouse`.

* Create a CQ with two functions and personalize the [field keys](https://github.com/influxdata/docs.influxdata.com/blob/master/influxdb/v0.13/concepts/glossary/#field-key) in the results:

  ```
  > CREATE CONTINUOUS QUERY minnie_maximus_1 ON world BEGIN SELECT min(mouse) AS minuscule,max(imus) AS monstrous INTO min_max_mouse FROM zoo GROUP BY time(30m) END
  ```

  The CQ `minnie_maximus_1` acts in the same way as `minnie_maximus`, however, InfluxDB names field keys`miniscule` and `monstrous` in the destination measurement instead of `min` and `max`. For more on `AS`, see[Functions](https://github.com/influxdata/docs.influxdata.com/blob/master/influxdb/v0.13/query_language/functions/#rename-the-output-column-s-title-with-as).

* Create a CQ with a 30 minute `GROUP BY time()` interval that runs every 15 minutes:

  ```
  > CREATE CONTINUOUS QUERY vampires ON transylvania RESAMPLE EVERY 15m BEGIN SELECT count(dracula) INTO vampire_populations FROM raw_vampires GROUP BY time(30m) END
  ```

  Without `RESAMPLE EVERY 15m`, `vampires` would run every 30 minutes - the same interval as the `GROUP BY time()`interval.

* Create a CQ with a 30 minute `GROUP BY time()` interval that runs every 30 minutes and computes the query for all `GROUP BY time()` intervals within the last hour:

  ```
  > CREATE CONTINUOUS QUERY vampires_1 ON transylvania RESAMPLE FOR 60m BEGIN SELECT count(dracula) INTO vampire_populations_1 FROM raw_vampires GROUP BY time(30m) END
  ```

  InfluxDB runs `vampires_1` every 30 minutes \(the same interval as the `GROUP BY time()` interval\) and it computes two queries per run: one where time is between `now()` and `now() - 30m` and one where time is between `now() - 30m` and `now() - 60m`. Without the `RESAMPLE` clause, InfluxDB would compute the query for only one 30 minute interval, that is, where time is between `now()` and `now() - 30m`.

* Create a CQ with a 30 minute `GROUP BY time()` interval that runs every 15 minutes and computes the query for all `GROUP BY time()` intervals within the last hour:

  ```
  > CREATE CONTINUOUS QUERY vampires_2 ON transylvania RESAMPLE EVERY 15m FOR 60m BEGIN SELECT count(dracula) INTO vampire_populations_2 FROM raw_vampires GROUP BY time(30m) END
  ```

  `vampires_2` runs every 15 minutes and computes two queries per run: one where time is between `now()` and`now() - 30m` and one where time is between `now() - 30m` and `now() - 60m`


### **CQs with backreferencing**

在`INTO` statement 中使用 `:MEASUREMENT` 来反向引用measurement names：

```
CREATE CONTINUOUS QUERY <cq_name> ON <database_name> BEGIN SELECT <function>(<stuff>)[,<function>(<stuff>)] INTO <database_name>.<retention_policy>.:MEASUREMENT FROM </relevant_measurement(s)/> [WHERE <stuff>] GROUP BY time(<interval>)[,<stuff>] END
```

_CQ backreferencing example:_

```
> CREATE CONTINUOUS QUERY elsewhere ON fantasy BEGIN SELECT mean(value) INTO reality."default".:MEASUREMENT FROM /elf/ GROUP BY time(10m) END
```

CQ `elsewhere` 每10分钟自动计算database `fantasy`中`elf` measurement的field `value`的平均值。它将计算的结果写入已经存在的database`reality`，并在`fantasy`保留了measurement name。

A sample of the data in `fantasy`:

```
> SHOW MEASUREMENTS
name: measurements
------------------
name
elf1
elf2
wizard
>
> SELECT * FROM elf1
name: cpu_usage_idle
--------------------
time                           value
2015-12-19T01:15:30Z     97.76333874796951
2015-12-19T01:15:40Z     98.3129217695576
[...]
2015-12-19T01:36:00Z     94.71778221778222
2015-12-19T01:35:50Z     87.8
```

A sample of the data in `reality` after `elsewhere` runs for a bit:

```
> SHOW MEASUREMENTS
name: measurements
------------------
name
elf1
elf2
>
> SELECT * FROM elf1
name: elf1
--------------------
time                           mean
2015-12-19T01:10:00Z     97.11668879244841
2015-12-19T01:20:00Z     94.50035091670394
2015-12-19T01:30:00Z     95.99739053789172
```

## **List CQs with **`SHOW`

显示database中的CQ：

```
SHOW CONTINUOUS QUERIES
```

_Example:_

```
> SHOW CONTINUOUS QUERIES
name: reality
-------------
name    query

name: fantasy
-------------
name             query
elsewhere    CREATE CONTINUOUS QUERY elsewhere ON fantasy BEGIN SELECT mean(value) INTO reality."default".:MEASUREMENT FROM fantasy."default"./cpu/ WHERE cpu = 'cpu-total' GROUP BY time(10m) END
```

结果显示了 database `reality` 没有 CQs，database `fantasy` 有一个CQ`elsewhere`.

## **Delete CQs with **`DROP`

从指定的database删除CQ：

```
DROP CONTINUOUS QUERY <cq_name> ON <database_name>
```

_Example:_

```
> DROP CONTINUOUS QUERY elsewhere ON fantasy
>
```

`DROP CONTINUOUS QUERY`执行成功之后返回结果为空

## **Backfilling**

CQs do not backfill data, that is, they do not compute results for data written to the database before the CQ existed. Instead, users can backfill data with the `INTO` clause. Unlike CQs, backfill queries require a `WHERE` clause with a`time` restriction.

### **Examples**

Here is a basic backfill example:

```
> SELECT min(temp) as min_temp, max(temp) as max_temp INTO "reading.minmax.5m" FROM reading
WHERE time >= '2015-12-14 00:05:20' AND time < '2015-12-15 00:05:20'
GROUP BY time(5m)
```

Tags \(`sensor_id` in the example below\) can be used optionally in the same way as in CQs:

```
> SELECT min(temp) as min_temp, max(temp) as max_temp INTO "reading.minmax.5m" FROM reading
WHERE time >= '2015-12-14 00:05:20' AND time < '2015-12-15 00:05:20'
GROUP BY time(5m), sensor_id
```

To prevent the backfill from creating a huge number of "empty" points containing only `null` values, [fill\(\)](https://github.com/influxdata/docs.influxdata.com/blob/master/influxdb/v0.13/query_language/data_exploration/#the-group-by-clause-and-fill) can be used at the end of the query:

```
> SELECT min(temp) as min_temp, max(temp) as max_temp INTO "reading.minmax.5m" FROM reading
WHERE time >= '2015-12-14 00:05:20' AND time < '2015-12-15 00:05:20'
GROUP BY time(5m), fill(none)
```

If you would like to further break down the queries and run them with even more control, you can add additional`WHERE` clauses:

```
> SELECT min(temp) as min_temp, max(temp) as max_temp INTO "reading.minmax.5m" FROM reading
WHERE sensor_id="EG-21442" AND time >= '2015-12-14 00:05:20' AND time < '2015-12-15 00:05:20'
GROUP BY time(5m)
```

## **Further reading**

Now that you know how to create CQs with InfluxDB, check out [Downsampling and Data Retention](https://github.com/influxdata/docs.influxdata.com/blob/master/influxdb/v0.13/guides/downsampling_and_retention) for how to combine CQs with retention policies to automatically downsample data and expire unnecessary data.

