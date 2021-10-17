# 监控nginx

对nginx的活动连接和当前状态等运行状态进行监控

nginx配置示例：

```shell
vim /etc/nginx/conf.d/nginx_status.conf
server {
    listen 80;
    server_name 10.80.244.66;
    location /status {
        stub_status on;
        access_log off;
        allow 127.0.0.1;
    }
}
```

状态页用于输出nginx的基本状态信息：

```shell
curl http://10.80.244.66/nginx_status
Active connections: 291
server accepts handled requests
	16630948 16630948 31070465   #这三个数字分别对应accepts,handled,requests三个值
Reading: 6 Writing: 179 Waiting: 106

#Active connections： 当前处于活动状态的客户端连接数，包括连接等待空闲连接数。
#accepts：统计总值，Nginx自启动后已经接受的客户端请求的总数。
#handled：统计总值，Nginx自启动后已经处理完成的客户端请求的总数，通常等于accepts，除非有因worker_connections限制等被拒绝的连接。
#requests：统计总值，Nginx自启动后客户端发来的总的请求数

#Reading：当前状态，正在读取客户端请求报文首部的连接的连接数。
#Writing：当前状态，正在向客户端发送响应报文过程中的连接数。
#Waiting：当前状态，正在等待客户端发出请求的空闲连接数，开启 keep-alive的情况下,这个值等于 active –(reading+writing)
```

监控项脚本：

```shell
vim /opt/scripts/nginx_status.sh
#!/bin/bash
#Date: 2021/10/15
#Author: ChenHonghui

HOST=10.80.244.66

#nginx进程是否运行
exist(){
    /sbin/pidof nginx | wc -l
}
#当前处于活动状态的客户端连接数
active(){
    /usr/bin/curl "http://$HOST/status" 2>/dev/null |grep "Active" | awk '{print $NF}'
}
#正在读取客户端请求报文首部的连接的连接数
reading(){
    /usr/bin/curl "http://$HOST/status" 2>/dev/null |grep "Reading" | awk '{print $2}'
}
#正在向客户端发送响应报文过程中的连接数
writing(){
    /usr/bin/curl "http://$HOST/status" 2>/dev/null |grep "Writing" | awk '{print $4}'
}
#正在等待客户端发出请求的空闲连接数
waiting(){
    /usr/bin/curl "http://$HOST/status" 2>/dev/null |grep "Waiting" | awk '{print $6}'
}

$1

# chmod o+x nginx_status.sh
# sh nginx_status.sh active
```

agent添加自定义监控项

```shell
vim /etc/zabbix/zabbix_agentd.d/nginx_status.conf
UserParameter=nginx_status[*],/opt/scripts/nginx_status.sh $1
```

测试监控项数据

```shell
zabbix_get -s 10.80.244.66 -p 10050 -k "nginx_status[active]"
```

配置nginx监控模板



