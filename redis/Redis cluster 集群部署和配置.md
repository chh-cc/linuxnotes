# Redis cluster 集群部署和配置

## 简介

redis cluster是去中心化的，集群中的每个节点都是平等的关系，每个节点都保存各自的数据和整个集群的状态。每个节点都和其他所有节点连接，而且这些连接保持活跃。

这样就保证了我们只需要连接集群中的任意一个节点，就可以获取到其他节点的数据。

![file](https://cdn.jsdelivr.net/gh/chh-cc/linuxnotes//img/20210206213247.png)

## cluster原理

Redis集群采用[一致性哈希槽](http://www.52wiki.cn/docs/zhishi/zhishi-1alavtuhdpf7v)的方式将集群中每个主节点都分配一定的哈希槽，对写入的数据进行哈希后分配到某个主节点进行存储。

集群中每个主节点将承担一部分槽点的维护，而槽点中存储着数据，每个主节点都有至少一个从节点用于高可用。

节点通信方式：
开启一个端口 设置的端口号+10000，用于集群之间节点通信交换信息。每个节点默认每秒10次选择随机5个节点发送ping消息，将自身信息和知道的集群信息传递，收到ping消息后返回pong消息做回复。最后通过这种随机的消息交换，最终每个节点将获得所有信息。

当某个主节点挂掉，所有节点将会发现主节点挂掉了，作为主节点的从节点，就会接替主节点的工作，然后告诉所有其它节点，他成为了主。这样其它存活节点，就将它们维护的信息表更新。

这样当新的数据从任何一个节点从节点将接任做主，如果都挂掉集群将报错。当从一个节点操作，根据计算后将存储在其中一个主节点中，从节点将同步主的数据。

## 集群部署

### 环境介绍

```
[Redis-Server-1]    
主机名 = host-1    
系统 = centos-7.3    
地址 = 1.1.1.1    
软件 = redis-3.2.9 7000 7001

[Redis-Server-2]    
主机名 = host-2    
系统 = centos-7.3    
地址 = 1.1.1.2    
软件 = redis-3.2.9 7002 7003

[Redis-Server-3]    
主机名 = host-3    
系统 = centos-7.3    
地址 = 1.1.1.3    
软件 = redis-3.2.9 7004 7005
```

### 节点部署

1.参照[Centos7源码部署Redis3.2.9](http://www.linkops.cn/303.htm)文档在每个节点上部署redis。

2.每台机器上创建2个节点，以第一台为例子
`cd /usr/local/redis/`
`mkdir -p cluster/{7000,7001}`

3.创建配置文件，编辑如下内容。在7000目录创建7000.conf配置文件，其他服务器和这台一样，都更改如下项目，端口对应即可。

```shell
[root@linkops ~]# vim 7000.conf
bind 1.1.1.1 127.0.0.1   #更改为绑定地址，127一定要在后面
protected-mode yes
port 7000    #监听端口
cluster-enabled yes
cluster-config-file nodes_7000.conf  #加载配置文件
cluster-node-timeout 5000
tcp-backlog 511
timeout 0
tcp-keepalive 300
daemonize yes
supervised no
pidfile /var/log/redis/redis_7000.pid #PID文件，需要修改对应的端口
loglevel notice
logfile "/var/log/redis/redis-server.log"
databases 16
save 900 1
save 300 10
save 60 10000
stop-writes-on-bgsave-error yes
rdbcompression yes
rdbchecksum yes
dbfilename dump.rdb
dir ./
slave-serve-stale-data yes
slave-read-only yes
repl-diskless-sync no
repl-diskless-sync-delay 5
repl-disable-tcp-nodelay no
slave-priority 100
appendonly yes
appendfilename "appendonly.aof"
appendfsync everysec
no-appendfsync-on-rewrite no
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb
aof-load-truncated yes
lua-time-limit 5000
slowlog-log-slower-than 10000
slowlog-max-len 128
latency-monitor-threshold 0
notify-keyspace-events ""
hash-max-ziplist-entries 512
hash-max-ziplist-value 64
list-max-ziplist-size -2
list-compress-depth 0
set-max-intset-entries 512
zset-max-ziplist-entries 128
zset-max-ziplist-value 64
hll-sparse-max-bytes 3000
activerehashing yes
client-output-buffer-limit normal 0 0 0
client-output-buffer-limit slave 256mb 64mb 60
client-output-buffer-limit pubsub 32mb 8mb 60
hz 10
aof-rewrite-incremental-fsync yes
```

4.创建启动脚本（3台操作），启动脚本创建，3台都一样，需要更改如下内容
`vim /etc/init.d/redis7000`

```shell
CLIEXEC=/usr/local/redis/bin/redis-cli
PIDFILE=/var/log/redis/redis_${REDISPORT}.pid
CONF="/usr/local/redis/cluster/${REDISPORT}/${REDISPORT}.conf"
case "$1" in
    start)
        if [ -f $PIDFILE ]
        then
                echo "$PIDFILE exists, process is already running or crashed"
        else
                echo "Starting Redis server..."
                $EXEC $CONF
        fi
        ;;
    stop)
        if [ ! -f $PIDFILE ]
        then
                echo "$PIDFILE does not exist, process is not running"
        else
                PID=$(cat $PIDFILE)
                echo "Stopping ..."
                $CLIEXEC -p $REDISPORT shutdown
                while [ -x /proc/${PID} ]
                do
                    echo "Waiting for Redis to shutdown ..."
                    sleep 1
                done
                echo "Redis stopped"
        fi
        ;;
    *)
        echo "Please use start or stop as first argument"
        ;;
esac
```

5.启动这些服务，加入自启动项目（3台都同样操作）
`bash /etc/init.d/redis7000`

### 启动集群

1.参照[文档](http://www.linkops.cn/401.htm)安装ruby（随便找一台即可操作）

2.安装redis的gem
`wget http://shell-auto-install.oss-cn-zhangjiakou.aliyuncs.com/package/redis-4.0.1.gem`
`gem install package/redis-4.0.1.gem`

3.启动集群
这里使用create命令，ruby脚本将创建集群。 –replicas 1 表示1主1从，前3个为主节点
`/usr/local/redis/src/redis-trib.rb create  --replicas 1 1.1.1.1:7000 1.1.1.1:7001 1.1.1.2:7002 1.1.1.2:7003 1.1.1.3:7004 1.1.1.3:7005`

4.测试集群

连接集群后，查看集群信息
`/usr/local/redis/bin/redis-cli -c -h 192.168.4.212 -p 7001`
`CLUSTER INFO`