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

监听端口：5601

配置文件kibana.yml

```shell
#####----------kibana服务相关----------#####
#提供服务的端口，监听端口
server.port: 5601
#主机地址，可以是ip,主机名
server.host: 0.0.0.0
#在代理后面运行，则可以指定安装Kibana的路径
#使用server.rewriteBasePath设置告诉Kibana是否应删除basePath
#接收到的请求，并在启动时防止过时警告
#此设置不能以斜杠结尾
server.basePath: ""
#指定Kibana是否应重写以server.basePath为前缀的请求，或者要求它们由反向代理重写，默认false
server.rewriteBasePath: false
#传入服务器请求的最大有效负载大小,以字节为单位，默认1048576
server.maxPayloadBytes: 1048576
#该kibana服务的名称，默认your-hostname
server.name: "your-hostname"
#服务的pid文件路径，默认/var/run/kibana.pid
pid.file: /var/run/kibana.pid
#####----------elasticsearch相关----------#####
#kibana访问es服务器的URL,就可以有多个，以逗号","隔开
elasticsearch.hosts: ["http://localhost:9200"]
#当此值为true时，Kibana使用server.host设定的主机名
#当此值为false时，Kibana使用连接Kibana实例的主机的主机名
#默认ture
elasticsearch.preserveHost: true
#Kibana使用Elasticsearch中的索引来存储已保存的搜索，可视化和仪表板
#如果索引尚不存在，Kibana会创建一个新索引
#默认.kibana
kibana.index: ".kibana"
#加载的默认应用程序
#默认home
kibana.defaultAppId: "home"
#kibana访问Elasticsearch的账号与密码(如果ElasticSearch设置了的话)
elasticsearch.username: "kibana_system"
elasticsearch.password: "pass"
#从Kibana服务器到浏览器的传出请求是否启用SSL
#设置为true时，需要server.ssl.certificate和server.ssl.key
server.ssl.enabled: true
server.ssl.certificate: /path/to/your/server.crt
server.ssl.key: /path/to/your/server.key
#从Kibana到Elasticsearch启用SSL后，ssl.certificate和ssl.key的位置
elasticsearch.ssl.certificate: /path/to/your/client.crt
elasticsearch.ssl.key: /path/to/your/client.key
#PEM文件的路径列表
elasticsearch.ssl.certificateAuthorities: [ "/path/to/your/CA.pem" ]
#控制Elasticsearch提供的证书验证
#有效值为none，certificate和full
elasticsearch.ssl.verificationMode: full
#Elasticsearch服务器响应ping的时间，单位ms
elasticsearch.pingTimeout: 1500
#Elasticsearch 的响应的时间，单位ms
elasticsearch.requestTimeout: 30000
#Kibana客户端发送到Elasticsearch的标头列表
#如不发送客户端标头，请将此值设置为空
elasticsearch.requestHeadersWhitelist: []
#Kibana客户端发往Elasticsearch的标题名称和值
elasticsearch.customHeaders: {}
#Elasticsearch等待分片响应的时间
elasticsearch.shardTimeout: 30000
#Kibana刚启动时等待Elasticsearch的时间，单位ms，然后重试
elasticsearch.startupTimeout: 5000
#记录发送到Elasticsearch的查询
elasticsearch.logQueries: false
#####----------日志相关----------#####
#kibana日志文件存储路径，默认stdout
logging.dest: stdout
#此值为true时，禁止所有日志记录输出
#默认false
logging.silent: false
#此值为true时，禁止除错误消息之外的所有日志记录输出
#默认false
logging.quiet: false
#此值为true时，记录所有事件，包括系统使用信息和所有请求
#默认false
logging.verbose: false
#####----------其他----------#####
#系统和进程取样间隔，单位ms，最小值100ms
#默认5000ms
ops.interval: 5000
#kibana web语言
#默认en
i18n.locale: "en"
```



### 4、Kafka

**数据缓冲队列**。作为消息队列解耦了处理过程，同时提⾼了可扩展 性。具有峰值处理能⼒，使⽤消息队列能够使关键组件顶住突发的访问 压⼒，⽽不会因为突发的超负荷的请求⽽完全崩溃。
1.发布和订阅记录流，类似于消息队列或企业消息传递系统。
2.以容错持久的⽅式存储记录流。
3.处理记录发⽣的流。

### 5、Filebeat

⾪属于Beats,**轻量级数据收集引擎**。基于原先 Logstash-fowarder 的源码改造出来。换句话说：Filebeat就是新版的 Logstashfowarder，也会是 ELK Stack 在 Agent 的第⼀选择。

配置文件：

input配置

