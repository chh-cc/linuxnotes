# Keepalived

## 一、简介

Keepalived是集群管理中**保证集群高可用**的一个服务软件，它的作用是检测web服务器的状态，如果有一台web服务器死机，或工作出现故障，Keepalived将检测到，并将有故障的web服务器从系统中**剔除**，当web服务器工作正常后，自动将web服务器**加入**到服务器集群中。解决了静态路由的单点故障问题。

## 二、Keepalived 工作原理

Keepalived是**以VRRP协议为实现基础**的，VRRP全称Virtual Router Redundancy Protocol ,即虚拟路由冗余协议。

虚拟路由冗余协议，可以认为是实现路由器高可用的协议。也就是说N台提供相同功能的路由器组成一个路由器组，这个组里面有一个master和多个backup，**master上面有一个对外提供服务的vip**，**master不断向backup发送心跳信息**，告诉backup自己还活着，当backup收不到心跳消息时就认为master已经宕机啦，这时就需要根据VRRP的优先级来选举一个backup当master。从而保证高可用。

  Keepalived主要**有三个模块**，分别是 core、check 和 vrrp。

- core 模块为 keepalived 的核心，负责主进程的启动、维护、以及全局配置文件的加载和解析。

- check 负责健康检查，包括常见的各种检查方式。

- vrrp 模块是来实现 VRRP 协议的。

## 三、Keepalived 配置文件

keepalived.conf，里面主要包括以下几个配置区域

### 1、global_defs 区域

主要是配置故障发生时的通知对象以及机器标志

```shell
global_defs {
   notification_email {
     acassen@firewall.loc
     failover@firewall.loc
     sysadmin@firewall.loc
   }
   notification_email_from Alexandre.Cassen@firewall.loc
   smtp_server 192.168.200.1
   smtp_connect_timeout 30
   router_id 192.168.224.206	#不要重复
   vrrp_skip_check_adv_addr
   vrrp_strict
   vrrp_garp_interval 0
   vrrp_gna_interval 0
}
- notification_email  故障发生时给谁发邮件通知
- notification_email_from  通知邮件从哪个地址发出
- smtp_server 通知邮件的smtp地址
- smtp_connect_timeout 连接smtp服务器的超时时间
- enable_traps开启SNMP（Simple Network Management Protocol）陷阱
- router_id 标志本节点的字符串，通常为ip地址，故障发生时邮件会通知到
```

### 2、vrrp_script 区域  

用来做健康检查的，当检查失败时会将 vrrp_instance 的 priority 减少相应的值

```shell
vrrp_script chk_nginx {
       script "/usr/local/keepalived-1.3.4/nginx_check.sh"
       interval 2 
       weight -20
}
-    script: 自己写的监测脚本。
-    interval 2: 每2s监测一次
-    weight -20：监测失败，则相应的vrrp_instance的优先级会减少20个点
```

### 3、vrrp_instance 区域

```shell
vrrp_instance VI_1 {	#定义一个实例
    state BACKUP
    interface eth0	#虚IP地址放置的网卡位置
    virtual_router_id 51	#同一个集群id一致
    mcast_src_ip 192.168.224.206
    priority 100	#优先级决定是主还是备    越大越优先
    advert_int 1	#主备通讯时间间隔
    authentication {
        auth_type PASS
        auth_pass 1111	#认证号，集群中要一致
    }
    track_script #调用定义的检查脚本
	{
   		httpd
	}
    virtual_ipaddress {
        192.168.224.208 dev eth0 label eth0:1	#使用的虚拟ip，要和网段内ip不冲突
    }
    track_script{
    	chk_nginx
    }
}
- state:只有BACKUP和MASTER。MASTER为工作状态，BACKUP是备用状态
- interface:为网卡接口：可通过ip addr查看自己的网卡接口
- virtual_router_id:虚拟路由标志。同组的virtual_router_id应该保持一致。它将决定多播的MAC地址
- priority：设置本节点的优先级，优先级高的为master
- advert_int: MASTER与BACKUP同步检查的时间间隔
- virtual_ipaddress：这就是传说中的虚拟ip
```

## 四、Keepalived实战项目

### 1、Nginx_Director + Keepalived

