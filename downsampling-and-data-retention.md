InfluxDB 每秒可以处理大量的data points。数据存储如果时间跨度很长的话会导致存储问题。 很自然解决办法是缩减采样数据，精度高的数据仅存储有限的时间，低精度、聚合的数据存储的时间更长或者永远保存。

这篇文章介绍如何通过InfluxDB的retention policies和continuous queries功能自动采样和数据过期。

## **Retention Policies**

### **Definition**

retention policy \(RP\) 是 InfluxDB's 数据结构的一部分，它描述了数据保留的时间\(duration\)和在集群中的数据副本数。一个database可以有多个RPs，每个database的RPs相互独立。

Replication factors do not serve a purpose with single node instances.

### **Purpose**

InfluxDB 设计的时候并没有考虑删除数据，但是基于这种设计的前提假设是很少需要删除数据，并且对性能要求不高。然而，InfluxDB意识到有必要丢弃一些已经失去使用价值的数据，这就是RPs的目的。

### **Working with RPs**

当创建了database的时候，InfluxDB自动创建了名为 `default` 的RP，保留时间无限长， replication factor 为1. `default` 同时作为database的 `DEFAULT` RP。如果你在写point时不指定其他的RP，则默认使用`DEFAULT` RP。

InfluxDB 读和写的时候会自动使用 `DEFAULT` RP 。如果想使用其他的RP，你必须指定measurement的全名，即: `<database_name>."<retention_policy>".<measurement_name>`.

你也可以 create、alter 和 delete RPs和修改 database 的 `DEFAULT` RP。更多见 [Database Management](/database-management.md) 。

> **Clarifying** `default` **vs.** `DEFAULT`
> 
> `default`: InfluxDB创建database时自动生成的RP的name，时长无限，副本数1，同时设置为database的 `DEFAULT` RP，但是这些都是可以修改的
> 
> `DEFAULT`: 没有指定其他的RP时，InfluxDB默认读写的RP

## **Continuous Queries**

### **Definition**

continuous query \(CQ\) 是一个 InfluxQL query，可以在database中定期自动运行 CQs 需要 `SELECT` clause 使用function，并且必须包含 `GROUP BY time()` clause。InfluxDB 将 CQ 的结果存储至特定的 measurement。

### **Purpose**

CQs 是定期 downsampling data 不错的解决方案，创建 CQ之后，InfluxDB 定期自动执行query，和之前返回结果不一样，Influx将结果存储至指定的measurement中。

### **Working with CQs**

下面的内容仅对创建CQ提供了简单的说明，更多信息可以参见 [Continuous Queries](/continuous-queries.md)

## **Combining RPs and CQs - a casestudy**

我们有一份实时数据，记录了餐厅内每10s来自不同phone和不同website的订单量。长时间来看，我们只关心每30分钟的每个phone和webstie的平均值。下面，我们在InfluxDB中使用RPs和CQs实现以下步骤：

* 自动删除2小时前10s一次的数据
* 自动将10s级别的数据聚合成30分钟
* 将30分钟的数据永久保存

