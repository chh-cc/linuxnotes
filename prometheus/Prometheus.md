# Prometheus

## 简介

Prometheus是一个开源的系统监控和告警工具包，最初由SoundCloud开发。自2012年开始，许多公司和组织开始使用了Prometheus，该项目拥有非常活跃的开发者和用户社区。Prometheus现在是一个独立的开源项目，独立于任何公司进行维护。为了强调这一点，并澄清项目的治理结构，Prometheus在2016年加入了云原生计算基金会(CNCF)，成为继Kubernetes之后的第二个托管项目，也是从CNCF第二个毕业的项目。

## 特点

1、Prometheus使用的是  度量(metric)名称和键/值对标签(label)的时间序列数据，是一种多维的数据模型。

```text
时序数据是在一段时间通过重复测量而获得的观测数据的集合；将这些观测数据绘制于图形上，会有一个数据轴和时间轴
```

![image-20210218214547111](https://gitee.com/c_honghui/picture/raw/master/img/20210218214554.png)

2、PromQL是一种灵活的查询语言，可以利用度量(metric)名称和标签进行查询、聚合。

3、不依赖于分布式存储，单个Prometheus服务也是自治理的。

4、使用基于HTTP的拉(pull)从配置文件中指定的网络端点周期性获取指标数据

![image-20210218214947697](https://gitee.com/c_honghui/picture/raw/master/img/20210218214947.png)

​       prometheus支持三种类型的途径从目标抓取数据：

​       exporters（不兼容）

​       instrumentation（测量系统、兼容）

​       pushgateway

![image-20210218222629887](https://gitee.com/c_honghui/picture/raw/master/img/20210218222629.png)

5、同时也支持通过一个中间网关(pushgateway)来推送时间序列。

6、目标对象(主机)是通过静态配置或者服务发现来添加的。

7、支持多种图形模式和仪表盘。

## 架构

<img src="https://gitee.com/c_honghui/picture/raw/master/img/20210218215507.webp" alt="img" style="zoom: 67%;" />



### 组件

Prometheus server：收集和存储时间序列数据

```text
prometheus仅用于以“键值”形式存储时序式的聚合数据
其中键称为指标，通常表示cpu速率、内存使用率等
同一指标可能会适配多个目标或设备，因而使用“标签”作为元数据
```

![image-20210218232158122](https://gitee.com/c_honghui/picture/raw/master/img/20210218232158.png)

Pushgateway：接收通常由短期作业生成的指标数据的网关

Exporters：用于暴露现有应用程序或服务的指标给Prometheus server

Alertmanager：从Prometheus server接收到告警通知后，通过去重、分组、路由等预处理后向用户发送告警

Data visualization：Grafana

Service discovery：动态发现待监控的target，从而完成监控配置的重要组件，在容器化环境中很重要

### 概念

作业（job）和实例（instance）

```text
instance：能够接收Prometheus server数据scrape操作的每个端点

job：具有类似功能的instance的集合，例如一个mysql主从复制集群中的所有mysql进程
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
...
  - job_name: 'nodes'
    static_configs:
    - targets:
      - 10.1.96.3:9100
```

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

### 简介

promql是prometheus内置的数据查询语言，promql使用表达式来表述查询需求。

时间序列由指标名称和标签来唯一标识，格式为<指标>{<标签>=<标签值>,...}

- 指标：用于描述系统上要测定的某个特征

  如http_requests_total表示接收到的http请求数

- 标签：键值型数据，让指标支持多纬度特征

  如http_requests_total{method=GET}和http_requests_total{method=POST}代表两个不同的时间序列

### 时间序列选择器

- 即使向量选择器：返回相同时间戳的数据样本

  需要将返回值绘制成图形时，仅支持即使向量选择器

  即使向量选择器由两部分组成：指标名称和匹配器

  ```text
  匹配器用于定义标签过滤条件：
  =	等于
  !=	不等于
  =~	匹配
  !~	不匹配
  http_requests_total{method=GET}
  ```

- 范围向量选择器：返回相同时间范围内的数据样本 

  范围向量选择几乎总是结合速率类的函数rate一同使用

  范围向量选择器组成：表达式后紧跟一个[]，如[5m]指过去5分钟内

  ```text
  必须使用整数时间，时间单位由大到小，如1h30m，但不能用1.5h
  ```

- 偏移量修改器

  默认情况下向量选择器以当前时间为基准时间点，偏移量修改器能够修改该基准。

  在表达式后面使用offset

  http_requests_total[5m] offset 1d表示获取此刻一天前的5分钟内的所有样本

### 指标类型

4种类型的指标：
counter：计数器，单调递增型的数据，不能为负数也不能减少，但可以重置为0（重启服务器或服务）

```text
通常counter的总数并没有直接作用，要借助rate、topk、increase和irate等函数来生成样本的变化状况
rate(http_requests_total[2h]),获取2小时内，该指标下各时间序列上http总请求数的增长速率
topk(3,http_requests_total),获取该指标下http请求总数排名前3的时间序列
irate(http_requests_total[2h]),高灵敏度函数，计算指标的瞬时速率
```

gauge：仪表盘，可增可减的数据，如内存空闲大小

```text
常用于求和、平均值、最小值、最大值等聚合计算，也常结合predict_linear和delta函数使用
```

histogram：直方图，将时间范围内的数据划分为不同的时间段，并各自评估其样本个数和样本值之和
summary：类似histogram，但客户端会直接计算并上报分位数



