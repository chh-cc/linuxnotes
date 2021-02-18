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

Prometheus通过从Jobs/exporters中拉取度量数据；而短周期的jobs在结束前可以先将度量数据推送到网关(pushgateway)，然后Prometheus再从pushgateway中获取短周期jobs的度量数据；还可以通过自动发现目标的方式来监控kubernetes集群。所有收集的数据可以存储在本地的TSDB数据库中，并在这些数据上运行规则、检索、聚合和记录新的时间序列，将产生的告警通知推送到Alertmanager组件。通过PromQL来计算指标，再结合Grafana或其他API客户端来可视化数据。

### 组件

Prometheus server：收集和存储时间序列数据

```text
prometheus仅用于以“键值”形式存储时序式的聚合数据
其中键称为指标，通常表示cpu速率、内存使用率等
同一指标可能会适配多个目标或设备，因而使用“标签”作为元数据
```

![image-20210218232158122](https://gitee.com/c_honghui/picture/raw/master/img/20210218232158.png)

```text
4种类型的指标：
counter：计数器，单调递增型的数据如站点访问次数，不能为负数也不能减少，但可以重置为0
gauge：仪表盘，有起伏特征的数据，如内存空闲大小
histogram：直方图，它会在一段时间范围内对数据进行采样，并将其计入可配置的bucket中
summary：摘要，histogram的扩展类型
```

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



## 部署