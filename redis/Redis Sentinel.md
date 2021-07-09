# Redis Sentinel

官方文档：<https://redis.io/topics/sentinel>

主从的问题在于,需要手动把slave提升为master,这个过程需要花时间,而哨兵具备故障自动切换的能力.



我们引入一个"观察者",让它去实时监测master的健康状态,这个观察者就是"哨兵"

- 哨兵隔一段时间,询问master是否正常
- master正常回复,表示正常;回复超时表示异常
- 哨兵发现异常,发起主从切换

![image-20210709114501879](https://gitee.com/c_honghui/picture/raw/master/img/20210709114501.png)

但这样有一个问题,如果master正常但是哨兵和master之间网络异常,那么哨兵就会误判

解决方法是,部署多个哨兵，让它们分布在不同的机器上，它们一起监测 master 的状态

- 多个哨兵每间隔一段时间，询问 master 是否正常
- master 正常回复，表示状态正常，回复超时表示异常
- 一旦有一个哨兵判定 master 异常（不管是否是网络问题），就询问其它哨兵，如果多个哨兵（设置一个阈值）都认为 master 异常了，这才判定 master 确实发生了故障
- 多个哨兵经过协商后，判定 master 故障，则发起主从切换

选举一个哨兵作为领导者,发起主从切换

![image-20210709115844670](https://gitee.com/c_honghui/picture/raw/master/img/20210709115844.png)



## 核心概念

- 哨兵至少需要 3 个实例，来保证自己的健壮性。
- 哨兵 + Redis 主从的部署架构，是**不保证数据零丢失**的，只能保证 Redis 集群的高可用性。
- 对于哨兵 + Redis 主从这种复杂的部署架构，尽量在测试环境和生产环境，都进行充足的测试和演练。

哨兵集群必须部署 2 个以上节点，如果哨兵集群仅仅部署了 2 个哨兵实例，quorum = 1。

```
+----+         +----+
| M1 |---------| R1 |
| S1 |         | S2 |
+----+         +----+
```

配置 `quorum=1` ，如果 master 宕机， s1 和 s2 中只要有 1 个哨兵认为 master 宕机了，就可以进行切换，同时 s1 和 s2 会选举出一个哨兵来执行故障转移。但是同时这个时候，需要 majority，也就是大多数哨兵都是运行的。

```
2 个哨兵，majority=2
3 个哨兵，majority=2
4 个哨兵，majority=2
5 个哨兵，majority=3
...
```

如果此时仅仅是 M1 进程宕机了，哨兵 s1 正常运行，那么故障转移是 OK 的。但是如果是整个 M1 和 S1 运行的机器宕机了，那么哨兵只有 1 个，此时就没有 majority 来允许执行故障转移，虽然另外一台机器上还有一个 R1，但是故障转移不会执行。

经典的 3 节点哨兵集群是这样的：

```
       +----+
       | M1 |
       | S1 |
       +----+
          |
+----+    |    +----+
| R2 |----+----| R3 |
| S2 |         | S3 |
+----+         +----+
```

配置 `quorum=2` ，如果 M1 所在机器宕机了，那么三个哨兵还剩下 2 个，S2 和 S3 可以一致认为 master 宕机了，然后选举出一个来执行故障转移，同时 3 个哨兵的 majority 是 2，所以还剩下的 2 个哨兵运行着，就可以允许执行故障转移。

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