```shell
一、Nginx负载均衡

二、Keepalived实现调度器HA
1. 主/备调度器安装软件
[root@master ~]# yum -y install keepalived 
[root@backup ~]# yum -y install keepalived

2. Keepalived
[root@master ~]# vim /etc/keepalived/keepalived.conf
! Configuration File for keepalived

global_defs {
   router_id director1
}
vrrp_instance VI_1 {
    state MASTER
    nopreempt
    interface eth0
    virtual_router_id 90
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        10.8.16.50
    }
}
[root@slave ~]# vim /etc/keepalived/keepalived.conf
! Configuration File for keepalived

global_defs {
   router_id director2
}
vrrp_instance VI_1 {
    state BACKUP
    nopreempt
    interface eth0
    virtual_router_id 90
    priority 50
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        10.8.16.50
    }
}

3. 启动KeepAlived（主备均启动）
[root@qfedu.com ~]# chkconfig keepalived on
[root@qfedu.com ~]# service keepalived start
[root@qfedu.com ~]# ip addr
到此：
可以解决心跳故障  keepalived
不能解决Nginx服务故障

4. 对调度器Nginx健康检查
思路：
让Keepalived以一定时间间隔执行一个外部脚本，脚本的功能是当Nginx失败，则关闭本机的Keepalived
[root@master ~]# cat /etc/keepalived/check_nginx_status.sh
#!/bin/bash											        	
/usr/bin/curl -I http://localhost &>/dev/null	
if [ $? -ne 0 ];then									    	
	/etc/init.d/keepalived stop					    	
fi															        	
[root@master ~]# chmod a+x /etc/keepalived/check_nginx_status.sh

[root@master ~]# vim /etc/keepalived/keepalived.conf
! Configuration File for keepalived

global_defs {
   router_id director1
}

vrrp_script check_nginx {
   script "/etc/keepalived/check_nginx_status.sh"
   interval 5
}

vrrp_instance VI_1 {
    state MASTER
    interface eth0
    nopreempt
    virtual_router_id 90
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass qfedu
    }
    
    virtual_ipaddress {
        10.8.16.50
    }

    track_script {
        check_nginx
    }
}

注：必须先启动nginx，再启动keepalived
```

### 2、MySQL + Keepalived

