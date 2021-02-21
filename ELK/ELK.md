# ELK

## 简介

ELK由`Elasticsearch`、`Logstash`和`Kibana`三部分组件组成；

Elasticsearch是个开源分布式搜索引擎，它的特点有：分布式，零配置，自动发现，索引自动分片，索引副本机制，restful风格接口，多数据源，自动搜索负载等。

Logstash是一个完全开源的工具，它可以对你的日志进行收集、分析，并将其存储供以后使用

kibana 是一个开源和免费的工具，它可以为 Logstash 和 ElasticSearch 提供的日志分析友好的 Web 界面，可以帮助您汇总、分析和搜索重要数据日志。

## 架构

### Elasticsearch + Logstash + Kibana

![image-20210221174418582](https://gitee.com/c_honghui/picture/raw/master/img/20210221174418.png)

- 这是一种最`简单`的架构。这种架构，通过logstash收集日志，Elasticsearch分析日志，然后在Kibana(web界面)中展示。这种架构虽然是官网介绍里的方式，但是往往在生产中很少使用。

### Elasticsearch + Logstash + filebeat + Kibana

- 与上一种架构相比，这种架构增加了一个`filebeat`模块。filebeat是一个轻量的日志收集代理，用来部署在客户端，优势是消耗非常少的资源(较logstash)， 所以生产中，往往会采取这种架构方式，但是这种架构有一个缺点，当logstash出现故障， 会造成日志的丢失。

![image-20210221174432189](https://gitee.com/c_honghui/picture/raw/master/img/20210221174432.png)

### Elasticsearch + Logstash + filebeat + redis（kafka）

![image-20210221174600614](https://gitee.com/c_honghui/picture/raw/master/img/20210221174600.png)

- 这种架构是上面那个架构的完善版，通过`增加中间件`，来避免数据的丢失。当Logstash出现故障，日志还是存在中间件中，当Logstash再次启动，则会读取中间件中积压的日志。目前我司使用的就是这种架构，我个人也比较推荐这种方式。

## 组件

### 1、Elasticsearch  

是⼀个基于Lucene的**搜索服务器**。提供搜集、分析、存储数据 三⼤功能。它提供了⼀个分布式多⽤户能⼒的全⽂搜索引擎，基于 RESTful web接⼝。Elasticsearch是⽤Java开发的，并作为Apache许可 条款下的开放源码发布，是当前流⾏的**企业级搜索引擎**。设计⽤于云计 算中，能够达到实时搜索，稳定，可靠，快速，安装使⽤⽅便。

9200端口：客户端连接该端口发送和接收数据

9300端口：分布式节点之间通信的端口

**配置文件elasticsearch.yml**

```shell
# ---------------------------------- Cluster -----------------------------------
#es集群名称，es会自动发现在同一网段下的es，如果在同一网段下有多个集群，就可以用这个属性来区分不同的集群
#识别集群的标识，同一个集群名字必须相同
cluster.name: my-application
# ------------------------------------ Node ------------------------------------
#该节点名称，自定义或者默认
node.name: node-1
#该节点是否可以成为一个master节点
node.master: true 
#该节点是否存储数据，即是否是一个数据节点，默认true
node.data: true
#节点的通用属性，用于后期集群进行碎片分配时的过滤
node.attr.rack: r1
# ----------------------------------- Paths ------------------------------------
#配置文件路径，默认es安装目录下的config
path.conf: /path/to/conf
#数据存储路径，默认es安装目录下的data
#可以设置多个存储路径，用逗号隔开
path.data: /path/to/data
#日志路径，默认es安装目录下的logs
path.logs: /path/to/logs
#临时文件路径，默认es安装目录下的work
path.work: /path/to/work 
#插件存放路径，默认es安装目录下的plugins
path.plugins: /path/to/plugins 
# ----------------------------------- Memory -----------------------------------
#当JVM开始写入交换空间时（swapping）ElasticSearch性能会低下
#设置为true来锁住内存,同时也要允许elasticsearch的进程可以锁住内存,linux下可以通过 `ulimit -l unlimited` 命令 
bootstrap.memory_lock: true
# ---------------------------------- Network -----------------------------------
#该节点绑定的地址，即对外服务的地址，可以是IP，主机名
network.host: 0.0.0.0
#该节点对外服务的http端口，默认9200
http.port: 9200
#节点间交互的tcp端口,默认9300
transport.tcp.port: 9300
#HTTP请求的最大内容，默认100MB
http.max_content_length: 100MB
#HTTP URL的最大长度，默认为4KB
http.max_initial_line_length: 4KB
#允许的标头的最大大小，默认为8KB
http.max_header_size: 8KB
#压缩，默认true
http.compression: true
#压缩级别，有效值:1-9，默认为3
http.compression_level: 3
#是否开启http协议对外提供服务,默认为true
http.enabled: true
# --------------------------------- Discovery ----------------------------------
#集群列表
#port为节点间交互端口，未设置时，默认9300
discovery.seed_hosts: ["host1:port", "ip2:port"]
#初始主节点列表
cluster.initial_master_nodes: ["node-1", "node-2"]
# ---------------------------------- Gateway -----------------------------------
#gateway的类型,默认为local，即为本地文件系统
gateway.type: local 
#集群中的N个节点启动后,才允许进行恢复处理，默认3
gateway.recover_after_nodes: 3
#设置初始化恢复过程的超时时间,超时时间从上一个配置中配置的N个节点启动后算起 
gateway.recover_after_time: 5m 
#设置这个集群中期望有多少个节点，一旦这N个节点启动，立即开始恢复过程
gateway.expected_nodes: 2
# ---------------------------------- Various -----------------------------------
#删除索引时需要显式名称
action.destructive_requires_name: true
```

**elastic-head视图插件**

1. 集群健康值的几种状态如下：
   绿颜色，最健康的状态，代表所有的分片包括备份都可用
   黄颜色，基本的分片可用，但是备份不可用（也可能是没有备份）
   红颜色，部分的分片可用，表明分片有一部分损坏。此时执行查询部分数据仍然可以查到，遇到这种情况，还是赶快解决比较好
   灰色，未连接到elasticsearch服务

![file](https://gitee.com/c_honghui/picture/raw/master/img/20210222001554.png)

2. 五星代表是主节点，圆代表是从节点

###  2、Logstash  

主要是⽤来**⽇志的搜集、分析、过滤⽇志的⼯具**。⽤于管理⽇志 和事件的⼯具，你可以⽤它去收集⽇志、转换⽇志、解析⽇志并将他们 作为数据提供给其它模块调⽤，例如搜索、存储等。

配置文件logstash.conf
该文件定义了logstash从哪里获取输入，然后输出到哪里

```shell
#从Beats输入，以json格式输出到Elasticsearch
input {
  beats {
    port => 5044
    codec => 'json'
  }
}
output {
  elasticsearch {
    #elasticsearch地址，多个以','隔开
    hosts => ["http://localhost:9200"]
    #创建的elasticsearch索引名，可以自定义也可以使用下面的默认
    index => "%{[@metadata][beat]}-%{[@metadata][version]}-%{+YYYY.MM.dd}"
  }
}
#从kafka输入，以json格式输出到Elasticsearch
input {
  kafka {
    #kafka地址，多个用逗号','隔开
    bootstrap_servers => ["ip1:port2","ip2:port2"]
    #消费者组
    group_id => 'test'       
    # kafka topic 名称    
    topics => 'logstash-topic' 
    codec => 'json'
  }
}
output {
  elasticsearch {
    hosts => ["http://localhost:9200"]
    index => "%{[@metadata][beat]}-%{[@metadata][version]}-%{+YYYY.MM.dd}"
  }
}
```

###  3、Kibana  

是⼀个优秀的**前端⽇志展示框架**，它可以⾮常详细的将⽇志转化 为各种图表，为⽤户提供强⼤的数据可视化⽀持,它能够搜索、展示存 储在 Elasticsearch 中索引数据。使⽤它可以很⽅便的⽤图表、表格、 地图展示和分析数据。 

### 4、Kafka

**数据缓冲队列**。作为消息队列解耦了处理过程，同时提⾼了可扩展 性。具有峰值处理能⼒，使⽤消息队列能够使关键组件顶住突发的访问 压⼒，⽽不会因为突发的超负荷的请求⽽完全崩溃。
1.发布和订阅记录流，类似于消息队列或企业消息传递系统。
2.以容错持久的⽅式存储记录流。
3.处理记录发⽣的流。

### 5、Filebeat

⾪属于Beats,**轻量级数据收集引擎**。基于原先 Logstash-fowarder 的源码改造出来。换句话说：Filebeat就是新版的 Logstashfowarder，也会是 ELK Stack 在 Agent 的第⼀选择。