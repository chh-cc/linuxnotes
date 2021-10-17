# agent配置

Zabbix的监控分为主动和被动，这2个都是针对客户端说的。

当监控的 item 比较多服务器队列比较大时可以采用主动状态,被监控客户端主动 从server 端去下载需要监控的 item 然后取数据上传到 server 端。 这种方式对服务器的负载比较小。

## 被动模式

zabbix server会根据主机关联的模板中的 监控项和数据采集间隔时间，周期性的打开随机端口并向**zabbix agent服务器的10050**发起tcp连接，然后发送获取监控项数据的指令，即zabbix server发送什么指令那么zabbix agent就收集什么数据，**zabbix server什么时候发送zabbix agent就什么时候采集**

但是被动模式的最大问题 就是会加大zabbix server的工作量，如果zabbix server负载很高还会导致不能及时获取到最新数据

工作流程：

![image-20210802223208153](https://gitee.com/c_honghui/picture/raw/master/img/20210802223208.png)

默认的监控项模板都是被动模式的，所以修改配置文件并套用模板即可

```shell
vim zabbix_agentd.conf
#指定pid文件位置
PidFile=/var/run/zabbix/zabbix_agentd.pid
#指定日志文件位置
LogFile=/var/log/zabbix/zabbix_agentd.log
#被动状态时默认启动的实例数(进程数)，为0不监听任何端口
StartAgents=3
#当前的主机名，要写正确，否则服务端会不识别
Hostname=DEV-TEST
#服务端的地址，用逗号(,)可以隔开写多个
Server=192.168.1.100
#自定义的脚本超时时间，
Timeout=8
#允许自定义脚本
UnsafeUserParameters=1
#加载其它配置文件
Include=/etc/zabbix/zabbix_agentd.d/*.conf
```

## 主动模式

由zabbix agent主动向**zabbix server的10051端口**发起tcp连接请求，因此主动模式下必须在zabbix agent配置文件中**指定zabbix server的IP**或者主机名(必须可以被解析为IP地址)，在连接到zabbix server之前 zabbix agent是不知道自己要采集那些数据以及间隔多久采集一次数据的，然后在连接到zabbix server以后获取到 自己的监控项和数据采集间隔周期时间，然后再根据监控项采集数据并返回给zabbix server，在主动模式下不再需 要zabbix serve向zabbix agent发起连接请求，因此主动模式在一定程度上可减轻zabbix server打开的本地随机端 口和进程数，在一定程度就减轻看zabbix server的压力。

工作流程：

![image-20210803001853572](https://gitee.com/c_honghui/picture/raw/master/img/20210803001853.png)

需要修改配置文件和模板，将监控项改为主动模式

修改配置文件

```shell
vim zabbix_agentd.conf
#指定pid文件位置
PidFile=/var/run/zabbix/zabbix_agentd.pid
#指定日志文件位置
LogFile=/var/log/zabbix/zabbix_agentd.log
#设置为主动模式，将关闭被动模式，不监听端口
StartAgents=0
#当前的主机名，要写正确，否则服务端会不识别
Hostname=DEV-TEST
#zabbix server的地址，用逗号(,)可以隔开写多个
ServerActive=192.168.1.100
#自定义的脚本超时时间，
Timeout=8
#允许自定义脚本
UnsafeUserParameters=1
#加载其它配置文件
Include=/etc/zabbix/zabbix_agentd.d/*.conf
```

克隆模板：

完全克隆原来被动模式的模版为主动模式

![img](https://gitee.com/c_honghui/picture/raw/master/img/20210425154545.png)        

​        <img src="https://gitee.com/c_honghui/picture/raw/master/img/20210425154559.png" alt="img" style="zoom:67%;" />        

取消链接   

​     <img src="https://gitee.com/c_honghui/picture/raw/master/img/20210425154605.png" alt="img" style="zoom:67%;" />        

​        <img src="https://gitee.com/c_honghui/picture/raw/master/img/20210425154609.png" alt="img" style="zoom:67%;" />        

把模板关联到主机，然后重启主机的agent配置文件

查看最新数据，发现获取数据的时间都是一样的：

​        ![img](https://gitee.com/c_honghui/picture/raw/master/img/20210425154621.png)

## 综合配置

这样主动和被动都开启，监控项根据情况进行设置。

```shell
#指定pid文件位置
PidFile=/var/run/zabbix/zabbix_agentd.pid
#指定日志文件位置
LogFile=/var/log/zabbix/zabbix_agentd.log
#设置为被动模式，将开启端口
StartAgents=3
#当前的主机名，要写正确，否则服务端会不识别
Hostname=web2
#服务端的地址，用逗号(,)可以隔开写多个
Server=10.10.10.137
#服务端的地址，用逗号(,)可以隔开写多个
ServerActive=10.10.10.137
#自定义的脚本超时时间，
Timeout=8
#允许自定义脚本
UnsafeUserParameters=1
#加载其它配置文件
Include=/etc/zabbix/zabbix_agentd.d/*.conf
```