下面的步骤中使用了虚构的database `food_data` 、 [measurement](https://github.com/influxdata/docs.influxdata.com/blob/master/influxdb/v0.13/concepts/glossary/#measurement) `orders。` `orders` 有两个field， `phone` 和 `website`, 记录了每10s来自各个渠道的订单数量。

### **Prepare the database**

在写入数据至 database `food_data`之前，我们做以下步骤。

> **Note:** We do this before inserting any data because InfluxDB only performs CQs on new data, that is, data with timestamps that occur after the time at which we create the CQ.

#### **Create a new **`DEFAULT`** RP**

当我们初始化 [created the database](/database-management.md) `food_data`时，InfluxDB自动生成了RP`default` ， `default`也是`food_data`的`DEFAULT` RP。如果没有指定RP的情况下，所有的point都会写入 `default`中并一直保存下去。

我们希望`food_data`中的`DEFAULT` 仅保留2小时，命令如下：

```
> CREATE RETENTION POLICY two_hours ON food_data DURATION 2h REPLICATION 1 DEFAULT
```

That query makes the `two_hours` RP the `DEFAULT` RP in `food_data`. When we write data to the database and do not supply an RP in the write, InfluxDB automatically stores those data in the `two_hours` RP. Once those data have timestamps that are older than two hours, InfluxDB deletes those data. For a more detailed discussion on the `CREATE RETENTION POLICY` syntax, see [Database Management](https://github.com/influxdata/docs.influxdata.com/blob/master/influxdb/v0.13/query_language/database_management/#retention-policy-management).

To clarify, we've included the results from the `SHOW RETENTION POLICIES` query below. Notice that there are two RPs in `food_data` \(`default` and `two_hours`\) and that the third column identifies `two_hours` as the `DEFAULT` RP.

```
> SHOW RETENTION POLICIES ON food_data
name              duration    replicaN    default
default        0                1               false
two_hours     2h0m0s           1                true
```

#### **Create the CQ**

Now we create a CQ that automatically downsamples the 10 second level data to 30 minute level data:

```
> CREATE CONTINUOUS QUERY cq_30m ON food_data BEGIN SELECT mean(website) AS mean_website,mean(phone) AS mean_phone INTO food_data."default".downsampled_orders FROM orders GROUP BY time(30m) END
```

That CQ makes InfluxDB automatically and periodically calculate the 30 minute average from the 10 second website order data and the 30 minute average from the 10 second phone order data. InfluxDB also writes the CQ's results into the measurement `downsampled_orders` and to the RP `default`; InfluxDB stores the aggregated data in`downsampled_orders` forever.

> **Note:** You must specify the RP in the `INTO` clause to write the results of the query to an RP other than the`DEFAULT` RP. In the CQ above, we write the results of the query to the infinite RP `default` by fully qualifying the measurement. To fully qualify a measurement, specify its database and RP with `<database_name>."<retention_policy>".<measurement_name>`. If you do not fully qualify the measurement, InfluxDB writes the results of the query to the two hour RP `DEFAULT`.

For a more detailed discussion on the `CREATE CONTINUOUS QUERY` syntax, see [Continuous Queries](https://github.com/influxdata/docs.influxdata.com/blob/master/influxdb/v0.13/query_language/continuous_queries).

### **Write the data to InfluxDB and see the results**

Now that we've prepped `food_data`, we start writing the data to InfluxDB and let things run for a bit. After a while, we see that the database has two measurements: `orders` and `downsampled_orders`.

A sample of the oldest data in `orders` - these are the raw 10 second data subject to the `two_hours` RP:

```
> SELECT * FROM orders LIMIT 5
name: orders
-----------------
time                                    phone   website
2015-12-04T20:00:11Z     1       6
2015-12-04T20:00:20Z        9        10
2015-12-04T20:00:30Z        2        17
2015-12-04T20:00:40Z        3        10
2015-12-04T20:00:50Z        1        15
```

We submitted this query on 12\/04\/2015 at 22:08:19 UTC - notice that the oldest data have timestamps that are no older than around two hours ago.

> **Note:** By default, InfluxDB checks to enforce an RP every 30 minutes so you may have data that are older than two hours between checks. The rate at which InfluxDB checks to enforce an RP is a configurable setting, see[Database Configuration](https://github.com/influxdata/docs.influxdata.com/blob/master/influxdb/v0.13/administration/config/#retention).

A sample of the oldest data in `downsampled_orders` - these are the aggregated data subject to the `default` RP:

```
> SELECT * FROM food_data."default".downsampled_orders LIMIT 5
name: downsampled_orders
------------------------
time                           mean_phone              mean_website
2015-12-03T22:30:00Z     4.318181818181818   9.254545454545454
2015-12-03T23:00:00Z     4.266666666666667   9.827777777777778
2015-12-03T23:30:00Z     4.766666666666667   9.677777777777777
2015-12-04T00:00:00Z     4.405555555555556   8.5
2015-12-04T00:30:00Z     4.788888888888889   9.383333333333333
```

Notice that the timestamps in `downsampled_orders` occur at 30 minute intervals and that the measurement has timestamps that are older than those in the `orders` measurement. The data in `downsampled_orders` aren't subject to the `two_hours` RP.

> **Note:** You must specify the RP in your query to select data that are subject to an RP other than the `DEFAULT` RP. In the second `SELECT` statement, we get the CQ results by fully qualifying the measurement. To fully qualify a measurement, specify its database and RP with `<database_name>."<retention_policy>".<measurement_name>`.

Using a combination of RPs and CQs, we've made InfluxDB automatically downsample data and expire old data. Now that you have a general understanding of how these features can work together, we recommend looking at the detailed documentation on [CQs](https://github.com/influxdata/docs.influxdata.com/blob/master/influxdb/v0.13/query_language/continuous_queries) and [RPs](https://github.com/influxdata/docs.influxdata.com/blob/master/influxdb/v0.13/query_language/database_management/#retention-policy-management) to see all that they can do for you.

