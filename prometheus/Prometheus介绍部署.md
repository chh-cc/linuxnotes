# Prometheus介绍部署

https://prometheus.io
https://github.com/prometheus  

## 简介

Prometheus主要用于大规模的云端环境和容器化微服务(k8s)的监控
Prometheus在2016年加入了云原生计算基金会(CNCF)，成为**继Kubernetes之后的第二个托管项目**，也是从CNCF第二个毕业的项目。

prometheus server端口：9090

监控什么？

| 硬件 | 温度、硬件故障等                             |
| ---- | -------------------------------------------- |
| 系统 | cpu、内存、网卡流量、tcp状态、进程数         |
| 应用 | nginx、mysql、redis                          |
| 日志 | 系统日志、服务日志、访问日志、错误日志       |
| api  | 可用性、接口请求、响应时间                   |
| 业务 | 如电商网站，每分钟产生多少订单、注册多少用户 |

## 特点

1、多维数据模型，Prometheus存储数据的格式是  **指标(metric)名称**和**键值对（称为标签）**的**时间序列**。

```text
时序数据是在一段时间通过重复测量而获得的观测数据的集合；将这些观测数据绘制于图形上，会有一个数据轴和时间轴

具有相同度量名称以及标签属于同一指标，通过标签查询指定指标
指标格式：<mertic name>{<label name>=<label value>,...}
```

<img src="https://gitee.com/c_honghui/picture/raw/master/img/20210602231330.png" alt="image-20210602231323197" style="zoom:67%;" />

2、PromQL是一种灵活的查询语言，可以利用多维度数据完成复杂的查询。

3、不依赖于分布式存储（zabbix需要外部的数据库），单个Prometheus服务节点可直接工作。

4、使用基于HTTP的拉(pull)方式采集时间序列数据

