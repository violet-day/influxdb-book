在深入了解InfluxDB之前，有必要熟悉一下相关的核心概念。这篇文章摘要介绍这些概念和InfluxDB的术语。下面提供了一份清单，建议从头开始阅读，这样能更好的理解InfluxDB。

### 样本数据

下面的数据虽然是虚构的，但是还是在InfluxDB中还是能够理解的。它们描述了两个`scientists`\(`langstroth`和`perpetua`\)在两个不同的`location`\(`1`和`2`\)从2015-08-18凌晨到2015-08-18上午6:12观察到的butterflies和honeybees。假设这些数据在`my_database`中，并且使用了`default`保留策略。

name: census

| time | butterfies | **honeybees** | **location** | **scientist** |
| --- | --- | --- | --- | --- |
| 2015-08-18T00:00:00Z | 12 | 23 | 1 | langstroth |
| 2015-08-18T00:00:00Z | 1 | 30 | 1 | perpetua |
| 2015-08-18T00:06:00Z | 11 | 28 | 1 | langstroth |
| 2015-08-18T00:06:00Z | 3 | 28 | 1 | perpetua |
| 2015-08-18T05:54:00Z | 2 | 11 | 2 | langstroth |
| 2015-08-18T06:00:00Z | 1 | 10 | 2 | langstroth |
| 2015-08-18T06:06:00Z | 8 | 23 | 2 | perpetua |
| 2015-08-18T06:12:00Z | 7 | 22 | 2 | perpetua |

### Discussion

现在，你在InfluxDB中看到了一些样本数据，这节会讲解它们所包含的意思。

InfluxDB是一个时间序列数据库，所以它从一切的根源`time`入手。在这些数据中，有一列`time`，InfluxDB中所有的数据都包含这一列。`time`作为时间戳存储，时间戳包含了日起和时间。

接下来的两列是`butterfiles`和`honeybees`，它们是`filed`。Filed由filed key和filed value组成。Filed keys\(`butterfiles`和`honeybees`\)类型是string，存储了元信息；field key `butterfies` 告诉我们field value`12`-`7`和相关，同理 `honeybees`说明field values `23`-`22`和｀它关联。

**Field value**是你自己的数据，可以是string，float，integer或者boolean。同时，因为InfluxDB是时间序列数据库，所以这些数据一直和时间戳相关，样本数据中的filed valus是：

```
12 23 
1 30 
11 28 
3 28 
2 11 
1 10 
8 23 
7 22 
```

上面的数据中，filed-key和field-value组成了**field set**

* `butterflies = 12 honeybees = 23` 
* `butterflies = 1 honeybees = 30` 
* `butterflies = 11 honeybees = 28` 
* `butterflies = 3 honeybees = 28` 
* `butterflies = 2 honeybees = 11` 
* `butterflies = 1 honeybees = 10` 
* `butterflies = 8 honeybees = 23` 
* `butterflies = 7 honeybees = 22`

在InfluxDB的数据结构中，fields是必须的，不能创建没有filed的数据。同时，需要注意，fileds是没有索引的。Queries如果使用fields作为过滤条件，会扫描相关的全部数据，因为查询性能不如tags。通常，fields应该包含常用查询元信息。

样本数据中的最后两列`location`和`scientist`是tags。Tags由key和value组成。key和value都以string和元数据作为存储。样本数据中，tag key为`location`和`scientist`。`locaiton`有两个值是`1`和`2`。tag key `scientist`两个值为`langstroth`和`prepetua`

上述数据中tag key-value组成了**tag set**:

* `location = 1`, `scientist = langstroth` 
* `location = 2`, `scientist = langstroth` 
* `location = 1`, `scientist = perpetua` 
* `location = 2`, `scientist = perpetua`

Tags是可选的，在你的数据结构中不一定要包含tags，但是使用tags是个不错的选择，毕竟加了索引。这意味着在tags上的查询会很快，tags用来存储常用查询元信息是个不错的方案。

measurement为tags、fields和time扮演了容器的角色，measurement的name描述了相关的数据存储。measurment的name是string类型，对于SQL用户，measurement概念和table类似。样本数据中唯一的measrement是`census`，`census`这个名称告诉了我们fields values记录了`butterfies`和`honeybees`的数量，而不是它们的大小、方向、或者排序索引。

一个measurement可以属于多个retention policies。retention policy描述了InfluxDB保留数据的时长\(`DURATION`\)，在集群\(`REPLICATION`\)中保留多少数据副本。

在样本数据中，`census`指标属于`default` retention policy。InfluxDB自动创建了该policy，保留时长没有限制，replication factor设置为1

现在，你已经很熟悉measurement、tag set和retention policy了，是时候讨论下series了。在InfluxDB中，series是数据的集合，共享同一个retention policy，measurment和tag set。下面的数据组成了4组series

| **Arbitrary series number** | **Retention policy** | **Measurement** | Tag set |
| --- | --- | --- | --- |
| series 1 | default | census | location = 1,scientist = langstroth |
| series 2 | default | census | location = 2,scientist = langstroth |
| series 3 | default | census | location = 1,scientist = perpetua |
| series 4 | default | census | location = 2,scientist = perpetua |

在设计schema和使用InfluxDB的时候，理解series的概念是非常有必要的。

最后，**point** 是series中使用了相同时间戳的field set。比如，这就是一个point

```
name: census 
----------------- 
time                   butterflies honeybees location scientist 
2015-08-18T00:00:00Z   1           30        1        perpetua
```

示例中的series定义由retention policy(default)、measuremnt(census)和tag set(location=1,scientist=perpetua)组成。对应点的时间戳为2015-08-18T00:00:00Z

以上我们提到的痘存储在database中，样本数据在database `my_database`中。InfluxDB中的database和关系型数据库累死，为users、retention policy和continuous query和time series data提供了逻辑存储。


