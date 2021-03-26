# Redis入门简介

## 简介

Redis是一款开源的、高性能的**键-值存储**。它常被称作是一款数据结构服务器、**缓存服务器**。Rredis属于非关系型数据库和Memcached类似，Redis也是一种key-value型存储系统。

Redis在奇数版本为非稳定版本，例如2.7，3.1。如果为**偶数则为稳定版本**，例如3.2，3.4

- 官方网站：[https://redis.io](https://redis.io/)
- 官方各版本下载地址：<http://download.redis.io/releases/>
- Redis 中文命令参考：[http://redisdoc.com](http://redisdoc.com/)
- 中文网站1：[http://redis.cn](http://redis.cn/)
- 中文网站2：[http://www.redis.net.cn](http://www.redis.net.cn/)

### 应用场景

缓存

排行榜系统

计数器应用

社交网络

消息队列

### 项目中缓存如何使用

### 为什么要用缓存

高性能和高并发

高性能：

```text
假设一个场景，一个请求过来操作mysql，耗时600ms，可以把它扔缓存里，一个key对应一个value，下次再有人查直接走缓存不走MySQL，2ms搞定，性能提升300倍
```

高并发：

```text
mysql这么重要的数据库，不是让你玩高并发的，单机支撑到2000QPS开始容易报警。要是高峰期1秒有1万请求过来，mysql单机绝对死掉，这时可以把很多数据放缓存里，单机支撑的并发量轻松一秒几万十几万。
缓存是走内存的，天然支持高并发。
```

### 用缓存会有什么不良后果

缓存与数据库双写不一致

缓存雪崩、缓存穿透、缓存击穿

缓存并发竞争

### redis和memcached对比：

- redis支持复杂的数据结构

- redis原生支持集群

  redis3.x中便支持cluster模式

- 性能对比

  存储100k以上的数据，memcached性能要高于redis

### redis线程模型

单线程和I/O多路复用模型来实现高性能

## 信息

- **默认端口**：TCP`6379`
- 编写语言：c
- 启动redis：`redis-server`
- redis客户端：`redis-cli`
- redis基准测试工具：`redis-benchmark`
- AOF文件检测和修复工具：`redis-check-aof`
- ADB文件检测和修复工具：`redis-check-dump`
- 启动哨兵：`redis-sentinel`

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

如果用6380作为端口启动:

```shell
redis-server --port 6380
```

指定配置文件启动:

```shell
redis-server /opt/redis/redis.conf
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

交互方式：

```shell
[root@qfedu.com redis]# redis-cli -h 192.168.152.161 -p 6379
192.168.152.161:6379> set hello world
OK
192.168.152.161:6379> get hello
"world"
```

非交互方式：

```shell
redis-cli -h 127.0.0.1 -p 6379 get hello
"world"
```

### 6、在线变更配置

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

## Redis数据类型

<img src="https://gitee.com/c_honghui/picture/raw/master/img/20210326143138.png" alt="image-20210326143019276" style="zoom:67%;" />

**string:**

信息缓存、计数器、分布式锁等

```shell
set key value [ex seconds] [px milliseconds] [nx|xx]
ex seconds:为键设置秒级过期时间
px milliseconds：为键设置毫秒级过期时间
nx：键必须不存在才能设置
xx：键必须存在才能设置

mset key value [key value ...]

mget key [key ...]
#返回nil为空

#计数
incr key
值不是整数，返回错误
值是整数返回自增后结果
键不存在按照0自增，返回1
```

场景1：缓存功能，如用户信息

方案：先从redis读取，没有值就从mysql读取，并写一份到redis作缓存，要设置过期时间

键值设计：直接将用户一条mysql记录做序列化(通常序列化为json)作为值，userInfo:userid 作为key，键名如：userInfo:123，value存储对应用户信息的json串。如 key为："user:id :name:1", value为"{"name":"leijia","age":18}"。

场景2：计数

场景3：分布式session

如果你的应用做了负载均衡，将网站的项目放在多个服务器上，当用户在服务器A上进行登陆，session文件会写在A服务器；当用户跳转页面时，请求被分配到B服务器上的时候，就找不到这个session文件，用户就要重新登陆。

可以将session存放在redis中，redis可以独立于所有负载均衡服务器，也可以放在其中一台负载均衡服务器上；但是所有应用所在的服务器连接的都是同一个redis服务器。

场景4：短信验证码限速

**hash：**

键值本身就是一个键值对结构

场景：以购物车为例子，用户id设置为key，那么购物车里所有的商品就是用户key对应的值了，每个商品有id和购买数量，对应hash的结构就是商品id为field，商品数量为value。

```shell
hset person name bingo
hset person age 20
hset person id 1
hget person name

(person = {
  "name": "bingo",
  "age": 20,
  "id": 1
})
```

**list：**

列表用来存储多个有序的可重复的字符串。

```shell
从左到右获取列表所有元素
lrange listkey 0 -1
获取第2到第4个元素
lrange listkey 1 3
获取当前列表最后一个元素
lindex listkey -1
获取列表长度
llen key

从右边插入元素
rpush key value [value ...]
从左边插入元素
lpush key value [value ...]
在元素b前插入java
linsert listkey before b java
```

场景：定时排行榜，不支持实时排行榜

**set：**

集合的特点是无序和去重

```shell
添加元素
sadd key element [element ...]
计算元素个数（不会遍历集合所有元素，是直接用内部变量）
scard key
判断元素是否在集合中（在则返回1，反之为0）
sismember key element
```

场景：收藏夹；每一个用户做一个收藏的集合，每个收藏的集合存放用户收藏过的歌曲id，key为用户id，value为歌曲id的集合。

**sorted set：**

有序集合的特点是有序，无重复值。

```shell
zadd key score member [score member ...]
成员个数
zcard key
```

场景：实时排行；QQ音乐中有多种实时榜单，比如飙升榜、热歌榜、新歌榜，可以用redis key存储榜单类型，score为点击量，value为歌曲id，用户每点击一首歌曲会更新redis数据，sorted set会依据score即点击量将歌曲id排序。
