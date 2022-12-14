# linux

1、linux内核参数优化

全局用户可打开文件数：echo "fs.file-max=102400" >> /etc/sysctl.conf

用户可打开文件数和进程数：/etc/security/limits.conf；查看：ulimit -n

单个进程可操作的文件数：/etc/security/limits.d/20-nproc.conf



端口范围：net.ipv4.ip_local_port_range = 1024 65000

tcp_wmem：TCP写buffer,可参考的优化值: 8192 436600 873200

tcp_rmem ：TCP读buffer,可参考的优化值: 32768 436600 873200

netdev_max_backlog：进入包的最大设备队列.默认是300,对重负载服务器而言,该值太低,可调整到1000

tcp_max_syn_backlog：表示SYN队列的长度，默认为1024，加大队列长度为8192，可以容纳更多等待连接的网络连接数。

somaxconn：listen()的默认参数,挂起请求的最大数量.默认是128.对繁忙的服务器,增加该值有助于网络性能.可调整到256.



2、根目录满了

cd /;du -sh *

du比df容量少，因为文件删除了但资源没释放，lsof +L1



3、木马

挖矿木马会对cpu进行超频运算，占用主机大量cpu资源

1.不影响业务的情况下隔离主机

2.查看有没有异常iptables规则、计划任务、服务启动项、ssh公钥

3.删除挖矿进程、删除挖矿进程对应的文件



4、应用cpu使用率高

1.用top查找cpu高的进程

2.用top -H -p查找cpu高的线程

3.如果是java应用可以用jstack，其他可以用perf来分析进程