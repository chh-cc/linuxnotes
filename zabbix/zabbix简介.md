# zabbix简介

zabbix主要由2部分构成zabbix server和zabbix agent，可选组建zabbix proxy

zabbix server可以通过SNMP，zabbix agent，fping端口监视等方法对远程服务器或网络状态完成监视，数据收集等功能。同时支持Linux以及Unix平台，Windows平台只能安装客户端。

## 监控原理

Zabbix 通过C/S 模式采集数据，通过B/S模式在web 端展示和配置。

被监控端：主机通过安装agent 方式采集数据，网络设备通过SNMP 方式采集数据

Server 端：通过收集SNMP 和agent 发送的数据，写入数据库（MySQL，ORACLE 等），再通过php+apache 在web 前端展示。

## 名词解析

zabbix-server：zabbix的控制中心，收集数据，写入数据
zabbix-agent：部署在被监控端的一个程序，用于收集本机信息给服务端
host：服务器
item：监控项，例如监控cpu平均负载就是一个监控项，每隔一段时间获取一个值反馈到服务端
trigger：触发器，一些逻辑规则的组合，他有三个值，正常，异常，未知。当监控项获取的值达到设定的阈值的时候，就会触发
action：当trigger符合某个值的时候，就会触发操作，比如发送邮件

## zabbix进程

zabbix_agentd
客户端守护进程，此进程收集客户端数据，例如cpu负载、内存、硬盘使用情况等

zabbix_get
zabbix工具，单独使用的命令，通常在server或者proxy端执行获取远程客户端信息的命令。通常用户排错。例如在server端获取不到客户端的内存数据，我们可以使用zabbix_get获取客户端的内容的方式来做故障排查。

zabbix_sender
zabbix工具，用于发送数据给server或者proxy，通常用于耗时比较长的检查。很多检查非常耗时间，导致zabbix超时。于是我们在脚本执行完毕之后，使用sender主动提交数据。

zabbix_server
zabbix服务端守护进程。zabbix_agentd、zabbix_get、zabbix_sender、zabbix_proxy、zabbix_java_gateway的数据最终都是提交到server
备注：当然不是数据都是主动提交给zabbix_server,也有的是server主动去取数据。

zabbix_proxy
zabbix代理守护进程。功能类似server，唯一不同的是它只是一个中转站，它需要把收集到的数据提交/被提交到server里。为什么要用代理？代理是做什么的？卖个关子，请继续关注运维生存时间zabbix教程系列。

zabbix_java_gateway
zabbix2.0之后引入的一个功能。顾名思义：Java网关，类似agentd，但是只用于Java方面。需要特别注意的是，它只能主动去获取数据，而不能被动获取数据。它的数据最终会给到server或者proxy。