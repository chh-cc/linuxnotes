# Redis Sentinel

官方文档：<https://redis.io/topics/sentinel>

Redis-Sentinel 是 Redis 官方推荐的**高可用性(HA)**解决方案，当用Redis做Master-slave的高可用方案时，假如master宕机了，Redis本身(包括它的很多客户端)都没有实现自动进行主备切换，而Redis-sentinel本身也是一个独立运行的进程，它能监控多个master-slave集群，发现master宕机后**能进行自动切换**。
![file](https://gitee.com/c_honghui/picture/raw/master/img/20210308212404.png)

Sentinel 是一个监视器，它可以根据被监视实例的身份和状态来判断应该执行何种动作。

## Redis Sentinel 功能

#### 1、监控（Monitoring）

Sentinel 会不断地检查你的主服务器和从服务器是否运作正常。

#### 2、提醒（Notification）

当被监控的某个 Redis 服务器出现问题时， Sentinel 可以通过 API 向管理员或者其他应用程序发送通知。

#### 3、自动故障迁移（Automatic failover）

当一个主服务器不能正常工作时， Sentinel 会开始一次自动故障迁移操作， 它会将失效主服务器的其中一个从服务器升级为新的主服务器， 并让失效主服务器的其他从服务器改为复制新的主服务器； 当客户端试图连接失效的主服务器时， 集群也会向客户端返回新主服务器的地址， 使得集群可以使用新主服务器代替失效服务器。

## Redis Sentinel 服务器连接

#### 1、发现并连接主服务器

Sentinel 通过用户给定的配置文件来发现主服务器。

Sentinel 会与被监视的主服务器创建两个网络连接：

- 命令连接用于向主服务器发送命令。


- 订阅连接用于订阅指定的频道，从而发现监视同一主服务器的其他 Sentinel。

![image-20210310152839284](https://gitee.com/c_honghui/picture/raw/master/img/20210310152856.png)

#### 2、发现并连接从服务器

 Sentinel 通过向主服务器发送 INFO 命令来自动获得所有从服务器的地址。

 跟主服务器一样，Sentinel 会与每个被发现的从服务器创建命令连接和订阅连接。

![image-20210310153258826](https://gitee.com/c_honghui/picture/raw/master/img/20210310153258.png)

#### 3、发现其他 Sentinel

#### 4、多个 Sentienl 之间的链接

## Redis Sentinel 检测实例的状态

Sentinel 使用 PING 命令来检测实例的状态：如果实例在指定的时间内没有返回回复，或者返回错误的回复，那么该实例会被 Sentinel 判断为下线。

## 哨兵部署

#### 1、环境准备

| 机器名称     | IP配置         | 服务角色 | 备注          |
| ------------ | -------------- | -------- | ------------- |
| redis-master | 192.168.30.107 | redis主  | 开启 sentinel |
| redis-slave1 | 192.168.30.7   | redis从  | 开启 sentinel |
| redis-slave2 | 192.168.30.2   | redis从  | 开启 sentinel |

#### 2、实现主从

（1）打开所有机器上的redis 服务

```
[root@redis-master ~]# systemctl start redis
```

（2）在主上登录查询主从关系，确实主从已经实现

```
[root@redis-master ~]# redis-cli -h 192.168.30.107 -p 6379
192.168.30.107:6379> info Replication
```

#### 3、所有节点配置 Redis Sentinel 哨兵

##### 1、Redis Sentinel 配置 

```shell
[root@redis-master ~]# vim /etc/redis-sentinel.conf
#默认监听端口26379
port 26379
#sentinel announce-ip 1.2.3.4   #监听地址，注释默认是0.0.0.0

#指定主redis和投票裁决的机器数，即至少有1个sentinel节点同时判定主节点故障时，才认为其真的故障
sentinel monitor mymaster 192.168.30.107 6379 1

#下面保存默认就行，根据自己的需求修改
sentinel down-after-milliseconds mymaster 5000   #如果联系不到节点5000毫秒，我们就认为此节点下线。
sentinel failover-timeout mymaster 60000   #设定转移主节点的目标节点的超时时长。
sentinel auth-pass <master-name> <password>   #如果redis节点启用了auth，此处也要设置password。
sentinel parallel-syncs <master-name> <numslaves>   #指在failover过程中，能够被sentinel并行配置的从节点的数量；
```

 注意：只需指定主机器的IP，等sentinel 服务开启，它能自己查询到主上的从redis；并能完成自己的操作

#####  2、指定 master 选举优先级

```shell
vim /etc/redis.conf  #根据自己的需求设置优先级
slave-priority 100   #复制集群中，主节点故障时，sentinel应用场景中的主节点选举时使用的优先级；数字越小优先级越高，但0表示不参与选举；当优先级一样时，随机选举。
```


##### 3、开启 Redis Sentienl 服务

1、开启 Redis Sentienl 服务

```shell
[root@redis-master ~]# systemctl start redis-sentinel  # 在主上开启服务，打开了26379端口
```

2、开启服务后，/etc/redis-sentinel.conf 配置文件会生成从redis 的信息