![image-20210218214947697](https://gitee.com/c_honghui/picture/raw/master/img/20210218214947.png)



## 架构

<img src="https://gitee.com/c_honghui/picture/raw/master/img/20210218215507.webp" alt="img" style="zoom: 67%;" />



Prometheus 架构由客户端在被监控系统上利用导出器采集指标数据，在服务端配置静态目标或者动态的服务发现.

- 短周期的jobs先将度量数据推送到网关(pushgateway)，然后Prometheus再从pushgateway中获取短周期jobs的度量数据

- Prometheus通过从exporters中拉取度量数据

- 通过自动发现目标的方式来监控kubernetes集群。
- 所有收集的数据可以存储在本地的TSDB数据库中，同时设定记录规则(PromQL表达式)和告警规则(频率)并发送给alertmanager进行发送报警事件到运维人员手中。再利用Grafana的仪表盘展示Prometheus服务中的数据。

### 组件

**Prometheus server**：

**收集指标和存储时间序列数据，并提供查询接口**  

主要负责数据采集和存储，提供PromQL查询语言的支持 (默认端口: 9090)

**Pushgateway**：

**短期存储指标数据。主要用于临时性的任务**  

跨网段被监控主机指标采集数据转发到网关代理等待Server的Pull。

**Exporters**：

监控指标采集器

Exporter将监控数据采集的端点**通过HTTP服务的形式暴露**给Prometheus Server

一般来说可以将Exporter分为2类：

- **直接采集**：这一类Exporter直接**内置**了对Prometheus监控的支持，比如cAdvisor，Kubernetes，Etcd，Gokit等，都直接内置了用于向Prometheus暴露监控数据的端点。
- **间接采集**：间接采集，原有监控目标并**不直接支持**Prometheus，因此我们需要通过Prometheus提供的Client Library编写该监控目标的监控采集程序。例如： Mysql Exporter，JMX Exporter，Consul Exporter等。

**Alertmanager**：

用来进行报警、prometheus_cli：命令行工具 (默认端口: 9093)

### 概念

target

```text
每个被监控的组件称为target
```

job（作业）和instance（实例）

```text
在Prometheus中，每一个暴露监控样本数据的HTTP服务称为一个实例。例如在当前主机上运行的node exporter可以被称为一个实例(Instance)，也是job中的target。
而一组用于相同采集目的的实例，或者同一个采集进程的多个副本则通过一个一个任务(Job)进行管理。
```

![image-20210218235408414](https://gitee.com/c_honghui/picture/raw/master/img/20210218235408.png)

## 基本原理

- Prometheus Server 读取配置解析静态监控端点（static_configs），以及服务发现规则(xxx_sd_configs)自动收集需要监控的端点
- Prometheus Server 周期刮取(scrape_interval)监控端点通过HTTP的Pull方式采集监控数据
- Prometheus Server HTTP 请求到达 Node Exporter，Exporter 返回一个文本响应，每个非注释行包含一条完整的时序数据：Name + Labels + Samples(一个浮点数和一个时间戳构成), 数据来源是一些官方的exporter或自定义sdk或接口
- Prometheus Server 收到响应，Relabel处理之后(relabel_configs)将其存储在TSDB中并建立倒排索引
- Prometheus Server 另一个周期计算任务(evaluation_interval)开始执行，根据配置的Rules逐个计算与设置的阈值进行匹配，若结果超过阈值并持续时长超过临界点将进行报警，此时发送Alert到AlertManager独立组件中。
- AlertManager 收到告警请求，根据配置的策略决定是否需要触发告警，如需告警则根据配置的路由链路依次发送告警，比如邮件、微信、Slack、PagerDuty、WebHook等等。
- 当通过界面或HTTP调用查询时序数据利用PromQL表达式查询，Prometheus Server 处理过滤完之后返回瞬时向量(Instant vector, N条只有一个Sample的时序数据)，区间向量(Range vector，N条包含M个Sample的时序数据)，或标量数据 (Scalar, 一个浮点数) 
- 采用Grafana开源的分析和可视化工具进行数据的图形化展示。

## 部署Prometheus

### 二进制包部署

[Download | Prometheus](https://prometheus.io/download/)

[Getting started | Prometheus](https://prometheus.io/docs/prometheus/latest/getting_started/)

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

配置systemctl管理

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

### docker部署

[Installation | Prometheus](https://prometheus.io/docs/prometheus/latest/installation/)

### 访问web

http://localhost:9090

## prometheus+grafana

grafana部署文档：https://grafana.com/grafana/download

访问地址：http://ip:3000

用户名密码：admin/admin



接下来需要配置的就是 Grafana 的采集数据源要从哪个地方采集，所以我们需要把 Grafana 的数据源配置成从 Prometheus 去采集，我们在登录到 Grafana 界面以后，在浏览器里面点击 Data Source 这个按钮：

<img src="https://gitee.com/c_honghui/picture/raw/master/img/20210331000513.png" alt="image-20210331000513090" style="zoom:67%;" />

然后点击 add data source，它里面有一个默认的配置模板，我们可以点击 Prometheus：

<img src="https://gitee.com/c_honghui/picture/raw/master/img/20210331000537.png" alt="image-20210331000537932" style="zoom:67%;" />

然后我们就可以进入 Grafana 的控制台，针对 Prometheus 这样的一个数据源，它的具体的配置界面。这里需要配置的有如下几大块配置：

<img src="https://gitee.com/c_honghui/picture/raw/master/img/20210331000621.png" alt="image-20210331000620930" style="zoom:67%;" />

一个是 HTTP，也就是 Prometheus 对外服务的接口的地址，我们填写的是 Prometheus 的服务地址和它对应的服务端口，并且设置它的权限是浏览的权限。另外Scrape interval 配置的是采集间隔，也就是每 15 秒去做一次采集。HTTP Method 代表是以 GET 的方式去请求 Prometheus 服务。这里就完成了整个对于 Prometheus 数据源的采集。

导入 Grafna 对于普罗修斯所默认携带的面板插件：

登录到我的浏览器打开grafana后台，然后点击 Dashboards 按钮，下面有一个 manager，然后点击 import，这里我搜索一个关键词 “405“，然后回车一下。这个时候我们点击完 load ，界面中有 Prometheus Metric 的插件，然后我们点击 input，这样就完成了面板插件导入。

<img src="https://gitee.com/c_honghui/picture/raw/master/img/20210331001015.png" alt="image-20210331001015461" style="zoom:80%;" />

最后我们点击主界面，这个时候我们在 Dashboard 的下面会看到一个新的 Dashboard 按钮（Node Exporter Server Metrics），我们点击进去，就会看到对应节点的 IP 地址和端口信息：

![image-20210331001206327](https://gitee.com/c_honghui/picture/raw/master/img/20210331001206.png)