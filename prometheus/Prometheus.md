# Prometheus

## 简介

Prometheus是一个开源的系统监控和告警工具包，最初由SoundCloud开发。自2012年开始，许多公司和组织开始使用了Prometheus，该项目拥有非常活跃的开发者和用户社区。Prometheus现在是一个独立的开源项目，独立于任何公司进行维护。为了强调这一点，并澄清项目的治理结构，Prometheus在2016年加入了云原生计算基金会(CNCF)，成为继Kubernetes之后的第二个托管项目，也是从CNCF第二个毕业的项目。

## 特点

1、Prometheus存储数据的格式是  **指标(metric)名称**和**键/值对标签(label)**的**时间序列**。

```text
时序数据是在一段时间通过重复测量而获得的观测数据的集合；将这些观测数据绘制于图形上，会有一个数据轴和时间轴
```

![image-20210218214547111](https://gitee.com/c_honghui/picture/raw/master/img/20210218214554.png)

2、PromQL是一种灵活的查询语言，可以利用度量(metric)名称和标签进行查询、聚合。

3、不依赖于分布式存储，单个Prometheus服务也是自治理的。

4、使用基于HTTP的拉(pull)从配置文件中指定的网络端点周期性获取指标数据

![image-20210218214947697](https://gitee.com/c_honghui/picture/raw/master/img/20210218214947.png)

​       prometheus从以下三种类型的目标抓取数据：

​       **exporters**

​       **instrumentation**

​       **pushgateway**

![image-20210218222629887](https://gitee.com/c_honghui/picture/raw/master/img/20210218222629.png)

5、目标对象(主机)是通过静态配置或者服务发现来添加的。

6、支持多种图形模式和仪表盘。

## 架构

<img src="https://gitee.com/c_honghui/picture/raw/master/img/20210218215507.webp" alt="img" style="zoom: 67%;" />



### 组件

**Prometheus server**：

Prometheus Server是Prometheus组件中的核心部分，负责实现**对监控数据的获取，存储以及查询**。 Prometheus Server可以通过静态配置管理监控目标，也可以配合使用Service Discovery的方式动态管理监控目标，并从这些监控目标中获取数据。其次Prometheus Server需要对采集到的监控数据进行存储，Prometheus Server本身就是一个时序数据库，将采集到的监控数据按照时间序列的方式存储在本地磁盘当中。最后Prometheus Server对外提供了自定义的PromQL语言，实现对数据的查询以及分析。

```text
prometheus仅用于以“键值”形式存储时序式的聚合数据
其中键称为指标，通常表示cpu速率、内存使用率等
同一指标可能会适配多个目标或设备，因而使用“标签”作为元数据
```

![image-20210218232158122](https://gitee.com/c_honghui/picture/raw/master/img/20210218232158.png)

**Pushgateway**：

由于Prometheus数据采集基于Pull模型进行设计，因此在网络环境的配置上必须要让Prometheus Server能够直接与Exporter进行通信。 当这种网络需求无法直接满足时，就可以利用PushGateway来进行中转。可以通过PushGateway将内部网络的监控数据主动Push到Gateway当中。而Prometheus Server则可以采用同样Pull的方式从PushGateway中获取到监控数据。

**Exporters**：

Exporter将监控数据采集的端点**通过HTTP服务的形式暴露**给Prometheus Server，Prometheus Server通过访问该Exporter提供的Endpoint端点，即可获取到需要采集的监控数据。

一般来说可以将Exporter分为2类：

- 直接采集：这一类Exporter直接**内置**了对Prometheus监控的支持，比如cAdvisor，Kubernetes，Etcd，Gokit等，都直接内置了用于向Prometheus暴露监控数据的端点。
- 间接采集：间接采集，原有监控目标并**不直接支持**Prometheus，因此我们需要通过Prometheus提供的Client Library编写该监控目标的监控采集程序。例如： Mysql Exporter，JMX Exporter，Consul Exporter等。

**Alertmanager**：

在Prometheus Server中支持**基于PromQL创建告警规则**，如果满足PromQL定义的规则，则会产生一条告警，而告警的后续处理流程则由AlertManager进行管理。在AlertManager中我们可以与邮件，Slack等等内置的通知方式进行集成，也可以通过Webhook自定义告警处理方式。AlertManager即Prometheus体系中的告警处理中心。

Data visualization：Grafana

Service discovery：动态发现待监控的target，从而完成监控配置的重要组件，在容器化环境中很重要

### 概念

target

```text
每个被监控的组件称为target
```

job（作业）和instance（实例）

```text
job：多个具有相同功能角色的target组合在一起就构成了一个job(作业)，例如，具有相同用途的一组主机的资源监控器(node_exporter)，又或者是MySQL数据库监控器(mysqld_exporter)

instance：job当中的target我们一般称为instance
```

![image-20210218235408414](https://gitee.com/c_honghui/picture/raw/master/img/20210218235408.png)

### 工作模式

prometheus server基于服务发现（service discovery）机制或静态配置获取要监控的目标（target），并通过每个目标上的指标exporter来采集（scrape）指标数据

而短周期的jobs在结束前可以先将度量数据推送到网关(pushgateway)，然后Prometheus再从pushgateway中获取短周期jobs的度量数据；还可以通过自动发现目标的方式来监控kubernetes集群。

所有收集的数据可以存储在本地的TSDB数据库中，并在这些数据上运行规则、检索、聚合和记录新的时间序列，将产生的告警通知推送到Alertmanager组件。通过PromQL来计算指标，再结合Grafana或其他API客户端来可视化数据。

## 部署Prometheus

安装

```shell
wget https://github.com/prometheus/prometheus/releases/download/v2.25.0/prometheus-2.25.0.linux-amd64.tar.gz
cd /usr/local/
ln -sv prometheus-2.25.0.linux-amd64/ prometheus
cd prometheus
ll
total 167992
drwxr-xr-x 2 3434 3434     4096 Feb 17 16:11 console_libraries
drwxr-xr-x 2 3434 3434     4096 Feb 17 16:11 consoles
-rw-r--r-- 1 3434 3434    11357 Feb 17 16:11 LICENSE
-rw-r--r-- 1 3434 3434     3420 Feb 17 16:11 NOTICE
-rwxr-xr-x 1 3434 3434 91044140 Feb 17 14:19 prometheus
-rw-r--r-- 1 3434 3434      926 Feb 17 16:11 prometheus.yml
-rwxr-xr-x 1 3434 3434 80948693 Feb 17 14:21 promtool
```

启动

```shell
nohup ./prometheus &
ss -tnl	#9090端口
State       Recv-Q Send-Q                                    Local Address:Port                                                   Peer Address:Port              
LISTEN      0      128                                                   *:22                                                                *:*                  
LISTEN      0      128                                                [::]:22                                                             [::]:*                  
LISTEN      0      128                                                [::]:9090  
```

配置文件

```shell
vim prometheus.yml
global:
	scrape_interval:     15s	#每隔15秒采集一次数据
	evaluation_interval: 15s
```



## 部署node export监控系统级指标

Prometheus Server并不直接服务监控特定的目标，其主要任务负责数据的收集，存储并且对外提供数据查询支持。因此为了能够能够监控到某些东西，如主机的CPU使用率，我们需要使用到Exporter。Prometheus周期性的从Exporter暴露的HTTP服务地址（通常是/metrics）拉取监控样本数据。

在被监控的服务器上下载安装node exporter

```shell
wget https://github.com/prometheus/node_exporter/releases/download/v1.1.1/node_exporter-1.1.1.linux-amd64.tar.gz
tar xzf node_exporter-1.1.1.linux-amd64.tar.gz
cd node_exporter-1.1.1.linux-amd64
cp node_exporter /usr/local/bin
nohup ./node_exporter &
ss -tnl	#9100端口
State       Recv-Q Send-Q                                    Local Address:Port                                                   Peer Address:Port              
LISTEN      0      128                                                   *:22                                                                *:*                  
LISTEN      0      128                                                [::]:10851                                                          [::]:*                  
LISTEN      0      128                                                [::]:9100                                                           [::]:*                  
LISTEN      0      128                                                [::]:22                                                             [::]:*  
```

在prometheus服务器修改配置文件并重启服务

```shell
vim /usr/local/prometheus/prometheus.yml
scrape_configs:
...
  - job_name: 'nodes'
    static_configs:
    - targets:
      - 10.1.96.3:9100
```

当我们需要采集不同的监控指标(例如：主机、MySQL、Nginx)时，我们只需要运行相应的监控采集程序，并且让Prometheus Server知道这些Exporter实例的访问地址。

在Prometheus中，每一个暴露监控样本数据的HTTP服务称为一个实例。例如在当前主机上运行的node exporter可以被称为一个**实例(Instance)**。
而一组用于相同采集目的的实例，或者同一个采集进程的多个副本则通过一个一个**任务(Job)**进行管理。

常用的node exporter指标

```text
node_cpu_seconds_total

node_memory_MemTotal_bytes

node_filesystem_size_bytes{mount_point=PATH}

node_system_unit_state{name=}

node_vmstat_pswpin：系统每秒从硬盘读到内存的字节数

node_vmstat_pswpout：系统每秒从内存写到硬盘的字节数
```

## PromQL

### 时间序列

通过Node Exporter暴露的HTTP服务，Prometheus可以采集到当前主机所有监控指标的样本数据。例如：

```shell
# HELP node_cpu Seconds the cpus spent in each mode.
# TYPE node_cpu counter
node_cpu{cpu="cpu0",mode="idle"} 362812.7890625
# HELP node_load1 1m load average.
# TYPE node_load1 gauge
node_load1 3.0703125
```

非#开头的每一行表示当前Node Exporter采集到的一个监控样本：node_cpu和node_load1表明了当前指标的名称、大括号中的标签则反映了当前样本的一些特征和维度、浮点数则是该监控样本的具体值。

#### 样本

Prometheus会将所有采集到的样本数据以时间序列（time-series）的方式保存在内存数据库中，并且定时保存到硬盘上。time-series是按照时间戳和值的序列顺序存放的，我们称之为向量(vector). 每条time-series通过指标名称(metrics name)和一组标签集(labelset)命名。如下所示，可以将time-series理解为一个以时间为Y轴的数字矩阵：

```shell
 ^
  │   . . . . . . . . . . . . . . . . .   . .   node_cpu{cpu="cpu0",mode="idle"}
  │     . . . . . . . . . . . . . . . . . . .   node_cpu{cpu="cpu0",mode="system"}
  │     . . . . . . . . . .   . . . . . . . .   node_load1{}
  │     . . . . . . . . . . . . . . . .   . .  
  v
    <------------------ 时间 ---------------->
```

在time-series中的**每一个点称为一个样本**（sample），样本由以下三部分组成：

- **指标(metric)**：metric name和描述当前样本特征的labelsets;
- **时间戳(timestamp)**：一个精确到毫秒的时间戳;
- **样本值(value)**： 一个float64的浮点型数据表示当前样本的值。

```shell
<--------------- metric ---------------------><-timestamp -><-value->
http_request_total{status="200", method="GET"}@1434417560938 => 94355
```

#### 指标

指标格式:

```shell
<metric name>{<label name>=<label value>, ...}
```

指标的名称(metric name)可以反映被监控样本的含义（比如，`http_request_total` - 表示当前系统接收到的HTTP请求总量）。
标签(label)反映了当前样本的特征维度，通过这些维度Prometheus可以对样本数据进行过滤，聚合等。

### PromQL

PromQL是Prometheus内置的数据查询语言，其提供对时间序列数据丰富的查询，聚合以及逻辑运算能力的支持。

#### 查询时间序列

当我们直接使用监控指标名称查询时，可以查询该指标下的所有时间序列。如：

```shell
http_requests_total
等同于
http_requests_total{}
```

PromQL还支持用户根据时间序列的标签匹配模式来对时间序列进行过滤，目前主要支持两种匹配模式：完全匹配和正则匹配。

```text
匹配器用于定义标签过滤条件：
=	等于
!=	不等于
=~	匹配
!~	不匹配
```

例如，如果我们只需要查询所有http_requests_total时间序列中满足标签instance为localhost:9090的时间序列:

```shell
http_requests_total{instance="localhost:9090"}
```

例如，如果想查询多个环节下的时间序列序列可以使用如下表达式：

```shell
http_requests_total{environment=~"staging|testing|development",method!="GET"}
```

#### 范围查询

直接通过类似于PromQL表达式`http_requests_total`查询时间序列时，返回值中只会包含该时间序列中的最新的一个样本值，这样的返回结果我们称之为**瞬时向量**。而相应的这样的表达式称之为**瞬时向量表达式**。

而如果我们想过去一段时间范围内的样本数据时，我们则需要使用**区间向量表达式**。时间范围通过时间范围选择器`[]`进行定义。

通过以下表达式可以选择最近5分钟内的所有样本数据：

```shell
http_requests_total{}[5m]
```

除了使用m表示分钟以外，PromQL的时间范围选择器支持其它时间单位：

- s - 秒
- m - 分钟
- h - 小时
- d - 天
- w - 周
- y - 年

#### 时间位移操作

在瞬时向量表达式或者区间向量表达式中，都是以当前时间为基准：

而如果我们想查询，5分钟前的瞬时样本数据，就可以使用位移操作，位移操作的关键字为**offset**

使用offset时间位移操作：

```shell
http_request_total{} offset 5m
http_request_total{}[1d] offset 1d
```

#### 使用聚合操作

如果描述样本特征的标签(label)在并非唯一的情况下，通过PromQL查询数据，会返回多条满足这些特征维度的时间序列。而PromQL提供的聚合操作可以用来对这些时间序列进行处理，形成一条新的时间序列：

```shell
# 查询系统所有http请求的总量
sum(http_request_total)

# 按照mode计算主机CPU的平均使用时间
avg(node_cpu) by (mode)

# 按照主机查询各个主机的CPU使用率
sum(sum(irate(node_cpu{mode!='idle'}[5m]))  / sum(irate(node_cpu[5m]))) by (instance)
```

#### 标量和字符串

除了使用瞬时向量表达式和区间向量表达式以外，PromQL还直接支持用户使用标量(Scalar)和字符串(String)。

标量（Scalar）：一个浮点型的数字值

需要注意的是，当使用表达式count(http_requests_total)，返回的数据类型，依然是瞬时向量。用户可以通过内置函数scalar()将单个瞬时向量转换为标量。

字符串（String）：一个简单的字符串值

直接使用字符串，作为PromQL表达式，则会直接返回字符串。

### 指标类型

Prometheus从exporter抓取的每一个指标均是有注释度量类型的，例如，我们来查看node_exporter的度量指标，curl http://xxx.xxx.xxx.xxx:9100/metrics
4种类型的指标：

**counter：只增不减的计数器**

只增不减（除非系统发生重置）。常见的监控指标，如http_requests_total，node_cpu都是Counter类型的监控指标。 一般在定义Counter类型指标的名称时推荐使用_total作为后缀。

PromQL内置的聚合操作和函数可以让用户对这些数据进行进一步的分析：

```shell
rate(http_requests_total[2h]) #获取2小时内，该指标下各时间序列上http总请求数的增长速率
topk(3,http_requests_total) #获取该指标下http请求总数排名前3的时间序列
irate(http_requests_total[2h]) #高灵敏度函数，计算指标的瞬时速率
```

**gauge：可增可减的仪表盘**

Gauge类型的指标侧重于反应系统的当前状态。常见指标如：node_memory_MemFree（主机当前空闲的内容大小）、node_memory_MemAvailable（可用内存大小）

```shell
常用于求和、平均值、最小值、最大值等聚合计算，也常结合predict_linear和delta函数使用
delta(cpu_temp_celsius{host="zeus"}[2h]) #计算CPU温度在两个小时内的差异
predict_linear(node_filesystem_free{job="node"}[1h], 4 * 3600) #预测系统磁盘空间在4个小时之后的剩余情况
```

**使用Histogram和Summary分析数据分布情况**

Histogram和Summary主用用于统计和分析样本的分布情况。
如果大多数API请求都维持在100ms的响应时间范围内，而个别请求的响应时间需要5s，那么就会导致某些WEB页面的响应时间落到中位数的情况，而这种现象被称为长尾问题。
为了区分是平均的慢还是长尾的慢，最简单的方式就是按照请求延迟的范围进行分组。例如，统计延迟在0~10ms之间的请求数有多少而10~20ms之间的请求数又有多少。通过这种方式可以快速分析系统慢的原因。

例如，指标prometheus_tsdb_wal_fsync_duration_seconds的指标类型为Summary。 它记录了Prometheus Server中wal_fsync处理的处理时间，通过访问Prometheus Server的/metrics地址，可以获取到以下监控样本数据：

```shell
# HELP prometheus_tsdb_wal_fsync_duration_seconds Duration of WAL fsync.
# TYPE prometheus_tsdb_wal_fsync_duration_seconds summary
prometheus_tsdb_wal_fsync_duration_seconds{quantile="0.5"} 0.012352463
prometheus_tsdb_wal_fsync_duration_seconds{quantile="0.9"} 0.014458005
prometheus_tsdb_wal_fsync_duration_seconds{quantile="0.99"} 0.017316173
prometheus_tsdb_wal_fsync_duration_seconds_sum 2.888716127000002
prometheus_tsdb_wal_fsync_duration_seconds_count 216
#当前Prometheus Server进行wal_fsync操作的总次数为216次，耗时2.888716127000002s。其中中位数（quantile=0.5）的耗时为0.012352463，9分位数（quantile=0.9）的耗时为0.014458005s。
```

我们还能找到类型为Histogram的监控指标prometheus_tsdb_compaction_chunk_range_bucket。

```shell
# HELP prometheus_tsdb_compaction_chunk_range Final time range of chunks on their first compaction
# TYPE prometheus_tsdb_compaction_chunk_range histogram
prometheus_tsdb_compaction_chunk_range_bucket{le="100"} 0
prometheus_tsdb_compaction_chunk_range_bucket{le="400"} 0
prometheus_tsdb_compaction_chunk_range_bucket{le="1600"} 0
prometheus_tsdb_compaction_chunk_range_bucket{le="6400"} 0
prometheus_tsdb_compaction_chunk_range_bucket{le="25600"} 0
prometheus_tsdb_compaction_chunk_range_bucket{le="102400"} 0
prometheus_tsdb_compaction_chunk_range_bucket{le="409600"} 0
prometheus_tsdb_compaction_chunk_range_bucket{le="1.6384e+06"} 260
prometheus_tsdb_compaction_chunk_range_bucket{le="6.5536e+06"} 780
prometheus_tsdb_compaction_chunk_range_bucket{le="2.62144e+07"} 780
prometheus_tsdb_compaction_chunk_range_bucket{le="+Inf"} 780
prometheus_tsdb_compaction_chunk_range_sum 1.1540798e+09
prometheus_tsdb_compaction_chunk_range_count 780
```

与Summary类型的指标相似之处在于Histogram类型的样本同样会反应当前指标的记录的总数(以_count作为后缀)以及其值的总量（以_sum作为后缀）。不同在于Histogram指标直接反应了在不同区间内样本的个数，区间通过标签len进行定义。

同时对于Histogram的指标，我们还可以通过histogram_quantile()函数计算出其值的分位数。不同在于Histogram通过histogram_quantile函数是**在服务器端计算的分位数**。 而Sumamry的分位数则是**直接在客户端计算完成**。因此对于分位数的计算而言，Summary在通过PromQL进行查询时有更好的性能表现，而Histogram则会消耗更多的资源。反之对于客户端而言Histogram消耗的资源更少。在选择这两种方式时用户应该按照自己的实际场景进行选择。

### PromQL操作符

#### 数学运算

例如，我们可以通过指标node_memory_free_bytes_total获取当前主机可用的内存空间大小，其样本单位为Bytes。这是如果客户端要求使用MB作为单位响应数据：

```shell
node_memory_free_bytes_total / (1024 * 1024)
```

在上一小节中我们称该表达式为瞬时向量表达式，而返回的结果成为瞬时向量。
当瞬时向量与标量之间进行数学运算时，数学运算符会依次作用于瞬时向量中的每一个样本值，从而得到一组新的时间序列。

如果是瞬时向量与瞬时向量之间进行数学运算时，过程会相对复杂一点。 例如，如果我们想根据node_disk_bytes_written和node_disk_bytes_read获取主机磁盘IO的总量：

```shell
node_disk_bytes_written + node_disk_bytes_read
```

那这个表达式是如何工作的呢？依次找到与左边向量元素匹配（标签完全一致）的右边向量元素进行运算，如果没找到匹配元素，则直接丢弃。同时新的时间序列将不会包含指标名称。 该表达式返回结果的示例如下所示：

```shell
{device="sda",instance="localhost:9100",job="node_exporter"}=>1634967552@1518146427.807 + 864551424@1518146427.807
{device="sdb",instance="localhost:9100",job="node_exporter"}=>0@1518146427.807 + 1744384@1518146427.807
```

PromQL支持的所有数学运算符如下所示：

- `+` (加法)
- `-` (减法)
- `*` (乘法)
- `/` (除法)
- `%` (求余)
- `^` (幂运算)

#### 使用布尔运算过滤时间序列

布尔运算则支持用户根据时间序列中样本的值，对时间序列进行过滤。

当前所有主机节点的内存使用率：

```shell
(node_memory_bytes_total - node_memory_free_bytes_total) / node_memory_bytes_total
```

而系统管理员在排查问题的时候可能只想知道当前内存使用率超过95%的主机呢？通过使用布尔运算符可以方便的获取到该结果：

```shell
(node_memory_bytes_total - node_memory_free_bytes_total) / node_memory_bytes_total > 0.95
```

瞬时向量与标量进行布尔运算时，PromQL依次比较向量中的所有时间序列样本的值，如果比较结果为true则保留，反之丢弃。

瞬时向量与瞬时向量直接进行布尔运算时，同样遵循默认的匹配模式：依次找到与左边向量元素匹配（标签完全一致）的右边向量元素进行相应的操作，如果没找到匹配元素，则直接丢弃。

目前，Prometheus支持以下布尔运算符如下：

- `==` (相等)
- `!=` (不相等)
- `>` (大于)
- `<` (小于)
- `>=` (大于等于)
- `<=` (小于等于)

#### 使用bool修饰符改变布尔运算符的行为

例如，只需要知道当前模块的HTTP请求量是否>=1000，如果大于等于1000则返回1（true）否则返回0（false）。这时可以使用bool修饰符改变布尔运算的默认行为。 例如：

```shell
http_requests_total > bool 1000
```

#### 使用集合运算符

通过集合运算，可以在两个瞬时向量与瞬时向量之间进行相应的集合操作。目前，Prometheus支持以下集合运算符：

- `and` (并且)
- `or` (或者)
- `unless` (排除)

***vector1 and vector2*** 会产生一个由vector1的元素组成的新的向量。该向量包含vector1中完全匹配vector2中的元素组成。

***vector1 or vector2*** 会产生一个新的向量，该向量包含vector1中所有的样本数据，以及vector2中没有与vector1匹配到的样本数据。

***vector1 unless vector2*** 会产生一个新的向量，新向量中的元素由vector1中没有与vector2匹配的元素组成。

#### 操作符优先级

查询主机的CPU使用率，可以使用表达式：

```shell
100 * (1 - avg (irate(node_cpu{mode='idle'}[5m])) by(job) )
```

在PromQL操作符中优先级由高到低依次为：

1. `^`
2. `*, /, %`
3. `+, -`
4. `==, !=, <=, <, >=, >`
5. `and, unless`
6. `or`

### PromQL聚合操作

Prometheus还提供了下列内置的聚合操作符，这些操作符作用域瞬时向量。可以将瞬时表达式返回的样本数据进行聚合，形成一个新的时间序列。

- `sum` (求和)
- `min` (最小值)
- `max` (最大值)
- `avg` (平均值)
- `stddev` (标准差)
- `stdvar` (标准方差)
- `count` (计数)
- `count_values` (对value进行计数)
- `bottomk` (后n条时序)
- `topk` (前n条时序)
- `quantile` (分位数)

使用聚合操作的语法如下：

```shell
<aggr-op>([parameter,] <vector expression>) [without|by (<label list>)]
```

其中只有`count_values`, `quantile`, `topk`, `bottomk`支持参数(parameter)。

without用于从计算结果中移除列举的标签，而保留其它标签。by则正好相反，结果向量中只保留列出的标签，其余标签则移除。通过without和by可以按照样本的问题对数据进行聚合。

例如：

```
sum(http_requests_total) without (instance)
```

等价于

```
sum(http_requests_total) by (code,handler,job,method)
```

如果只需要计算整个应用的HTTP请求总量，可以直接使用表达式：

```
sum(http_requests_total)
```

count_values用于时间序列中每一个样本值出现的次数。count_values会为每一个唯一的样本值输出一个时间序列，并且每一个时间序列包含一个额外的标签。

例如：

```
count_values("count", http_requests_total)
```

topk和bottomk则用于对样本值进行排序，返回当前样本值前n位，或者后n位的时间序列。

获取HTTP请求数前5位的时序样本数据，可以使用表达式：

```
topk(5, http_requests_total)
```

quantile用于计算当前样本数据值的分布情况quantile(φ, express)其中0 ≤ φ ≤ 1。

例如，当φ为0.5时，即表示找到当前样本数据中的中位数：

```
quantile(0.5, http_requests_total)
```

### 内置函数

https://prometheus.io/docs/prometheus/latest/querying/functions/

```shell
1、increase()函数，该函数结合counter数据类型使用，获取区间向量中的第一个和最后一个样本并返回其增长量。如果除以[区间]时间(秒)就可以获取该时间内的平均增长率。

# 获取近1分钟内网卡接收的字节数，如果不除以时间，则为区间内累计增量。
increase(node_network_receive_bytes_total{device=~"ens.*",nodename=~"monitor01"}[1m])

2-1、rate()函数，该函数配置counter数据类型使用，用于获取在这个时间段内的平均每秒增量。

# 通过rate()函数获取在1分钟时间内网卡每秒接收字节数，如果再乘以区间时间，则同increase()函数。
rate(node_network_receive_bytes_total{device=~"ens.*",nodename=~"monitor01"}[1m])

# 以下查询结果同increase()函数。
rate(node_network_receive_bytes_total{device=~"ens.*",nodename=~"monitor01"}[1m]) * 60

2-2、irate()函数，用于计算指定时间范围内每秒瞬时增长率，是基于该时间范围内最后两个数据点来计算。rate和irate函数都用于计算某个指标在一定时间间隔内的变化速率。但是它们的计算方法有所不同：irate取的是在指定时间范围内的最后两个数据点来算速率，而rate会取指定时间范围内所有数据点，算出一组速率，然后取平均值作为结果。

所以官网文档说：irate适合快速变化(fast-moving)的计数器（counter），而rate适合缓慢变化(slow-moving)的计数器（counter）。

3、sum()函数，在实际工作中CPU大多是多核心，而node_cpu_seconds_total会将每个核的数据都单独显示出来，但我们关心的是CPU总的使用情况，因此可以使用sum()函数求和后得出一条总的数据，

# 使用sum()函数获取实例在1分钟时间内，user在所有vCPU上的使用百分比之和
sum(increase(node_cpu_seconds_total{nodename=~"monitor01",mode="user"}[1m])/60)

4、count()函数，该函数用于进行统计，或用来做一些模糊判断，比如判断服务器连接数大于某个值，为真则返回1，否则返回null。

# 例如统计vCPU数
count(node_cpu_seconds_total{nodename="monitor01",mode="idle"})

# 或者统计当前TCP建立连接数是否大于200
count(node_netstat_Tcp_CurrEstab{nodename=~"monitor01"} >200 )

5、topk()函数，该函数可以从大量数据中取出排行前N的数值，N可以自定义。

# 例如从所有主机中找出近5分钟网卡流量排名前3的主机(Counter类型数据)。
topk(3,rate(node_network_receive_bytes_total{device=~'ens.*'}[5m]))

# 或者用increase
topk(3,increase(node_network_receive_bytes_total{device=~'ens.*'}[5m])/300)

# 例如从所有主机中找出连接数前3的主机(Gauge类型数据)。
topk(3,node_netstat_Tcp_CurrEstab)  

6、predict_linear()函数，根据前一个时间段的值来预测未来某个时间点数据的走势。例如，我们可以用predict_linear()函数，根据近1天的磁盘使用来预测磁盘在未来2天增长情况，大概在未来多长时间可以将磁盘占满。

predict_linear(node_filesystem_free_bytes{device="rootfs",nodename=~"monitor01",mountpoint="/"}[1d],2*86400)
```

## Prometheus告警处理

### 简介

通过在Prometheus中定义AlertRule（告警规则），Prometheus会周期性的对告警规则进行计算，如果满足告警触发条件就会向Alertmanager发送告警信息。

![image-20210301232909110](https://gitee.com/c_honghui/picture/raw/master/img/20210301232909.png)

在Prometheus中一条告警规则主要由以下几部分组成：

- 告警名称：用户需要为告警规则命名，当然对于命名而言，需要能够直接表达出该告警的主要内容
- 告警规则：告警规则实际上主要由PromQL进行定义，其实际意义是当表达式（PromQL）查询结果持续多长时间（During）后出发告警

Alertmanager作为一个独立的组件，负责接收并处理来自Prometheus Server(也可以是其它的客户端程序)的告警信息。Alertmanager可以对这些告警信息进行进一步的处理，比如当接收到大量重复告警时能够消除重复的告警信息，同时对告警信息进行分组并且路由到正确的通知方，Prometheus内置了对邮件，Slack等多种通知方式的支持，同时还支持与Webhook的集成，以支持更多定制化的场景。例如，目前Alertmanager还不支持钉钉，那用户完全可以通过Webhook与钉钉机器人进行集成，从而通过钉钉接收告警信息。同时AlertManager还提供了静默和告警抑制机制来对告警通知行为进行优化。

**Alertmanager特性**

**分组**

分组机制可以将详细的告警信息合并成一个通知。

例如，当集群中有数百个正在运行的服务实例，并且为每一个实例设置了告警规则。假如此时发生了网络故障，可能导致大量的服务实例无法连接到数据库，结果就会有数百个告警被发送到Alertmanager。

而作为用户，可能只希望能够在一个通知中中就能查看哪些服务实例收到影响。这时可以按照服务所在集群或者告警名称对告警进行分组，而将这些告警内聚在一起成为一个通知。

**抑制**

抑制是指当某一告警发出后，可以停止重复发送由此告警引发的其它告警的机制。

例如，当集群不可访问时触发了一次告警，通过配置Alertmanager可以忽略与该集群有关的其它所有告警。这样可以避免接收到大量与实际问题无关的告警通知。

**静默**

快速根据标签对告警进行静默处理。如果接收到的告警符合静默的配置，Alertmanager则不会发送告警通知。

### 自定义Prometheus告警规则

**定义告警规则**

在告警规则文件中，我们可以将一组相关的规则设置定义在一个group下。在每一个group中我们可以定义多个告警规则(rule)。

一条告警规则主要由以下几部分组成：

- alert：告警规则的名称。
- expr：基于PromQL表达式告警触发条件，用于计算是否有时间序列满足该条件。
- for：评估等待时间，可选参数。用于表示只有当触发条件持续一段时间后才发送告警。在等待期间新产生告警的状态为pending。
- labels：自定义标签，允许用户指定要附加到告警上的一组附加标签。
- annotations：用于指定一组附加信息，比如用于描述告警详细信息的文字等，annotations的内容在告警产生时会一同作为参数发送到Alertmanager。

为了能够让Prometheus能够启用定义的告警规则，我们需要在Prometheus全局配置文件中通过**rule_files**指定一组告警规则文件的访问路径，Prometheus启动后会自动扫描这些路径下规则文件中定义的内容，并且根据这些规则计算是否向外部发送通知：

```
rule_files:
  [ - <filepath_glob> ... ]
```

默认情况下Prometheus会**每分钟**对这些告警规则进行计算，如果用户想定义自己的告警计算周期，则可以通过`evaluation_interval`来覆盖默认的计算周期：

```
global:
  [ evaluation_interval: <duration> | default = 1m ]
```

**模板化**

一般来说，在告警规则文件的annotations中使用`summary`描述告警的概要信息，`description`用于描述告警的详细信息。同时Alertmanager的UI也会根据这两个标签值，显示告警信息。为了让告警信息具有更好的可读性，Prometheus支持模板化label和annotations的中标签的值。

通过`$labels.<labelname>`变量可以访问当前告警实例中指定标签的值。$value则可以获取当前PromQL表达式计算的样本值。

```
# To insert a firing element's label values:
{{ $labels.<labelname> }}
# To insert the numeric expression value of the firing element:
{{ $value }}
```

例如，可以通过模板化优化summary以及description的内容的可读性：

```yaml
groups:
- name: example
  rules:

  # Alert for any instance that is unreachable for >5 minutes.
  - alert: InstanceDown
    expr: up == 0
    for: 5m
    labels:
      severity: page
    annotations:
      summary: "Instance {{ $labels.instance }} down"
      description: "{{ $labels.instance }} of job {{ $labels.job }} has been down for more than 5 minutes."

  # Alert for any instance that has a median request latency >1s.
  - alert: APIHighRequestLatency
    expr: api_http_request_latencies_second{quantile="0.5"} > 1
    for: 10m
    annotations:
      summary: "High request latency on {{ $labels.instance }}"
      description: "{{ $labels.instance }} has a median request latency above 1s (current value: {{ $value }}s)"
```

**查看告警状态**

可以通过表达式，查询告警实例：

```
ALERTS{alertname="<alert name>", alertstate="pending|firing", <additional alert labels>}
```

样本值为1表示当前告警处于活动状态（pending或者firing），当告警从活动状态转换为非活动状态时，样本值则为0。

**实例：定义主机监控告警**

修改Prometheus配置文件prometheus.yml,添加以下配置：

```yaml
rule_files:
  - /etc/prometheus/rules/*.rules
```

在目录/etc/prometheus/rules/下创建告警文件hoststats-alert.rules内容如下：

```yaml
groups:
- name: hostStatsAlert
  rules:
  - alert: hostCpuUsageAlert
    expr: sum(avg without (cpu)(irate(node_cpu{mode!='idle'}[5m]))) by (instance) > 0.85
    for: 1m
    labels:
      severity: page
    annotations:
      summary: "Instance {{ $labels.instance }} CPU usgae high"
      description: "{{ $labels.instance }} CPU usage above 85% (current value: {{ $value }})"
  - alert: hostMemUsageAlert
    expr: (node_memory_MemTotal - node_memory_MemAvailable)/node_memory_MemTotal > 0.85
    for: 1m
    labels:
      severity: page
    annotations:
      summary: "Instance {{ $labels.instance }} MEM usgae high"
      description: "{{ $labels.instance }} MEM usage above 85% (current value: {{ $value }})"
```

重启Prometheus后访问Prometheus UIhttp://127.0.0.1:9090/rules可以查看当前以加载的规则文件。

切换到Alerts标签http://127.0.0.1:9090/alerts可以查看当前告警的活动状态。

### 部署AlertManager

获取并安装软件包

```shell
export VERSION=0.15.2
curl -LO https://github.com/prometheus/alertmanager/releases/download/v$VERSION/alertmanager-$VERSION.darwin-amd64.tar.gz
tar xvf alertmanager-$VERSION.darwin-amd64.tar.gz
```

创建alertmanager配置文件

Alertmanager解压后会包含一个默认的alertmanager.yml配置文件，内容如下所示：

```yaml
global:
  resolve_timeout: 5m

route:
  group_by: ['alertname']
  group_wait: 10s
  group_interval: 10s
  repeat_interval: 1h
  receiver: 'web.hook'
receivers:
- name: 'web.hook'
  webhook_configs:
  - url: 'http://127.0.0.1:5001/'
inhibit_rules:
  - source_match:
      severity: 'critical'
    target_match:
      severity: 'warning'
    equal: ['alertname', 'dev', 'instance']
```

启动Alertmanager

Alermanager会将数据保存到本地中，默认的存储路径为`data/`。因此，在启动Alertmanager之前需要创建相应的目录：

```shell
./alertmanager
```

查看运行状态

Alertmanager启动后可以通过9093端口访问，http://192.168.33.10:9093

**关联Prometheus与Alertmanager**

编辑Prometheus配置文件prometheus.yml,并添加以下内容

```yaml
alerting:
  alertmanagers:
    - static_configs:
        - targets: ['localhost:9093']
```

重启Prometheus服务，成功后，可以从http://192.168.33.10:9090/config查看alerting配置是否生效。