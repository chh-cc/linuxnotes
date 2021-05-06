# Elasticsearch

是一个基于Lucene的**搜索服务器**。提供搜集、分析、存储数据 三个功能。Elasticsearch是Java开发的，并作为Apache许可 条款下的开放源码发布，是当前流行的**企业级搜索引擎**。

**9200端口：客户端连接该端口发送和接收数据**

9300端口：分布式节点之间通信的端口

基本组件：

- 索引：文档容器，类似属性文档的集合
- 类型：索引内部的逻辑分区
- 文档：索引和搜索的原子单位
- 映射：原始内容存储为文档之前需要事先进行分析，例如切词、过滤掉某些词等；映射用于定义此分析机制该如何实现

集群组件：

- cluster：ES的集群标识为集群名称；默认为“elasticsearch”，因为节点就是靠此名字来决定加入到哪个集群中，一个节点只能属于一个集群；
- Node：运行于单个ES实例的主机即为节点，用于存储数据，参与集群索引及搜索操作；节点标识靠节点名；
- Shard：将索引切割成为的物理存储组件，但每一个shard都是一个独立且完整的索引；创建索引时，ES默认将其分割成5个shard，用户也可以按需自定义；创建完成后不可修改；

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
git clone git://github.com/mobz/elasticsearch-head.gitcd elasticsearch-headnpm install -g cnpm --registry=https://registry.npm.taobao.orgcnpm install
```

4.启动

```shell
npm run startopen http://localhost:9100/
```

5.elasticsearch-head发现主机 并连接
修改elasticsearch.yml并重启

```shell
http.cors.enabled: truehttp.cors.allow-origin: "*"
```

### 使用

集群健康值的几种状态如下：
绿颜色，最健康的状态，代表所有的分片包括备份都可用
黄颜色，基本的分片可用，但是备份不可用（也可能是没有备份）
红颜色，部分的分片可用，表明分片有一部分损坏。此时执行查询部分数据仍然可以查到，遇到这种情况，还是赶快解决比较好
灰色，未连接到elasticsearch服务

![file](https://gitee.com/c_honghui/picture/raw/master/img/20210429113116.png)

五星代表是主节点，圆代表是从节点

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
#node.data: true
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

processors: 32
# 一般是当前cpu的2倍
thread_pool:
    write:
        size: 32
        # 默认是available processors，由17调整到32，一般是当前cpu的2倍
        queue_size: 1000000
        # 由4000调整大小为1000000
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

