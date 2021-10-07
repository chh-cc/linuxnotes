# bind

## 简介



## 部署内网dns

1.主从都安装
`yum -y install bind bind-chroot bind-utils`

安装包作用
bind : 提供了域名服务的主要程序及相关文件
bind-utils : 提供了对DNS服务器的测试工具程序（如：nslookup、dig etc）
bind-chroot : 为bind提供一个伪装的根目录以增强安全性;

2.修改主配置文件
`vim /etc/named.conf`

```shell
options {
	listen-on port 53 { any; };
	#listen-on-v6 port 53 { ::1; };  #注销或删除
	allow-query     { any; }； #允许所有ip访问
	forwarders { 219.141.136.10; }; #用于缓存转发，可不写
}
```

3.在文件行尾添加
`cd /etc`
`cp -p named.rfc1912.zones named.rfc1912.zones.bak`
`vim named.rfc1912.zones`

```shell
zone "baidu.com" IN { #设置正向DNS区域名称
	type master;
	notify yes;
	#允许哪个从DNS复制信息
	also-notify { 192.168.1.101; };
	allow-update { none; };
	file "rzsj.com.zone"; #设置对应的正向区域地址数据库文件
	allow-transfer { 192.168.1.101; }; #允许哪些从dns下载数据
};
```

4.检测文档，没有消息就是好消息
`named-checkconf`

5.建立区域文件，正向解析配置文件:
`cd /var/named`
`cp -p named.localhost baidu.com.zone`
`vim baidu.com.zone`

```shell
$TTL 1D
@       IN SOA  @ root (
                                        3       ; serial #每次同步要手动+1
                                        1D      ; refresh
                                        1H      ; retry
                                        1W      ; expire
                                        3H )    ; minimum
        NS      ns1.baidu.com. #一定带.
ns1     A       192.168.1.100
ns2     A       192.168.1.101
```

6.重新加载配置
`systemctl restart named` `systemctl enable named`

7.进行测试dns是否运行
`nslookup ns1.baidu.com 192.168.1.100`

8.修改从配置文件
`vim /etc/named.conf`

```shell
options {
    listen-on port 53 { any; };
    #listen-on-v6 port 53 { ::1; };  #注销或删除
    allow-query     { any; };
｝
```

9.行尾添加
`cd /etc`
`cp -p named.rfc1912.zones named.rfc1912.zones.bak`
`vim named.rfc1912.zones`

```shell
zone "rzsj.com" IN {
        type slave; #设置为从
        masters { 192.168.1.100; }; #主dns地址
        file "slaves/rzsj.com.zone"; #放到slaves下
        allow-update { none; };
};
```

10.检测配置，没反应就正常
`named-checkconf`

11.重新加载配制，查看是否为空
`ls /var/named/slaves/`
`systemctl restart named`
`systemctl enable named`

看是否有文件了
`ls /var/named/slaves/`

12.看是否同步了解析记录

`nslookup ns1.baidu.com 192.168.1.100`
`nslookup ns1.baidu.com 192.168.1.101`



