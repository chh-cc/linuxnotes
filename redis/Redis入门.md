# Redis入门简介

## 简介

Redis是一款开源的、高性能的**键-值存储**。它常被称作是一款数据结构服务器、**缓存服务器**。Rredis属于非关系型数据库和Memcached类似，Redis也是一种key-value型存储系统。

Redis在奇数版本为非稳定版本，例如2.7，3.1。如果为**偶数则为稳定版本**，例如3.2，3.4

- 官方网站：[https://redis.io](https://redis.io/)
- 官方各版本下载地址：<http://download.redis.io/releases/>
- Redis 中文命令参考：[http://redisdoc.com](http://redisdoc.com/)
- 中文网站1：[http://redis.cn](http://redis.cn/)
- 中文网站2：[http://www.redis.net.cn](http://www.redis.net.cn/)

缓存数据库对比：

1、Memcached

- 优点：高性能读写、单一数据类型、支持客户端式分布式集群、一致性hash多核结构、多线程读写性能高。

- 缺点：**无持久化**、节点故障可能出现缓存穿透、分布式需要客户端实现、跨房数据同步困难、架构扩容复杂度高

2、Redis

- 优点：高性能读写、多数据类型支持、数据持久化、高可用架构、支持自定义虚拟内存、支持分布式分片集群、单线程读写性能极高
- 缺点：多线程读写较Memcached慢

应用场景：

- 数据高速缓存,web会话缓存（Session Cache）
- 排行榜应用
- 消息队列,发布订阅

## Redis 基本部署

### 1、Yum 方式安装最新版本 Redis

#### 1、安装 redis-rpm源

```shell
[root@qfedu.com ~]# yum install http://rpms.famillecollet.com/enterprise/remi-release-7.rpm
```

#### 2、安装 Redis

```shell
[root@qfedu.com ~]# yum --enablerepo=remi install redis
```

#### 3、开机自启 Redis

```shell
[root@qfedu.com ~]# systemctl enable redis
```

#### 4、设置redis.conf

允许远程登录： bind 127.0.0.1 改为 bind 0.0.0.0 (可选) 

```shell
[root@qfedu.com ~]# vim /etc/redis.conf
```

### 2、编译安装最新版本redis

#### 1、编译安装 Redis

```shell
[root@qfedu.com  ~]# wget http://download.redis.io/releases/redis-5.0.5.tar.gz 
[root@qfedu.com  ~]# tar -zxvf redis-5.0.5.tar.gz -C /usr/local
[root@qfedu.com  ~]# yum install gcc -y       # gcc -v查看，如果没有需要安装
[root@qfedu.com  ~]# cd /usr/local/redis-5.0.5
[root@qfedu.com  redis-5.0.5]# make MALLOC=lib 
[root@qfedu.com  redis-5.0.5]# cd src && make all
[root@qfedu.com  src]# make install
```

#### 2、启动 Redis 实例

```shell
[root@qfedu.com  src]# ./redis-server
21522:C 17 Jun 2019 15:36:52.038 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
21522:C 17 Jun 2019 15:36:52.038 # Redis version=5.0.5, bits=64, commit=00000000, modified=0, pid=21522, just started
21522:C 17 Jun 2019 15:36:52.038 # Warning: no config file specified, using the default config. In order to specify a config file use ./redis-server /path/to/redis.conf
                _._                                                 
           _.-``__ ''-._                                            
      _.-``    `.  `_.  ''-._           Redis 5.0.5 (00000000/0) 64 bit
  .-`` .-```.  ```\/    _.,_ ''-._                                  
 (    '      ,       .-`  | `,    )     Running in standalone mode
 |`-._`-...-` __...-.``-._|'` _.-'|     Port: 6379
 |    `-._   `._    /     _.-'    |     PID: 21522
  `-._    `-._  `-./  _.-'    _.-'                                  
 |`-._`-._    `-.__.-'    _.-'_.-'|                                 
 |    `-._`-._        _.-'_.-'    |           http://redis.io       
  `-._    `-._`-.__.-'_.-'    _.-'                                  
 |`-._`-._    `-.__.-'    _.-'_.-'|                                 
 |    `-._`-._        _.-'_.-'    |                                 
  `-._    `-._`-.__.-'_.-'    _.-'                                  
      `-._    `-.__.-'    _.-'                                      
          `-._        _.-'                                          
              `-.__.-'                                              
 
出现以上界面说明安装成功
 
[root@qfedu.com  src]# ./redis-cli --version           # 查询是安装的最新版本的redis
redis-cli 5.0.5
[root@qfedu.com  src]# ./redis-server --version
Redis server v=5.0.5 sha=00000000:0 malloc=libc bits=64 build=4db47e2324dd3c5
```

### 3、配置启动数据库

#### 1、开启 Redis 服务守护进程

```shell
# 以./redis-server 启动方式，需要一直打开窗口，不能进行其他操作，不太方便，以后台进程方式启动 redis
[root@qfedu.com  src]# vim /usr/local/redis-5.0.5/redis.conf   # 默认安装好的配置文件并不在这个目录下，需要找到复制到该目录下
daemonize no 改为 daemonize yes        # 以守护进程运行           
 
[root@qfedu.com  src]# ./redis-server /usr/local/redis-5.0.5/redis.conf
21845:C 17 Jun 2019 15:44:14.129 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
21845:C 17 Jun 2019 15:44:14.129 # Redis version=5.0.5, bits=64, commit=00000000, modified=0, pid=21845, just started
21845:C 17 Jun 2019 15:44:14.129 # Configuration loaded
```

#### 2、关闭redis进程

```shell
[root@qfedu.com  src]# ps -ef|grep redis
root     21846     1  0 15:44 ?        00:00:00 ./redis-server 127.0.0.1:6379
root     22042  6950  0 15:46 pts/1    00:00:00 grep --color=auto redis
[root@qfedu.com  src]# kill -9 21846
 # 此方法启动关闭较为麻烦，且不能设置开机自启动
```

### 4、设置系统进程启动 Redis

#### 1、编辑Redis 配置文件

```shell
# 先编辑配置文件，然后在把配置文件拷贝到/etc/redis下
[root@qfedu.com  ~]# vim /usr/local/redis-5.0.5/redis.conf 
#bind 127.0.0.1           # 将bind 127.0.0.1注释掉，否则数据库只有本机能够使用 或者 修改为 0.0.0.0
daemonize yes             # 将no改为yes，使数据库能够以后台守护进程运行
protected-mode no         # 把保护模式的yes改为no,否则会阻止远程访问 
requirepass redis         # 打开注释，设置密码 
[root@qfedu.com  ~]# cp /root/redis-5.0.5/redis.conf /etc/redis/
```

#### 2、添加 Redis 系统启动

```shell
# 开机自启动，将redis的启动脚本复制一份放到/etc/init.d目录下
[root@qfedu.com ~]# cp /usr/local/redis-5.0.5/utils/redis_init_script /etc/init.d/redis 
[root@qfedu.com ~]# vim /etc/init.d/redis
CONF="/usr/local/redis-5.0.5/redis.conf"         # 将conf的变量修改下，否则读不到配置文件 
[root@qfedu.com  ~]# cd /etc/init.d
[root@qfedu.com  init.d]# chkconfig redis on
 
# 通过 systemctl 管理 redis
[root@qfedu.com ~]# systemctl start redis
[root@qfedu.com ~]# systemctl status redis
● redis.service - LSB: Redis data structure server
   Loaded: loaded (/etc/rc.d/init.d/redis; bad; vendor preset: disabled)
   Active: active (running) since Mon 2019-06-24 11:10:48 CST; 54s ago
     Docs: man:systemd-sysv-generator(8)
  Process: 3184 ExecStop=/etc/rc.d/init.d/redis stop (code=exited, status=0/SUCCESS)
  Process: 3187 ExecStart=/etc/rc.d/init.d/redis start (code=exited, status=0/SUCCESS)
   CGroup: /system.slice/redis.service
           └─3189 /usr/local/bin/redis-server *:6379
 
Jun 24 11:10:48 test systemd[1]: Starting LSB: Redis data structure server...
Jun 24 11:10:48 test redis[3187]: Starting Redis server...
Jun 24 11:10:48 test redis[3187]: 3188:C 24 Jun 2019 11:10:48.203 # oO0OoO0OoO0Oo R...0Oo
Jun 24 11:10:48 test redis[3187]: 3188:C 24 Jun 2019 11:10:48.203 # Redis version=5...ted
Jun 24 11:10:48 test redis[3187]: 3188:C 24 Jun 2019 11:10:48.203 # Configuration loaded
Jun 24 11:10:48 test systemd[1]: Started LSB: Redis data structure server.
Hint: Some lines were ellipsized, use -l to show in full.
```

### 5、Redis多实例配置

注意：本次多实例配置基于单实例配置完成后

#### 1、创建程序目录

```shell
[root@qfedu.com ~]# mkdir /application/redis  -p
[root@qfedu.com ~]# cd /application/redis/
```

####  2、修改配置文件

```shell
[root@qfedu.com redis]# vim install-redis.sh
#!/bin/bash
for i in 0 1 2 3
  do
    # 创建多实例(端口命名)目录
    mkdir -p 638$i
    # 复制启动程序到各实例
    \cp /usr/local/redis-5.0.5/src/redis-server /application/redis/638$i/
    # 复制配置文件。注意：此处基于单实例配置完成
    \cp /usr/local/redis-5.0.5/redis.conf  /application/redis/638$i/
    # 修改程序存储目录
    sed -i  "s#^dir .*#dir /application/redis/638$i/#g" /application/redis/638$i/redis.conf
    # 修改其他端口信息
    sed -i  "s#6379#638$i#g" /application/redis/638$i/redis.conf
    # 允许远程连接redis
    sed -i '/protected-mode/s#yes#no#g' /application/redis/638$i/redis.conf
done

```

#### 3、启动实例

```shell
[root@qfedu.com redis]# vim start-redis.sh
#!/bin/bash
for i in 0 1 2 3
  do
  /application/redis/638$i/redis-server /application/redis/638$i/redis.conf 
done
```

####  4、连接 redis

```shell
[root@qfedu.com redis]# redis-cli -h 192.168.152.161 -p 6379
192.168.152.161:6379>
```

### 6、Redis.conf 配置说明

1、是否后台运行

```
daemonize no/yes
```

2、默认端口

```
port 6379
```

3、AOF 日志开关是否打开

```
appendonly no/yes
```

4、日志文件位置

```
logfile /var/log/redis.log
```

5、RDB 持久化数据文件

```
dbfilename dump.rdb
```

6、指定IP进行监听

```
bind 10.0.0.51 ip2 ip3 ip4
```

7、禁止protected-mode

```
protected-mode yes/no （保护模式，是否只允许本地访问）
```

8、增加requirepass {password}

```
requirepass root
```

9、在redis-cli中使用

```
auth {password} 进行认证
```

### 7、在线变更配置

1、获取当前配置

```
CONFIG GET *
```

2、变更运行配置

```
CONFIG SET loglevel "notice"
```

3、修改密码为空

```
192.168.152.161:6379> config set requirepass ""
192.168.152.161:6379> exit
192.168.152.161:6379> config get dir
1) "dir"
2) "/usr/local/redis/data"
```



## 原理

命令结构

![file](https://gitee.com/c_honghui/picture/raw/master/img/20210217233221.png)

- 1.客户端发送命令后，Redis服务器将为这个客户端链接创造一个’输入缓存’，将命令放到里面。
- 2.再由Redis服务器进行分配挨个执行，顺序是随机的，这将不会产生并发冲突问题，也就不需要事物了。
- 3.再将结果返回到客户端的’输出缓存’中，输出缓存先存到’固定缓冲区’，如果存满了，就放入’动态缓冲区’，客户端再获得信息结果。



Redis高性能原因

1.基于内存的访问，非阻塞I/O，Redis使用事件驱动模型**epoll多路复用**实现，连接、读写、关闭都转换为事件不在网络I/O上浪费过多的时间

```text
I/O多路复用机制
打比方：小明开一家同城快送店，有一辆车和一批快递员
经营方式一：
客户每送来一个快递，就分配给一个快递员，谁拿到车就开车去送
问题：
几十个快递员基本上时间都花在了抢车上了，大部分快递员都处在闲置状态，谁抢到了车，谁就能去送快递
随着快递的增多，快递员也越来越多，小曲发现快递店里越来越挤，没办法雇佣新的快递员了
快递员之间的协调很花时间
经营方式二：
只雇一个快递员，他把客户所有的快递都标注好放在一个地方，最后依次一个一个派送

每个快递员------------------>每个线程
每个快递-------------------->每个socket(I/O流)
快递的送达地点-------------->socket的不同状态
客户送快递请求-------------->来自客户端的请求
小曲的经营方式-------------->服务端运行的代码
一辆车---------------------->CPU的核数

于是我们有如下结论
1、经营方式一就是传统的并发模型，每个I/O流(快递)都有一个新的线程(快递员)管理。
2、经营方式二就是I/O多路复用。只有单个线程(一个快递员)，通过跟踪每个I/O流的状态(每个快递的送达地点)，来管理多个I/O流。
```

下面类比到真实的redis线程模型，如图所示

![image](https://gitee.com/c_honghui/picture/raw/master/img/20210217233236.png)

2.**单线程**避免的高并发的时候，多线程有锁的问题和线程切换的CPU开销的问题。虽然是单线程的，我们还可以通过多实例来弥补。

3.使用C语言编写，更好的发挥服务器性能，并且代码简洁，性能高

## Redis数据类型

![img](https://gitee.com/c_honghui/picture/raw/master/img/20210217233241.png)

**string:**

信息缓存、计数器、分布式锁等

场景1：缓存频繁读取但不常修改，如用户信息

方案：先从redis读取，没有值就从mysql读取，并写一份到redis作缓存，要设置过期时间

键值设计：直接将用户一条mysql记录做序列化(通常序列化为json)作为值，userInfo:userid 作为key，键名如：userInfo:123，value存储对应用户信息的json串。如 key为："user:id :name:1", value为"{"name":"leijia","age":18}"。

场景2：分布式session

如果你的应用做了负载均衡，将网站的项目放在多个服务器上，当用户在服务器A上进行登陆，session文件会写在A服务器；当用户跳转页面时，请求被分配到B服务器上的时候，就找不到这个session文件，用户就要重新登陆。

可以将session存放在redis中，redis可以独立于所有负载均衡服务器，也可以放在其中一台负载均衡服务器上；但是所有应用所在的服务器连接的都是同一个redis服务器。

**hash：**

场景：以购物车为例子，用户id设置为key，那么购物车里所有的商品就是用户key对应的值了，每个商品有id和购买数量，对应hash的结构就是商品id为field，商品数量为value。

**list：**

列表本质是一个有序的，元素可重复的队列。

场景：定时排行榜，不支持实时排行榜

**set：**

集合的特点是无序性和确定性（不重复）

场景：收藏夹；每一个用户做一个收藏的集合，每个收藏的集合存放用户收藏过的歌曲id，key为用户id，value为歌曲id的集合。

**sorted set：**

有序集合的特点是有序，无重复值。与set不同的是sorted set每个元素都会关联一个score属性，redis正是通过score来为集合中的成员进行从小到大的排序。

场景：实时排行；QQ音乐中有多种实时榜单，比如飙升榜、热歌榜、新歌榜，可以用redis key存储榜单类型，score为点击量，value为歌曲id，用户每点击一首歌曲会更新redis数据，sorted set会依据score即点击量将歌曲id排序。

## 信息

- 默认端口：TCP`6379`
- 编写语言：c
- 启动redis：`redis-server`
- redis客户端：`redis-cli`
- redis基准测试工具：`redis-benchmark`
- AOF文件检测和修复工具：`redis-check-aof`
- ADB文件检测和修复工具：`redis-check-dump`
- 启动哨兵：`redis-sentinel`