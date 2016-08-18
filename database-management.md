InfluxQL 提供了完整的 administrative commands。

* [Data management](#data-management)

  ◦ [Create a database with ](#create-a-database-with-create-database)`CREATE DATABASE`

  ◦ [Delete a database with ](#delete-a-database-with-drop-database)`DROP DATABASE`

  ◦ [Drop series from the index with ](#drop-series-from-the-index-with-drop-series)`DROP SERIES`

  ◦ [Delete series with ](#delete-series-with-delete)`DELETE`

  ◦ [Delete measurements with ](#delete-measurements-with-drop-measurement)`DROP MEASUREMENT`

  ◦ [Delete a shard with ](#delete-a-shard-with-drop-shard)`DROP SHARD`

* [Retention policy management](#retention-policy-management)

  ◦ [Create retention policies with ](#create-retention-policies-with-create-retention-policy)`CREATE RETENTION POLICY`

  ◦ [Modify retention policies with ](#modify-retention-policies-with-alter-retention-policy)`ALTER RETENTION POLICY`

  ◦ [Delete retention policies with ](#delete-retention-policies-with-drop-retention-policy)`DROP RETENTION POLICY`


如果你在寻找 `SHOW` 查询 \(比如, `SHOW DATABASES` 或者 `SHOW RETENTION POLICIES`\), 见 [Schema Exploration](/schema-exploration.md).

这片文章中的例子使用 InfluxDB 的 [Command Line Interface \(CLI\)](https://github.com/influxdata/docs.influxdata.com/blob/master/influxdb/v0.13/introduction/getting_started) 。你还可以通过HTTP API之行这些命令，向`/query`endpoint中发送 `GET` 请求，命令放在URL参数 `q`中，见 [Querying Data](https://github.com/influxdata/docs.influxdata.com/blob/master/influxdb/v0.13/guides/querying_data) 中更多使用 HTTP API的例子。

> **Note:** 当认证开启时，仅admin可以执行这篇文章中的大部分命令，更多信息见 [authentication and authorization](https://github.com/influxdata/docs.influxdata.com/blob/master/influxdb/v0.13/administration/authentication_and_authorization)

## **Data Management**

### **Create a database with CREATE DATABASE**

---

`CREATE DATABASE` 的语法如下：

```
CREATE DATABASE [IF NOT EXISTS] <database_name> [WITH [DURATION <duration>] [REPLICATION <n>] [SHARD DURATION <duration>] [NAME <retention-policy-name>]]
```

> **Note:** `IF NOT EXISTS` 现在没什么鸟用，将在 InfluxDB version 1.0移除。

创建 database `NOAA_water_database`：

```
> CREATE DATABASE NOAA_water_database
>
```

创建 database `NOAA_water_database` 同时使用名为`liquid`的`DEFAULT` retention policy：

```
> CREATE DATABASE NOAA_water_database WITH DURATION 3d REPLICATION 3 SHARD DURATION 30m NAME liquid
>
```

When specifying a retention policy you can include one or more of the attributes `DURATION`, `REPLICATION`, `SHARD DURATION`, and `NAME`. For more on retention policies, see [Retention Policy Management](#retention-policy-management)

`CREATE DATABASE` 执行成功之后不返回任何结果，如果你想尝试已经存在的database，InfluxDB不会返回错误。

### **Delete a database with DROP DATABASE**

---

`DROP DATABASE` 删除指定database中所有的 data, measurements, series, continuous queries 和 retention policies 。语法如下：

```
DROP DATABASE [IF EXISTS] <database_name>
```

> **Note:** `IF EXISTS` 同上，没鸟用

删除 database NOAA\_water\_database:

```
> DROP DATABASE NOAA_water_database
>
```

A successful `DROP DATABASE` query returns an empty result. If you attempt to drop a database that does not exist, InfluxDB does not return an error.

### **Drop series from the index with DROP SERIES**

---

`DROP SERIES` 通过索引删除 [series](https://github.com/influxdata/docs.influxdata.com/blob/master/influxdb/v0.13/concepts/glossary/#series) 中的所有point。

> **Note:** `DROP SERIES` 在 `WHERE` clause中不支持time interval， 如需这样的功能，见 `DELETE` 。

查询与法如下，你必须指定 `FROM` clause 或者 `WHERE` clause：

```
DROP SERIES FROM <measurement_name[,measurement_name]> WHERE <tag_key>='<tag_value>'
```

删除指定measurement中所有的series：

```
> DROP SERIES FROM h2o_feet
```

删除指定measurement中的单个series：

```
> DROP SERIES FROM h2o_feet WHERE location = 'santa_monica'
```

删除database中指定tag的所有series：

```
> DROP SERIES WHERE location = 'santa_monica'
```

`DROP SERIES` 执行成功之后返回空结果。

### **Delete series with DELETE**

---

`DELETE` 删除series中的所有point。和`DROP SERIES`不一样，它不从series中删除索引，并且在`WHERE`中支持time interval。

查询必须包含 `FROM` clause 或者 `WHERE` clause：

```
DELETE FROM <measurement_name> WHERE [<tag_key>='<tag_value>'] | [<time interval>]
```

删除 measurement `h2o_feet`相关的所有数据：

```
> DELETE FROM h2o_feet
```

删除measurement `h2o_quality` 中所有`randtag=3` 的point：

```
> DELETE FROM h2o_quality WHERE randtag = '3'
```

删除`2016-01-01`之前的所有的point：

```
> DELETE WHERE time < '2016-01-01' 
```

`DELETE` 查询成功之后返回为空：

关于 `DELETE`值得注意的：

* `DELETE` 当指定measurement name并且`WHERE`中指定tag时，`FROM` clause中可以使用正则表达式
* `DELETE` 的`WHERE` clause不支持field

### **Delete measurements with DROP MEASUREMENT**

---

`DROP MEASUREMENT` 删除了指定measurement的所有的数据和series，并且删除了索引。

语法如下：

```
DROP MEASUREMENT <measurement_name>
```

删除measurement `h2o_feet`:

```
> DROP MEASUREMENT h2o_feet
```

> **Note:** `DROP MEASUREMENT` 删除measurement中的所有data和series。但是并不删除相关联的continuous queries。

`DROP MEASUREMENT` 执行成功后返回结果为空。

目前，InfluxDB 使用`DROP MEASUREMENTS`不支持正则表达式。详见GitHub Issue [\#4275](https://github.com/influxdb/influxdb/issues/4275)

### **Delete a shard with DROP SHARD**

---

The `DROP SHARD` query deletes a shard. It also drops the shard from the [metastore](https://github.com/influxdata/docs.influxdata.com/blob/master/influxdb/v0.13/concepts/glossary/#metastore). The query takes the following form:

```
DROP SHARD <shard_id_number>
```

Delete the shard with the id `1`:

```
> DROP SHARD 1
>

```

A successful `DROP SHARD` query returns an empty result. InfluxDB does not return an error if you attempt to drop a shard that does not exist.

## **Retention Policy Management**

The following sections cover how to create, alter, and delete retention policies. Note that when you create a database, InfluxDB automatically creates a retention policy named `default` which has infinite retention. You may disable that auto-creation in the configuration file.

### **Create retention policies with CREATE RETENTION POLICY**

---

The `CREATE RETENTION POLICY` query takes the following form:

```
CREATE RETENTION POLICY <retention_policy_name> ON <database_name> DURATION <duration> REPLICATION <n> [SHARD DURATION <duration>] [DEFAULT]
```

* `DURATION` determines how long InfluxDB keeps the data. The options for specifying the duration of the retention policy are listed below. Note that the minimum retention period is one hour.

  `m` minutes

  `h` hours

  `d` days

  `w` weeks

  `INF` infinite


Currently, the `DURATION` attribute supports only single units. For example, you cannot express the duration`7230m` as `120h 30m`. See GitHub Issue [\#3634](https://github.com/influxdb/influxdb/issues/3634) for more information.

* `REPLICATION` determines how many independent copies of each point are stored in the cluster, where `n` is the number of data nodes.

Replication factors do not serve a purpose with single node instances.

* `SHARD DURATION` determines the time range covered by a shard group. The options for specifying the duration of the shard group are listed below. The default shard group duration depends on your retention policy's `DURATION`.

  `u` microseconds

  `ms` milliseconds

  `s` seconds

  `m` minutes

  `h` hours

  `d` days

  `w` weeks


Currently, the `SHARD DURATION` attribute supports only single units. For example, you cannot express the duration`7230m` as `120h 30m`.

* `DEFAULT` sets the new retention policy as the default retention policy for the database.

Create a retention policy called `one_day_only` for the database `NOAA_water_database` with a one day duration and a replication factor of one:

```
> CREATE RETENTION POLICY one_day_only ON NOAA_water_database DURATION 1d REPLICATION 1
>
```

Create the same retention policy as the one in the example above, but set it as the default retention policy for the database.

```
> CREATE RETENTION POLICY one_day_only ON NOAA_water_database DURATION 1d REPLICATION 1 DEFAULT
>
```

A successful `CREATE RETENTION POLICY` query returns an empty response. If you attempt to create a retention policy that already exists, InfluxDB does not return an error.

> **Note:** You can also specify a new retention policy in the `CREATE DATABASE` query. See [Create a database with CREATE DATABASE](https://github.com/influxdata/docs.influxdata.com/blob/master/influxdb/v0.13/query_language/database_management/#create-a-database-with-create-database).

### **Modify retention policies with ALTER RETENTION POLICY**

---

The `ALTER RETENTION POLICY` query takes the following form, where you must declare at least one of the retention policy attributes `DURATION`, `REPLICATION`, `SHARD DURATION`, or `DEFAULT`:

```
ALTER RETENTION POLICY <retention_policy_name> ON <database_name> DURATION <duration> REPLICATION <n> SHARD DURATION <duration> DEFAULT
```

Replication factors do not serve a purpose with single node instances.

First, create the retention policy `what_is_time` with a `DURATION` of two days:

```
> CREATE RETENTION POLICY what_is_time ON NOAA_water_database DURATION 2d REPLICATION 1
>
```

Modify `what_is_time` to have a three week `DURATION`, a 30 minute shard group duration, and make it the `DEFAULT`retention policy for `NOAA_water_database`.

```
> ALTER RETENTION POLICY what_is_time ON NOAA_water_database DURATION 3w SHARD DURATION 30m DEFAULT
>
```

In the last example, `what_is_time` retains its original replication factor of 1.

A successful `ALTER RETENTION POLICY` query returns an empty result.

### **Delete retention policies with DROP RETENTION POLICY**

Delete all measurements and data in a specific retention policy with:

```
DROP RETENTION POLICY <retention_policy_name> ON <database_name>
```

Delete the retention policy `what_is_time` in the `NOAA_water_database` database:

```
> DROP RETENTION POLICY what_is_time ON NOAA_water_database
>
```

A successful `DROP RETENTION POLICY` query returns an empty result. If you attempt to drop a retention policy that does not exist, InfluxDB does not return an error.

* [Contact](https://github.com/contact)

