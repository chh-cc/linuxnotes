# Redis入门简介

## 简介

Redis是内存数据库、kv数据库、以及数据结构数据库

单线程和I/O多路复用模型来实现高性能



Redis在奇数版本为非稳定版本，例如2.7，3.1。如果为**偶数则为稳定版本**，例如3.2，3.4

- 官方网站：[https://redis.io](https://redis.io/)
- 官方各版本下载地址：<http://download.redis.io/releases/>
- Redis 中文命令参考：[http://redisdoc.com](http://redisdoc.com/)
- 中文网站1：[http://redis.cn](http://redis.cn/)
- 中文网站2：[http://www.redis.net.cn](http://www.redis.net.cn/)

### 应用场景

redis使用场景很明确，作为高速缓存或key-value数据库使用，当作为key-value数据库使用时，一般会开启持久化存储，但它的持久化数据是以快照形式存在，所以当重启导致内存数据失效、尝试用本地持久化数据恢复时，有可能会造成数据丢失

另外恢复也需要额外的时间，所以更常见的做法是以集群的形式对外提供服务，集群内可以分片，分片内还可以有主从、主备等各种策略来保证可靠性和效率



阿里云redis：[云数据库 Redis (aliyun.com)](https://help.aliyun.com/product/26340.html)



场景：秒杀订单数据、用户定位数据、弹幕信息、排行榜信息等，这些数据都有一个特点：瞬时高并发、实时记录、数据有过期性质、实时性很重要

### 用缓存会有什么不良后果

缓存与数据库双写不一致

缓存雪崩、缓存穿透、缓存击穿

缓存并发竞争



## 信息

- **默认端口**：TCP`6379`
- 编写语言：c
- 启动redis：`redis-server`
- redis客户端：`redis-cli`
- redis基准测试工具：`redis-benchmark`
- AOF文件检测和修复工具：`redis-check-aof`
- ADB文件检测和修复工具：`redis-check-dump`
- 启动哨兵：`redis-sentinel`



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

### string

信息缓存、计数器、分布式锁等

主要命令

```shell
#设置key的value值
set key value [ex seconds] [px milliseconds] [nx|xx]
ex seconds:为键设置秒级过期时间
px milliseconds：为键设置毫秒级过期时间
nx：键必须不存在才能设置
xx：键必须存在才能设置

#获取key的value值
get key

mset key value [key value ...]
mget key [key ...]

#计数
incr key
值不是整数，返回错误
值是整数返回自增后结果
键不存在按照0自增，返回1

#删除key
del key
```

场景1：缓存功能，如用户信息

方案：先从redis读取，没有值就从mysql读取，并写一份到redis作缓存，要设置过期时间

键值设计：直接将用户一条mysql记录做序列化(通常序列化为json)作为值，userInfo:userid 作为key，键名如：userInfo:123，value存储对应用户信息的json串。如 key为："user:id :name:1", value为"{"name":"leijia","age":18}"。

场景2：计数

场景3：分布式session

如果你的应用做了负载均衡，将网站的项目放在多个服务器上，当用户在服务器A上进行登陆，session文件会写在A服务器；当用户跳转页面时，请求被分配到B服务器上的时候，就找不到这个session文件，用户就要重新登陆。

可以将session存放在redis中，redis可以独立于所有负载均衡服务器，也可以放在其中一台负载均衡服务器上；但是所有应用所在的服务器连接的都是同一个redis服务器。

场景4：短信验证码限速

### hash

键值本身就是一个键值对结构

场景：以购物车为例子，用户id设置为key，那么购物车里所有的商品就是用户key对应的值了，每个商品有id和购买数量，对应hash的结构就是商品id为field，商品数量为value。

主要命令：

```shell
#获取key对应hash中的field对应的值
hget key field
hget person name
#设置key对应hash中的field对应的值
hset key field value
hset person name bingo
hset person age 20
hset person id 1
#设置多个hash键值
hmset key field1 value1 field2 value2 ...
hmget key field1 field2 ...
#获取key对应的hash有多少个键值对
hlen key
#删除key对应的hash键值对
(person = {
  "name": "bingo",
  "age": 20,
  "id": 1
})
```

### list

列表用来存储多个有序的可重复的字符串。

场景：朋友圈消息推送

主要命令

```shell
#从左到右获取列表所有元素
lrange listkey 0 -1
#获取第2到第4个元素
lrange listkey 1 3
#获取当前列表最后一个元素
lindex listkey -1
#获取列表长度
llen key

#从队列右边插入元素
rpush key value [value ...]
#从队列右边弹出一个元素
rpop key
#从队列左边插入元素
lpush key value [value ...]
#从队列左边弹出一个元素
lpop key
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
