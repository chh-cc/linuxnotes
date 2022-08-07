# nginx

1、nginx如何处理请求

nginx结合多进程和异步机制处理请求

每当收到一个客户端请求，master进程就会生成一个worker进程来和客户端建立连接，直到连接断开，该子进程也就结束

每个worker进程使用异步非阻塞机制，可以处理多个请求，当某个worker进程接受到客户端的请求后，调用IO进行处理，如果不能立即得到回应，就去处理其他的请求（非阻塞）；而客户端此时也无需等待响应，可以去处理其他事情（异步），当 IO 返回时，就会通知此工作进程 ；该进程得到通知，暂时挂起当前处理的事务去 响应客户端请求。

2、master进程和worker进程？

master：通过管理worker进程来实现重启服务、平滑升级、更换日志文件、配置文件实时生效等功能

worker：处理请求

3、nginx常见优化

**安全优化：**

隐藏nginx版本信息优化：修改nginx配置文件实现优化。
server_tokens off;
修改nginx进程用户信息：
修改默认进程用户nginx为其他，如www.
修改nginx服务上传文件限制：
client_max_body_size 设置客户端请求报文主体最大尺寸，用户上传文件 大小。
nginx图片及目录防盗链解决方法
根据HTTP referer实现防盗链
用户从哪里跳转过来的（通过域名）referer控制
根据cookie防盗链
nginx站点目录文件及目录权限优化
只将用户上传数据目录设置为755用户和组使用nginx
其余目录和文件为755/644，用户和组使用root
使用普通用户启动nginx
利用nginx -c参数启动nginx多实例，使master进程让普通用户管理。普通用户无法使用1-1024端口。使用iptables转发。
控制nginx并发连接数
控制客户端请求nginx的速率

**性能优化：**

**1.调整worker_processes**

指nginx要生成的worker数量，一般和cpu的核心数设置一致，高并发可以和cpu核心2倍.
cat /proc/cpuinfo

**2.优化nginx服务进程均匀分配到不同cpu进行处理。**

利用worker_cpu_affinity进行优化让cpu的每颗核心平均。

**3.优化nginx事件处理模型**

利用use epoll参数修改事件模型为epoll模型。
事件模型指定配置参数放置在event区块中

**4.优化nginx单进程客户端连接数**

利用worker_connections连接参数进行调整
用户最大并发连接数=worker进程数*worker连接数

**5.优化nginx服务进程打开文件数**

利用worker_rlimit_nofile 参数进行调整

**6.优化nginx服务数据高效传输模式。**

利用sendfile on开启高速传输模式。
tcp_nopush on 表示将数据积累到一定的量再进行传输。
tcp_nopush on 表示将数据信息进行快速传输

**7.优化nginx服务超时信息。**

keepalive_timeout 优化客户端访问 nginx服务端超时时间。
http协议特点：连接断开后会给你保留一段时间

4、location匹配优先级别

精确匹配=

前缀匹配^~

按文件中的顺序正则匹配

不带任何修饰的前缀匹配

/

