## 安装grafana

## 接入prometheus数据源

配置data sources

![image-20220731153022664](assets/image-20220731153022664.png)

<img src="assets/image-20220731153055157.png" alt="image-20220731153055157" style="zoom:67%;" />

## 界面介绍

![image-20220731160450661](assets/image-20220731160450661.png)

- 创建dashboard，一个dashboard包含多个panel

- 创建folder来分类dashboard
- import导入dashboard模板



![image-20220731153451177](assets/image-20220731153451177.png)

查询指标，输入promql

## 导入模板

导入的监控模板，可在如下链接搜索

https://grafana.com/dashboards?dataSource=prometheus&search=kubernetes



扩展：如果Grafana导入Prometheus之后，发现仪表盘没有数据，如何排查？

1、打开grafana界面，找到仪表盘对应无数据的图标

<img src="assets/image-20230228175458716.png" alt="image-20230228175458716" style="zoom:50%;" />

Edit之后出现如下：

<img src="assets/image-20230228175644837.png" alt="image-20230228175644837" style="zoom: 67%;" />

node_cpu_seconds_total就是grafana上采集的cpu的时间，需要到prometheus ui界面看看采集的指标是否是node_cpu_seconds_total

![image-20230228175830783](assets/image-20230228175830783.png)

如果在prometheus ui界面输入node_cpu_seconds_total没有数据，那就看看是不是prometheus采集的数据是node_cpu_seconds_totals，怎么看呢？

<img src="assets/image-20230228175857123.png" alt="image-20230228175857123" style="zoom:67%;" />