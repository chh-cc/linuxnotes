# snmp监控

SNMP简单介绍

1. Simple Network Management Protocol。简单网络管理协议

2. 使用snmp协议可以方便我们监控服务器、交换机、路由器等



SNMP监控实战说明

1. 监控Linux、Windows服务器建议使用Agent

2. 网络设备一般使用SNMP，监控网络接口存活和网络接口流量
3. 端口使用默认161端口



使用SNMP监控说明

1. 被监控服务器需要安装snmp服务

2. 路由器、交换机可以开启SNMP服务器，需要自己设置SNMP的密码

3. Zabbix服务器通过snmp协议去监控



Snmp的版本

1. v1基于community进行控制访问

2. v2c也是基于community进行控制访问，但比v1增强了部分功能。实战中使用v2c

3. v3加强了认证



SNMP的监控基于OID

1. OID，Object Identifier对象标识符

2. OID由数字组成比较难记



使用Oid获取监控数据

1. snmpwalk -v 2c -c shijiangepwd 192.168.237.50 .1.3.6.1.4.1.2021.10.1.3 #监控cpuload

2. .1.3.6.1.2.1.2  #监控网卡信息

3. SNMP基于Oid，Oid树图的理解有助于权限的开通



MIB库

1. 由于Oid的难记，产生了MIB。类似DNS服务器，把域名和IP的关系对应上

2. MIB，Management Information Base，管理信息库。把oid跟名字对应起来

3. MIB库有多个，网络相关的mib，系统相关的mib库



使用名字获取监控信息

1. laLoad

2. ifDescr

3. ifOperStatus

4. ifHCOutOctets 网口出的总流量(byte)

5. ifHCInOctets 网口入的总流量

6. bps



Snmp密码修改（community修改）

1. 管理-> 一般->宏定义

2. {$SNMP_COMMUNITY} = shijiangepwd



监控linux：



监控windows：