```shell
#每一个prospectors，起始于一个破折号”-“
filebeat.prospectors:
#默认log，从日志文件读取每一行。stdin，从标准输入读取
- input_type: log
#日志文件路径列表，可用通配符，不递归
paths:     - /var/log/*.log
#编码，默认无，plain(不验证或者改变任何输入), latin1, utf-8, utf-16be-bom, utf-16be, utf-16le, big5, gb18030, gbk, hz-gb-2312, euc-kr, euc-jp, iso-2022-jp, shift-jis
encoding: plain
#匹配行，后接一个正则表达式列表，默认无，如果启用，则filebeat只输出匹配行，如果同时指定了多行匹配，仍会按照include_lines做过滤
include_lines: [‘^ERR’, ‘^WARN’]
#排除行，后接一个正则表达式的列表，默认无
#排除文件，后接一个正则表达式的列表，默认无
exclude_lines: [“^DBG”]
#排除更改时间超过定义的文件，时间字符串可以用2h表示2小时，5m表示5分钟，默认0
ignore_older: 5m
#该type会被添加到type字段，对于输出到ES来说，这个输入时的type字段会被存储，默认log
document_type: log
#prospector扫描新文件的时间间隔，默认10秒
scan_frequency: 10s
#单文件最大收集的字节数，单文件超过此字节数后的字节将被丢弃，默认10MB，需要增大，保持与日志输出配置的单文件最大值一致即可
max_bytes: 10485760
#多行匹配模式，后接正则表达式，默认无
multiline.pattern: ^[
#多行匹配模式后配置的模式是否取反，默认false
multiline.negate: false
#定义多行内容被添加到模式匹配行之后还是之前，默认无，可以被设置为after或者before
multiline.match: after
#单一多行匹配聚合的最大行数，超过定义行数后的行会被丢弃，默认500
multiline.max_lines: 500
#多行匹配超时时间，超过超时时间后的当前多行匹配事件将停止并发送，然后开始一个新的多行匹配事件，默认5秒
multiline.timeout: 5s
#可以配置为true和false。配置为true时，filebeat将从新文件的最后位置开始读取，如果配合日志轮循使用，新文件的第一行将被跳过
tail_files: false
#当文件被重命名或被轮询时关闭重命名的文件处理。注意：潜在的数据丢失。请务必阅读并理解此选项的文档。默认false
close_renamed: false
#如果文件不存在，立即关闭文件处理。如果后面文件又出现了，会在scan_frequency之后继续从最后一个已知position处开始收集，默认true
close_removed: true
#每个prospectors的开关，默认true
enabled: true
#后台事件计数阈值，超过后强制发送，默认2048
filebeat.spool_size: 2048
#后台刷新超时时间，超过定义时间后强制发送，不管spool_size是否达到，默认5秒
filebeat.idle_timeout: 5s
#注册表文件，同logstash的sincedb，记录日志文件信息，如果使用相对路径，则意味着相对于日志数据的路径
filebeat.registry_file: ${path.data}/registry
#定义filebeat配置文件目录，必须指定一个不同于filebeat主配置文件所在的目录，目录中所有配置文件中的全局配置会被忽略
filebeat.config_dir
```

output.elasticsearch

```shell
#启用模块
enabled: true
#ES地址
hosts: [“localhost:9200”]
#gzip压缩级别，默认0，不压缩，压缩耗CPU
compression_level: 0
#每个ES的worker数，默认1
worker: 1
#可选配置，ES索引名称，默认filebeat-%{+yyyy.MM.dd}
index: “filebeat-%{+yyyy.MM.dd}”
#可选配置，输出到ES接收节点的pipeline，默认无
pipeline: “”
#可选的，HTTP路径，默认无
path: “/elasticsearch”
#http代理服务器地址，默认无
proxy_url: http://proxy:3128
#ES重试次数，默认3次，超过3次后，当前事件将被丢弃
max_retries: 3
#对一个单独的ES批量API索引请求的最大事件数,默认50
bulk_max_size: 50
#到ES的http请求超时时间,默认90秒
timeout: 90
```

output.logstash

```shell
#启用模块
enabled: true
#logstash地址
hosts: [“localhost:5044”]
#每个logstash的worker数，默认1
worker: 1
#压缩级别，默认3
compression_level: 3
#负载均衡开关，在不同的logstash间负载
loadbalance: true
#在处理新的批量期间，异步发送至logstash的批量次数
pipelining: 0
#可选配置，索引名称，默认为filebeat
index: ‘filebeat’
#socks5代理服务器地址
proxy_url: socks5://user:password@socks5-server:2233
#使用代理时是否使用本地解析，默认false
proxy_use_local_resolver: false
```

output.redis

```shell
#启用模块
enabled: true
#logstash地址
hosts: [“localhost:6379”]
#redis地址，地址为一个列表，如果loadbalance开启，则负载到里表中的服务器，当一个redis服务器不可达，事件将被分发到可到达的redis服务器
worker: 1
#redis端口，如果hosts内未包含端口信息，默认6379
port: 6379
#事件发布到redis的list或channel，默认filebeat
key: filebeat
#redis密码，默认无
password:
#redis的db值，默认0
db: 0
#发布事件使用的redis数据类型，如果为list，使用RPUSH命令（生产消费模式）。如果为channel，使用PUBLISH命令{发布订阅模式}。默认为list
datatype: list
#为每个redis服务器启动的工作进程数，会根据负载均衡配置递增
worker: 1
#负载均衡，默认开启
loadbalance: true
#redis连接超时时间，默认5s
timeout: 5s
#filebeat会忽略此设置，并一直重试到全部发送为止，其他beat设置为0即忽略，默认3
max_retries: 3
#对一个redis请求或管道批量的最大事件数，默认2048
bulk_max_size: 2048
#socks5代理地址，必须使用socks5://
proxy_url:
#使用代理时是否使用本地解析，默认false
proxy_use_local_resolver: false
```