```shell
项目环境：
VIP 192.168.122.100
mysql1 192.168.122.10
mysql2 192.168.122.20

实施步骤：
一、keepalived 主备配置文件
192.168.122.10 Master配置
[root@qfedu.com ~]# vim /etc/keepalived/keepalived.conf
! Configuration File for keepalived

global_defs {
   router_id mysql1
}

vrrp_script check_run {
   script "/root/keepalived_check_mysql.sh"
   interval 5
}

vrrp_instance VI_1 {
    state MASTER
    interface eth0
    virtual_router_id 88
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass qfedu
    }

    track_script {
        check_run
    }

    virtual_ipaddress {
        192.168.122.100
    }
}
=============================================================
192.168.122.20 Slave配置
[root@qfedu.com ~]# vim /etc/keepalived/keepalived.conf
! Configuration File for keepalived

global_defs {
   router_id mysql2
}

vrrp_script check_run {
   script "/root/keepalived_check_mysql.sh"
   interval 5
}

vrrp_instance VI_1 {
    state BACKUP
    interface eth0
    virtual_router_id 88
    priority 50
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass qfedu
    }

    track_script {
        check_run
    }

    virtual_ipaddress {
        192.168.122.100
    }
}

1. 注意空格
2. 日志查看脚本是否被执行
[root@xen2 ~]# tail -f /var/log/messages 
Jun 19 15:20:19 xen1 Keepalived_vrrp[6341]: Using LinkWatch kernel netlink reflector...
Jun 19 15:20:19 xen1 Keepalived_vrrp[6341]: VRRP sockpool: [ifindex(2), proto(112), fd(11,12)]
Jun 19 15:20:19 xen1 Keepalived_vrrp[6341]: VRRP_Script(check_run) succeeded

二、mysql状态检测脚本/root/keepalived_check_mysql.sh（两台MySQL同样的脚本）
版本一：
[root@qfedu.com ~]#  vim  /root/keepalived_check_mysql.sh
#!/bin/bash 
MYSQL=/usr/local/mysql/bin/mysql
MYSQL_HOST=localhost 
MYSQL_USER=root 
MYSQL_PASSWORD=qfedu
CHECK_TIME=3

#mysql  is working MYSQL_OK is 1 , mysql down MYSQL_OK is 0 
MYSQL_OK=1
 
check_mysql_helth (){ 
    $MYSQL -h $MYSQL_HOST -u $MYSQL_USER -p${MYSQL_PASSWORD} -e "show status" &>/dev/null 
    if [ $? -eq 0 ] ;then 
    	MYSQL_OK=1
    else 
    	MYSQL_OK=0 
    fi 
    return $MYSQL_OK 
}
 
while [ $CHECK_TIME -ne 0 ]
do 
    check_mysql_helth 
	if [ $MYSQL_OK -eq 1 ] ; then 
    		exit 0 
	fi
 
	if [ $MYSQL_OK -eq 0 ] &&  [ $CHECK_TIME -eq 1 ];then
     		/etc/init.d/keepalived stop					
     		exit 1								
 	fi										
	let CHECK_TIME--
 	sleep 1 
done
==================================================================
版本二：
[root@qfedu.com ~]#  vim  /root/keepalived_check_mysql.sh
#!/bin/bash 
MYSQL=/usr/local/mysql/bin/mysql
MYSQL_HOST=localhost 
MYSQL_USER=root 
MYSQL_PASSWORD=qfedu
CHECK_TIME=3

#mysql  is working MYSQL_OK is 1 , mysql down MYSQL_OK is 0 
MYSQL_OK=1
 
check_mysql_helth (){ 
    $MYSQL -h $MYSQL_HOST -u $MYSQL_USER -p${MYSQL_PASSWORD} -e "show status" &>/dev/null 
    if [ $? -eq 0 ] ;then 
    	MYSQL_OK=1
    else 
    	MYSQL_OK=0 
    fi 
    return $MYSQL_OK 
}
 
while [ $CHECK_TIME -ne 0 ]
do 
    check_mysql_helth 
	if [ $MYSQL_OK -eq 1 ] ; then 
    		exit 0 
	fi
 
	let CHECK_TIME--
 	sleep 1 
done

/etc/init.d/keepalived stop					
exit 1

[root@qfedu.com ~]# chmod 755 /root/keepalived_check_mysql.sh

两边均启动keepalived
[root@qfedu.com ~]# /etc/init.d/keepalived start
[root@qfedu.com ~]# chkconfig --add keepalived
[root@qfedu.com ~]# chkconfig keepalived on
```

### 3、Lvs_Director + Keepalived

## 五、keepalived脑裂

Keepalived的BACKUP主机在收到不MASTER主机报文后就会切换成为master，如果是它们之间的**通信线路出现问题**，无法接收到彼此的组播通知，但是两个节点**实际都处于正常工作状态**，**这时两个节点均为master**强行绑定虚拟IP，导致不可预料的后果，这就是脑裂。

解决方式:

检测网关
由于keepalived体系中主备两台机器所处的状态与对方有关。如果主备机器之间的通信出了网题，那就ping网关，如果失败则证明网络有问题，将当前节点关闭，如果成功再开启。

问题是，当内部mysql所在机器出现网络问题，但是他是给内网提供服务的，这会导致2台mysql都关闭虚拟ip。

所以可以改改，将两台机器互相ping，防止网络问题。

```shell
vim check_keepalived.sh
#!/bin/bash
#检测keepalived脑裂脚本
#ping网关失败2次则关闭keepalived服务，成功2次则启动
#[使用设置]
#网关地址或者对方keepalived节点地址，互ping
getway_ip=192.168.1.1
#[自带变量]
check_ok=0
check_no=0
while [ 1 ]
do
    ping -c 1 $getway_ip
    if [[ $? -eq 0 ]];then
        let check_ok++
    else
        let check_no++
    fi
    if [[ $check_ok -eq 2 ]];then
        systemctl start keepalived
        check_ok=0
    elif [[ $check_no -eq 2 ]];then
        systemctl stop keepalived
        check_no=0
    fi
    sleep 1
done
```

