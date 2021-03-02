# Nginx概念

## 应用场景

作为 Web 服务器

作为负载均衡服务器

作为邮件代理服务器

## Nginx VS Apache

- 轻量级，同样起web 服务比Apache 占用更少的内存及资源

- 静态处理，Nginx 静态处理性能比 Apache 高 3倍以上

- 抗并发，Nginx 处理请求是**异步非阻塞**的，而Apache则是阻塞型的。在高并发下Nginx 能保持低资源低消耗高性能。在Apache+PHP（prefork）模式下，如果PHP处理慢或者前端压力很大的情况下，很容易出Apache进程数飙升，从而拒绝服务的现象。

- 高度模块化的设计，编写模块相对简单

- 社区活跃，各种高性能模块出品迅速

## Nginx进程模型

<img src="https://gitee.com/c_honghui/picture/raw/master/img/20210302233050.png" alt="image-20210302233043576" style="zoom: 50%;" />

- 多进程：一个 Master 进程、多个 Worker 进程。
- Master 进程：**管理 Worker 进程**。对外接口：接收外部的操作（信号）；对内转发：根据外部的操作的不同，通过信号管理 Worker；监控：监控 Worker 进程的运行状态，Worker 进程异常终止后，自动重启 Worker 进程。
- Worker 进程：所有 Worker 进程都是平等的。实际处理：**网络请求**，由 Worker 进程处理。Worker 进程数量：在 nginx.conf 中配置，一般设置为核心数，充分利用 CPU 资源，同时，避免进程数量过多，避免进程竞争 CPU 资源，增加上下文切换的损耗。

**HTTP 连接建立和请求处理过程**

- Nginx 启动时，Master 进程，加载配置文件。
- Master 进程，初始化监听的 Socket。
- Master 进程，Fork 出多个 Worker 进程。
- Worker 进程，竞争新的连接，获胜方通过三次握手，建立 Socket 连接，并处理请求。

## Nginx 高性能、高并发

- Nginx 采用多进程+异步非阻塞方式（IO 多路复用 Epoll）。
- 请求的完整过程：建立连接→读取请求→解析请求→处理请求→响应请求。
- 请求的完整过程对应到底层就是：读写 Socket 事件。

I/O多路复用

## Nginx命令

```shell
nginx -c /path/to/nginx.conf  	  # 以特定目录下的配置文件启动nginx:
nginx -s reload            	 	 	  # 修改配置后重新加载生效
nginx -s reopen   			 	 				# 重新打开日志文件
nginx -s stop  				 	 	 				# 快速停止nginx
nginx -s quit  				  	 				# 完整有序的停止nginx
nginx -t    					 		 				# 测试当前配置文件是否正确
nginx -t -c /path/to/nginx.conf   # 测试特定的nginx配置文件是否正确
```

