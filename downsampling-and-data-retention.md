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

上面命令，创建了`two_hours` RP 作为`food_data` 的`DEFAULT` RP 。当写入的时候没有指定其他RP的时候，InfluxDB默认将数据存储至`two_hours` RP。一旦存储数据的时间戳大于2小时，InfluxDB就会删除这些数据。更多关于`CREATE RETENTION POLICY` 的语法，见 [Database Management](/database-management.md).

为了表述的更清楚一下，使用`SHOW RETENTION POLICIES` 看一下RP。注意到，有2个在`food_data` \(`default` 和`two_hours`\) 中，并且`two_hours` 的第三列意味着它是`DEFAULT` RP。

```
> SHOW RETENTION POLICIES ON food_data
name              duration    replicaN    default
default        0                1               false
two_hours     2h0m0s           1                true
```

#### **Create the CQ**

现在创建CQ来自动的将10s的数据聚合到30分钟：

```
> CREATE CONTINUOUS QUERY cq_30m ON food_data BEGIN SELECT mean(website) AS mean_website,mean(phone) AS mean_phone INTO food_data."default".downsampled_orders FROM orders GROUP BY time(30m) END
```

CQ让InfluxDB 自动定期计算phone和website每30分钟的平均值。InfluxDB 同时将数据写入至 CQ的结果写入至 measurement `downsampled_orders` 和 RP `default`中。InfluxDB 会在in`downsampled_orders`中一直保存这些聚合过的数据。

> **Note:** 你必需在`INTO` clause 中指定RP，而非`DEFAULT` RP。在上面的CQ 中，我们通过`<database_name>."<retention_policy>".<measurement_name>`全名的形式将结果写入`default` RP，如果不通过这种形式，InfluxDB则会将数据写入保留2小时的 `DEFAULT`RP

更多 `CREATE CONTINUOUS QUERY` 的语法见 [Continuous Queries](/continuous-queries.md)

### **Write the data to InfluxDB and see the results**

准备好`food_data`之后， 开始将数据写入 InfluxDB 中。过了一段时间之后，我们看到database中有两个measurement： `orders` and `downsampled_orders`。

一份 `orders` 的最早的样例数据 - 这些每10s一次的数据由 `two_hours` RP控制：

```
> SELECT * FROM orders LIMIT 5
name: orders
-----------------
time                       phone  website
2015-12-04T20:00:11Z        1        6
2015-12-04T20:00:20Z        9        10
2015-12-04T20:00:30Z        2        17
2015-12-04T20:00:40Z        3        10
2015-12-04T20:00:50Z        1        15
```

我们于 12\/04\/2015 at 22:08:19 UTC 提交了这些数据 - 请注意，最早的数据保留时间不超过2小时。

> **Note:** InfluxDB默认每30分钟强制chechk RP，在check过程中可能会存在2小时之前的数据。InfluxDB的RP check的频率是可以配置的，详见[Database Configuration](/database-management.md)

`downsampled_orders`中最早的一些数据的示例 - 这些数据被聚合至 `default` RP中：

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

注意到 `downsampled_orders` 中的数据每30分钟出现一次，当中的时间戳比 `orders` measurement还小，可见 `downsampled_orders` 中的数据并没有受`two_hours` RP的影响

将RPs 和 CQs组合使用，我们让InfluxDB自动 downsample data 和 expire old data。现在你已经对这些功能有了一个简单的了解，更多详细的内容建议阅读 [CQs](https://github.com/influxdata/docs.influxdata.com/blob/master/influxdb/v0.13/query_language/continuous_queries) 和 [RPs](https://github.com/influxdata/docs.influxdata.com/blob/master/influxdb/v0.13/query_language/database_management/#retention-policy-management)

