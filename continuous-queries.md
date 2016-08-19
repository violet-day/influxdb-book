当InfluxDB中写入了大量的数据之后，你可能会对原始数据做一些采样。通过`GROUP BY time()` 和 InfluxQL [function](https://github.com/influxdata/docs.influxdata.com/blob/master/influxdb/v0.13/query_language/functions) 讲一些高频的数据转化为低频的数据。一直重复的通过query执行这些操作略蠢。InfluxDB 的 continuous queries \(CQ\) 很容易处理这样的事情，CQs会定期自动将查询结果写至另外一个measurement。

* [CQ definition](#cq-definition)
* [InfluxQL for creating a CQ](#influxql-for-creating-a-cq)

  ◦ [The ](#the-create-continuous-query-statement)`CREATE CONTINUOUS QUERY`[ statement](https://github.com/influxdata/docs.influxdata.com/blob/master/influxdb/v0.13/query_language/continuous_queries/#the-create-continuous-query-statement)

  ◦ [CQs with backreferencing](#cqs-with-backreferencing)


* [List CQs with ](#list-cqs-with-show)`SHOW`
* [Delete CQs with ](#delete-cqs-with-drop)`DROP`
* [Backfilling](#backfilling)

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

CQ 属于某个database。通过`<database_name>` 指定属于的database，`RESAMPLE` clause决定了执行的频率，通过`FOR <interval>`决定执行时间范围。如果需要包含 `RESAMPLE` clause，则必须指定  `EVERY`或`FOR`或全部。没有`RESAMPLE` clause,时，InfluxDB 执行的间隔同 `GROUP BY time()`interval 一致，并且计算最近的`GROUP BY time()` 区间 \(即`now()` 和 `now()`减去 `GROUP BY time()` interval之间的部分\)。

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

* 创建CQ时使用一个function：

  ```
  > CREATE CONTINUOUS QUERY minnie ON world BEGIN SELECT min(mouse) INTO min_mouse FROM zoo GROUP BY time(30m) END
  ```

  执行之后， InfluxDB会自动计算 measurement`zoo`的field `mouse`每30分钟内的最小值，并将结果写入measurement `min_mouse`。请注意CQ `minnie` 仅存在于database `world`中

* 创建CQ时使用一个function 并将结果卸乳另外一个 [retention policy](https://github.com/influxdata/docs.influxdata.com/blob/master/influxdb/v0.13/concepts/glossary/#retention-policy-rp)

  ```
  > CREATE CONTINUOUS QUERY minnie_jr ON world BEGIN SELECT min(mouse) INTO world."7days".min_mouse FROM world."1day".zoo GROUP BY time(30m) END
  ```

  CQ `minnie_jr` 的作用方式和CQ `minnie`一样，InfluxDB用于计算的measurement `zoo`位于retention policy `1day` ，计算的结果位于 retention policy `7days`.

  将CQs 和 retention policies组合使用提供了自动的方式对数据取样并使没有必要的原始数据自动过期。关于这种方式的更多信息，见 [Downsampling and Data Retention](/downsampling-and-data-retention.md).

* 创建CQ时使用2个function：

  ```
  > CREATE CONTINUOUS QUERY minnie_maximus ON world BEGIN SELECT min(mouse),max(imus) INTO min_max_mouse FROM zoo GROUP BY time(30m) END
  ```

  CQ `minnie_maximus` 每30分钟自动计算了 field `mouse` 和 field `imus` \(字段都在 measurement `zoo`中\)，并将结果写入 measurement `min_max_mouse`

* 创建CQ时使用2个function，并在结果自定义field key：

  ```
  > CREATE CONTINUOUS QUERY minnie_maximus_1 ON world BEGIN SELECT min(mouse) AS minuscule,max(imus) AS monstrous INTO min_max_mouse FROM zoo GROUP BY time(30m) END
  ```

  CQ `minnie_maximus_1` 的方式和 `minnie_maximus`一样, 然而InfluxDB 使用`miniscule` 和 `monstrous` 作为目标 measurement name。更多 `AS`信息，见[Functions](/functions.md)

* 创建CQ 时使用 30分钟 `GROUP BY time()` 作为interval ，15分钟运行一次：

  ```
  > CREATE CONTINUOUS QUERY vampires ON transylvania RESAMPLE EVERY 15m BEGIN SELECT count(dracula) INTO vampire_populations FROM raw_vampires GROUP BY time(30m) END
  ```

  如果没有 `RESAMPLE EVERY 15m`, `vampires` 将会每30分钟运行一次，和 `GROUP BY time()`interval一致

* 创建CQ 时使用 30分钟 `GROUP BY time()` 作为interval ，30分钟运行一次，对1小时之前的数据做 `GROUP BY time()` 并计算：

  ```
  > CREATE CONTINUOUS QUERY vampires_1 ON transylvania RESAMPLE FOR 60m BEGIN SELECT count(dracula) INTO vampire_populations_1 FROM raw_vampires GROUP BY time(30m) END
  ```

  InfluxDB 每30分钟执行 `vampires_1` 一次， \(和`GROUP BY time()` interval一致\) ，但是每次其实是执行两次查询，第一次是 `now()` 和 `now() - 30m` 之间，第二次是 `now() - 30m` 和 `now() - 60m`之间。没有 `RESAMPLE` clause的情况下， InfluxDB将30分钟执行一次，查询范围为`now()` 和 `now() - 30m`之间

* 创建CQ 时使用 30分钟 `GROUP BY time()` 作为interval，每15分钟运行一次，计算最近1小时内的每个`GROUP BY time()` ：

  ```
  > CREATE CONTINUOUS QUERY vampires_2 ON transylvania RESAMPLE EVERY 15m FOR 60m BEGIN SELECT count(dracula) INTO vampire_populations_2 FROM raw_vampires GROUP BY time(30m) END
  ```

  `vampires_2` 15分钟运行一次，每次查询2次，一次是 `now()` 和`now() - 30m` 之间，第二次是 `now() - 30m` 和 `now() - 60m`


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

CQs 并不backfill数据，在CQ存在之前，它们不会计算数据。然而，用户可以使用 `INTO` clause回填数据。和CQ不一样的是，回填的查询需要 `WHERE` clause 和`time` 限制

### **Examples**

基本的backfill的例子：

```
> SELECT min(temp) as min_temp, max(temp) as max_temp INTO "reading.minmax.5m" FROM reading
WHERE time >= '2015-12-14 00:05:20' AND time < '2015-12-15 00:05:20'
GROUP BY time(5m)
```

Tags \(下例为`sensor_id`\) 可以和CQs一样作为可选项使用：

```
> SELECT min(temp) as min_temp, max(temp) as max_temp INTO "reading.minmax.5m" FROM reading
WHERE time >= '2015-12-14 00:05:20' AND time < '2015-12-15 00:05:20'
GROUP BY time(5m), sensor_id
```

为了避免backfill时大量仅包含`null` values的 "empty" points , 可以在query的最后使用[fill\(\)](https://github.com/influxdata/docs.influxdata.com/blob/master/influxdb/v0.13/query_language/data_exploration/#the-group-by-clause-and-fill)

```
> SELECT min(temp) as min_temp, max(temp) as max_temp INTO "reading.minmax.5m" FROM reading
WHERE time >= '2015-12-14 00:05:20' AND time < '2015-12-15 00:05:20'
GROUP BY time(5m), fill(none)
```

如果你想更细一步的在执行query，可以添加`WHERE` clauses：

```
> SELECT min(temp) as min_temp, max(temp) as max_temp INTO "reading.minmax.5m" FROM reading
WHERE sensor_id="EG-21442" AND time >= '2015-12-14 00:05:20' AND time < '2015-12-15 00:05:20'
GROUP BY time(5m)
```

