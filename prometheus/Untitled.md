## 数据模型

prometheus中存的是时序数据，时序数据有个特点是每条数据都有一个**时间戳**，并且时序数据都有一个**metric_name**(指标名)，和一系列的**label**，以及当前指标的值**value**。

```
<----------------------------metric-------------------------><--timestamp--><-value->
process_open_fds{instance="localhost:9090", job="prometheus"} @1434417560938 39
process_open_fds{instance="localhost:9090", job="prometheus"} @1434417561287 33
<--metric_name--><------------------label------------------->
```



**「在prometheus中，如果指标名和标签完全相同，那么将会认为他们是同一个指标，将一个指标不同时间戳的时序数据称为指标的样本。」**

## 四种指标类型

#### counter

Counter是计数器类型：

1、Counter用于累计值，可以统计Http请求数量，请求错误数，接口调用次数等单调递增的数据

2、**一直增加，不会减少**。

3、重启进程后，会被重置。



例如：

```shell
http_response_total{method="GET",endpoint="/api/tracks"}100
http_response_total{method="GET",endpoint="/api/tracks"}160
```



同时可以结合increase 和 rate 等函数统计变化速率，比如以HTTP应用请求量来进行说明：

1、通过rate()函数获取HTTP请求量的增长率

rate(http_requests_total[5m])

2、查询当前系统中，访问量前10的HTTP地址

topk(10, http_requests_total)

#### Gauge

Gauge是测量器类型：

1、Gauge是常规数值，例如温度变化、内存使用变化。

2、**可变大，可变小。**

3、重启进程后，会被重置



例如：

```shell
memory_usage_bytes{host="master-01"} 100
memory_usage_bytes{host="master-01"} 30
memory_usage_bytes{host="master-01"} 50
memory_usage_bytes{host="master-01"} 80
```

上面两种是数值指标，代表**数据的变化情况**，Histogram和Summary是统计类型的指标，表示**数据的分布情况**。

#### histogram

Histogram是一种直方图类型，可以观察到指标在各个不同的区间范围的分布情况



**有些情况下计算出来的平均值是不能反应出少部分的特殊情况**，比如用户的响应时间，这时候就可以用histogram，可以分别统计~=0.05秒的量有多少，0~0.05秒有多少，>2秒有多少，>10秒有多少



1. 对每个采样点进行统计（**并不是一段时间的统计**），打到各个桶(bucket)中（0.05秒以下多少，2秒以下多少，10秒以下多少）
2. 对每个采样点值累计和(sum) （请求总时间）
3. 对采样点的次数累计和(count) （请求的总次数）



度量指标名称: [basename]的柱状图, 上面三类的作用度量指标名称

1. [basename]_bucket{le=“上边界”}, 这个值为小于等于上边界的所有采样点数量
2. [basename]_sum
3. [basename]_count



有一点要注意的是，Histogram是累计直方图，即**每一个桶的是只有上区间**，例如表示小于0.1毫秒（le=“0.1”）的请求数量是18173个，小于 0.2毫秒（le=“0.2”)的请求是18182 个，在le=“0.2”这个桶中是包含了 le=“0.1”这个桶的数据，如果我们要拿到0.1毫秒到0.2毫秒的请求数量，可以通过两个桶想减得到。



通常会用到histogram_quantile去计算服务接口时间的耗时情况



#### summary

Summary也是用来做统计分析的，和Histogram区别在于，Summary直接存储的就是百分位数

例如：

```text
prometheus_tsdb_wal_fsync_duration_seconds{quantile="0.5"} 0.012352463
prometheus_tsdb_wal_fsync_duration_seconds{quantile="0.9"} 0.014458005
prometheus_tsdb_wal_fsync_duration_seconds{quantile="0.99"} 0.017316173
prometheus_tsdb_wal_fsync_duration_seconds_sum 2.888716127000002
prometheus_tsdb_wal_fsync_duration_seconds_count 216

#当前Prometheus Server进行wal_fsync操作的总次数为216次，耗时2.888716127000002s。其中中位数（quantile=0.5）的耗时为0.012352463，9分位数（quantile=0.9）的耗时为0.014458005s。
```

经验：

1. 如果需要聚合（aggregate），选择histograms。
2. 如果比较清楚要观测的指标的范围和分布情况，选择histograms。如果需要精确的分为数选择summar