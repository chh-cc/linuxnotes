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

### 简介

promql是prometheus内置的数据查询语言，promql使用表达式来表述查询需求。

时间序列由指标名称和标签来唯一标识，格式为<指标>{<标签>=<标签值>,...}

- 指标：用于描述系统上要测定的某个特征

  如http_requests_total表示接收到的http请求数

- 标签：键值型数据，让指标支持多纬度特征

  如http_requests_total{method=GET}和http_requests_total{method=POST}代表两个不同的时间序列

### 时间序列选择器

要在Prometheus中存储的大量时间序列中抽取过滤一部分，就要使用向量表达式（时间序列选择器）

- **即使向量选择器**：返回相同时间戳的数据样本

  需要将返回值绘制成图形时，仅支持即使向量选择器

  即使向量选择器由两部分组成：指标名称和匹配器

  ```text
  匹配器用于定义标签过滤条件：
  =	等于
  !=	不等于
  =~	匹配
  !~	不匹配
  node_cpu_seconds_total{nodename=~"monitor01", mode="idle"}
  ```

- **范围向量选择器**：返回相同时间范围内的数据样本 

  范围向量选择几乎总是结合速率类的函数rate一同使用

  范围向量选择器组成：表达式后紧跟一个[]，如[5m]指过去5分钟内

  ```text
  必须使用整数时间，时间单位由大到小，如1h30m，但不能用1.5h
  node_cpu_seconds_total{nodename=~"monitor01", mode="idle"}[1m]
  ```

- **偏移量修改器**

  默认情况下向量选择器以当前时间为基准时间点，偏移量修改器能够修改该基准。

  在表达式后面使用offset

  http_requests_total[5m] offset 1d表示获取此刻一天前的5分钟内的所有样本

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

同时对于Histogram的指标，我们还可以通过histogram_quantile()函数计算出其值的分位数。不同在于Histogram通过histogram_quantile函数是在服务器端计算的分位数。 而Sumamry的分位数则是直接在客户端计算完成。因此对于分位数的计算而言，Summary在通过PromQL进行查询时有更好的性能表现，而Histogram则会消耗更多的资源。反之对于客户端而言Histogram消耗的资源更少。在选择这两种方式时用户应该按照自己的实际场景进行选择。

### 常用函数

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



