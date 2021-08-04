# 监控nginx

对nginx的活动连接和当前状态等运行状态进行监控

nginx配置示例：

```shell
location /nginx_status {
	stub_status;
	allow 172.31.0.0/16;
	allow 127.0.0.1;
	deny all;
}
```

状态页用于输出nginx的基本状态信息：

```shell
curl http://127.0.0.1/nginx_status
Active connections: 291
server accepts handled requests
	16630948 16630948 31070465
	上面三个数字分别对应accepts,handled,requests三个值
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
vim /etc/zabbix/zabbix_agentd.d/nginx_status.sh
#!/bin/bash
#Date:2016/11/11
#Author: Zhangshijie

nginx_status_fun(){
	NGINX_PORT=$1
	NGINX_COMMAND=$2
	nginx_active(){ #获取nginx_active数量
		/usr/bin/curl "http://127.0.0.1:"$NGINX_PORT"/nginx_status/" 2>/dev/null | grep 'Active' | awk '{print $NF}'
	}
	nginx_reading(){ #获取nginx_reading状态的数量
		/usr/bin/curl "http://127.0.0.1:"$NGINX_PORT"/nginx_status/" 2>/dev/null | grep 'Reading' | awk '{print $2}'
	}
	nginx_writing(){ #获取nginx_writing状态的数量
		/usr/bin/curl "http://127.0.0.1:"$NGINX_PORT"/nginx_status/" 2>/dev/null | grep
'Writing' | awk '{print $4}'
	}
	nginx_waiting(){
		/usr/bin/curl "http://127.0.0.1:"$NGINX_PORT"/nginx_status/" 2>/dev/null | grep
'Waiting' | awk '{print $6}'
	}
	nginx_accepts(){
		/usr/bin/curl "http://127.0.0.1:"$NGINX_PORT"/nginx_status/" 2>/dev/null | awk
NR==3 | awk '{print $1}'
	}
	nginx_handled(){
		/usr/bin/curl "http://127.0.0.1:"$NGINX_PORT"/nginx_status/" 2>/dev/null | awk
NR==3 | awk '{print $2}'
	}
	nginx_requests(){
		/usr/bin/curl "http://127.0.0.1:"$NGINX_PORT"/nginx_status/" 2>/dev/null | awk
NR==3 | awk '{print $3}'
	}
	
	case $NGINX_COMMAND in
		active)
			nginx_active;
			;;
		reading)
			nginx_reading;
			;;
		writing)
			nginx_writing;
			;;
		waiting)
			nginx_waiting;
			;;
		accepts)
			nginx_accepts;
			;;
		handled)
			nginx_handled;
			;;
		requests)
			nginx_requests;
	esac
}

#主函数
main(){
	case $1 in
		nginx_status) #当输入nginx_status就调用nginx_status_fun，并传递第二和第三个参数
			nginx_status_fun $2 $3;
			;;
		*)
			echo "Usage: $0 {nginx_status key}"
	esac
}

main $1 $2 $3

# chmod a+x nginx_status.sh
# bash nginx_status.sh nginx_status 80 active
```

agent添加自定义监控项

```shell
vim /etc/zabbix/zabbix_agentd.d/nginx_status.conf
UserParameter=nginx_status[*],/etc/zabbix/zabbix_agentd.d/nginx_status.sh "$1" "$2"
"$3"
```

测试监控项数据

```shell
zabbix_get -s 172.31.0.107 -p 10050 -k "nginx_status["nginx_status","80","active"]"
```

导入nginx监控模板

