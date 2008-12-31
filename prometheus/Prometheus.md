# Prometheus

## 简介

Prometheus主要用于大规模的云端环境和容器化微服务(k8s)的监控
Prometheus在2016年加入了云原生计算基金会(CNCF)，成为**继Kubernetes之后的第二个托管项目**，也是从CNCF第二个毕业的项目。

prometheus server端口：9090

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



从上述架构图我们可以知道，Prometheus通过从Jobs/exporters中拉取度量数据；而短周期的jobs在结束前可以先将度量数据推送到网关(pushgateway)，然后Prometheus再从pushgateway中获取短周期jobs的度量数据；还可以通过自动发现目标的方式来监控kubernetes集群。所有收集的数据可以存储在本地的TSDB数据库中，并在这些数据上运行规则、检索、聚合和记录新的时间序列，将产生的告警通知推送到Alertmanager组件。通过PromQL来计算指标，再结合Grafana或其他API客户端来可视化数据。

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

- **直接采集**：这一类Exporter直接**内置**了对Prometheus监控的支持，比如cAdvisor，Kubernetes，Etcd，Gokit等，都直接内置了用于向Prometheus暴露监控数据的端点。
- **间接采集**：间接采集，原有监控目标并**不直接支持**Prometheus，因此我们需要通过Prometheus提供的Client Library编写该监控目标的监控采集程序。例如： Mysql Exporter，JMX Exporter，Consul Exporter等。

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



## 部署Prometheus

安装

```shell
wget https://github.com/prometheus/prometheus/releases/download/v2.25.0/prometheus-2.25.0.linux-amd64.tar.gz
tar xzf prometheus-2.25.0.linux-amd64.tar.gz
mv prometheus-2.25.0.linux-amd64 /usr/local/prometheus
ln -s /usr/local/prometheus/promtool /usr/bin/promtool
cd /usr/local/prometheus
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
	evaluation_interval: 15s	#记录规则和报警规则的执行间隔（频率）
	scrape_timeout: 15s	#Server 拉取数据的超时时间，该值不能大于scrape_interval的值。
...
```

```shell
# cat > /usr/lib/systemd/system/prometheus.service <<EOF

[Unit]
Description=Prometheus

[Service]
ExecStart=/usr/local/prometheus/prometheus --config.file=/usr/local/prometheus/prometheus.yml --storage.tsdb.path=/data/prometheus --web.enable-lifecycle --storage.tsdb.retention.time=180d
Restart=on-failure

[Install]
WantedBy=multi-user.target 
EOF

# promtool check config prometheus.yml   #使用promtool工具来验证配置文件 
# systemctl enable prometheus.service 
# systemctl start prometheus.service 
```



