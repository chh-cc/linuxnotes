# zabbix简介

支持实时监控数千台服务器，虚拟机和网络设备，采集百万级监控指标

使用场景：

![image-20210802202344443](https://gitee.com/c_honghui/picture/raw/master/img/20210802202351.png)

## 监控原理

Zabbix 通过C/S 模式采集数据，通过B/S模式在web 端展示和配置。

被监控端：主机通过安装agent 方式采集数据，网络设备通过SNMP 方式采集数据

## zabbix架构

![image-20210731205950021](https://gitee.com/c_honghui/picture/raw/master/img/20210731205950.png)

zabbix server：

通过收集SNMP 和agent 发送的数据，写入数据库，再通过php+apache 在web 前端展示。

agent：

Zabbix agents监控代理 部署在监控目标上，能够主动监控本地资源和应用程序，并将收集到的数据报告给Zabbix Server

数据库：

所有配置信息和zabbix收集的数据都存储在数据库

proxy：

Zabbix proxy 可以替Zabbix Server收集性能和可用性数据。

数据流：

- 监控方面，为了创建一个监控项(item)用于采集数据，必须先创建一个主机（host）。

- 告警方面，在监控项里创建触发器（trigger），通过触发器（trigger）来触发告警动作（action）。

## 监控流程

![image-20210801220038828](https://gitee.com/c_honghui/picture/raw/master/img/20210801220045.png)