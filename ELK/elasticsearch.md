# Elasticsearch

是一个基于Lucene的**搜索服务器**。提供搜集、分析、存储数据 三个功能。Elasticsearch是Java开发的，并作为Apache许可 条款下的开放源码发布，是当前流行的**企业级搜索引擎**。

**9200端口：客户端连接该端口发送和接收数据**

9300端口：分布式节点之间通信的端口

## 配置文件

vim elasticsearch.yml

```shell
# ---------------------------------- Cluster -----------------------------------
#es集群名称，es会自动发现在同一网段下的es，如果在同一网段下有多个集群，就可以用这个属性来区分不同的集群
#识别集群的标识，同一个集群名字必须相同
cluster.name: my-application
# ------------------------------------ Node ------------------------------------
#该节点名称，自定义或者默认
node.name: ${HOSTNAME}
#该节点是否可以成为一个master节点
node.master: true 
#该节点是否存储数据，即是否是一个数据节点，默认true
node.data: true
#节点的通用属性，用于后期集群进行碎片分配时的过滤
#node.attr.rack: r1

#index.number_of_shard: 5   【修改每个索引默认shard的数量】
#index.number_of_replica: 1   【修改每个shard的副本有几个】
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
# 一般是当前cpu的2倍
processors: 32

thread_pool:
    write:
        # 默认是available processors，由17调整到32，一般是当前cpu的2倍
        size: 32
        # 由4000调整大小为1000000
        queue_size: 1000000
    search:
        size: 128
        queue_size: 5000
        min_queue_size: 1000
        max_queue_size: 10000
        auto_queue_frame_size: 2000
        target_response_time: 1s
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
discovery.seed_hosts: ["ip1:port", "ip2:port"]
#集群节点IP列表。提供了自动组织集群，自动扫描端口9300-9305连接其他节点。无需额外配置
discovery.zen.ping.unicast.hosts: ["192.168.1.195", "192.168.1.196", "192.168.1.197"]
#最少主节点数,为避免脑裂应设置符合节点的法定人数：(nodes / 2 ) + 1
discovery.zen.minimum_master_nodes: 2
#初始主节点列表
cluster.initial_master_nodes: ["ip1", "ip2"]
#单机运行
#discovery.type: single-node
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
http.cors.enabled: true
http.cors.allow-origin: "*"
#删除索引时需要显式名称
action.destructive_requires_name: true
action.auto_create_index:  "*"
xpack.security.enabled: false
xpack.monitoring.enabled: true
xpack.graph.enabled: false
xpack.watcher.enabled: false
xpack.ml.enabled: false
```

## 概念

Node：运行单个ES实例的服务器 

Cluster：一个或多个节点构成集群 

Index：索引是多个文档的集合 

Document：Index里每条记录称为Document，若干文档构建一个Index 

Type：一个Index可以定义一种或多种类型，将Document逻辑分组 

Field：ES存储的最小单元 

Shards：ES将Index分为若干份，每一份就是一个分片 

Replicas：Index的一份或多份副本

| elasticsearch | 关系型数据库（如mysql） |
| ------------- | ----------------------- |
| index         | database                |
| type          | table                   |
| document      | row（行）               |
| field         | Column（字段）          |

## 数据操作

用curl操作会比较麻烦

curl -X<verb> '<protocol>://<host>:<port>/<path>?<query_string>' -d '<body>'

| 参数         | 描述                                     |
| ------------ | ---------------------------------------- |
| verb         | http方法，如get、post、put、head、delete |
| host         | es集群中的任意节点主机名                 |
| port         | es http服务端口，默认9200                |
| path         | 索引路径                                 |
| query_string | 可选的查询请求参数                       |
| -d           | 里面放一个get的json格式请求主体          |
| body         | 自己写的json格式的请求主体               |

列出所有索引：

https://www.elastic.co/guide/en/elasticsearch/reference/6.2/_list_all_indices.html

创建索引：

https://www.elastic.co/guide/en/elasticsearch/reference/6.2/_create_an_index.html

curl -X PUT "192.168.0.212:9200/logs-2021.10.20"

插入和查询文档：

https://www.elastic.co/guide/en/elasticsearch/reference/6.2/_index_and_query_a_document.html

```shell
curl -X PUT "192.168.0.212:9200/customer/_doc/1?pretty" -H 'Content-Type: application/json' -d' { "name": "John Doe" } '
{
  "_index" : "customer",
  "_type" : "_doc",
  "_id" : "1",
  "_version" : 1,
  "result" : "created",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 0,
  "_primary_term" : 1
}

curl -X GET "192.168.0.212:9200/customer/_doc/1?pretty"
{
  "_index" : "customer",
  "_type" : "_doc",
  "_id" : "1",
  "_version" : 1,
  "found" : true,
  "_source" : { "name": "John Doe" }
}
```

查看集群节点，标记为*的为master节点：

curl -XGET 'http://127.0.0.1:9200/_cat/nodes?pretty'

## elastic-head视图插件

ealsticsearch只是后端提供各种api，可以添加索引查看数据等，但需要在命令行操作，elasticsearch-head是一个基于node.js的前端工程，可以提供WEB方式的信息查看和各种API的视图化操作。

### 安装

1.nodejs安装

```shell
wget https://nodejs.org/dist/v10.9.0/node-v10.9.0-linux-x64.tar.xz
tar xf  node-v10.9.0-linux-x64.tar.xz
echo 'PATH=$PATH:/usr/local/node-v10.9.0-linux-x64/bin' >> /etc/profilesource /etc/profile
```

2.phantomjs安装配置

```shell
Wget https://github.com/Medium/phantomjs/releases/download/v2.1.1/phantomjs-2.1.1-linux-x86_64.tar.bz2
yum -y install bzip2
tar –jxvf  phantomjs-2.1.1-linux-x86_64.tar.bz2
echo 'export PATH=$PATH:/usr/local/phantomjs-2.1.1-linux-x86_64/bin' >> /etc/profilesource /etc/profile
```

3.elasticsearch-head安装

```shell
git clone git://github.com/mobz/elasticsearch-head.gitcd
cd elasticsearch-head
npm install
如果报错，执行操作：
vim Gruntfile.js
options: {
       port: 9100,
       ...
       hostname: '*' #增加这行内容
}
```

4.启动

```shell
npm run start
open http://localhost:9100/
```

5.elasticsearch-head发现主机 并连接
修改elasticsearch.yml并重启

```shell
http.cors.enabled: true
http.cors.allow-origin: "*"
```

### 使用

集群健康值的几种状态如下：
绿颜色，最健康的状态，代表所有的分片包括备份都可用
黄颜色，基本的分片可用，但是备份不可用（也可能是没有备份）
红颜色，部分的分片可用，表明分片有一部分损坏。此时执行查询部分数据仍然可以查到，遇到这种情况，还是赶快解决比较好
灰色，未连接到elasticsearch服务

![file](https://gitee.com/c_honghui/picture/raw/master/img/20210429113116.png)

五星代表是主节点，圆代表是从节点

