# ELK

## 产品兼容性

ES:

```text
说明
^：Elasticsearch作为output插件的兼容性（Beats或Logstash同步索引数据到Elasticsearch）。
*：建议您运行最新版本的Beats、Logstash以及ES-Hadoop，早期版本的功能比较少。
**：从6.3版本开始，X-Pack功能已包含在Elastic Stack的默认发行版中
```

| Elasticsearch | Kibana | X-Pack | Beats^*     | Logstash^*  | ES-Hadoop（jar）* | APM Server  | App Search |
| :------------ | :----- | :----- | :---------- | :---------- | :---------------- | :---------- | :--------- |
| 5.0.x         | 5.0.x  | 5.0.x  | 1.3.x-5.6.x | 2.4.x-5.6.x | 5.0.x-5.6.x       | N/A         | N/A        |
| 5.1.x         | 5.1.x  | 5.1.x  | 1.3.x-5.6.x | 2.4.x-5.6.x | 5.0.x-5.6.x       | N/A         | N/A        |
| 5.2.x         | 5.2.x  | 5.2.x  | 1.3.x-5.6.x | 2.4.x-5.6.x | 5.0.x-5.6.x       | N/A         | N/A        |
| 5.3.x         | 5.3.x  | 5.3.x  | 1.3.x-5.6.x | 2.4.x-5.6.x | 5.0.x-5.6.x       | N/A         | N/A        |
| 5.4.x         | 5.4.x  | 5.4.x  | 1.3.x-5.6.x | 2.4.x-5.6.x | 5.0.x-5.6.x       | N/A         | N/A        |
| 5.5.x         | 5.5.x  | 5.5.x  | 1.3.x-5.6.x | 2.4.x-5.6.x | 5.0.x-5.6.x       | N/A         | N/A        |
| 5.6.x         | 5.6.x  | 5.6.x  | 1.3.x-6.0.x | 2.4.x-6.0.x | 5.0.x-6.0.x       | N/A         | N/A        |
| 6.0.x         | 6.0.x  | 6.0.x  | 5.6.x-6.8.x | 5.6.x-6.8.x | 6.0.x-6.8.x       | N/A         | N/A        |
| 6.1.x         | 6.1.x  | 6.1.x  | 5.6.x-6.8.x | 5.6.x-6.8.x | 6.0.x-6.8.x       | N/A         | N/A        |
| 6.2.x         | 6.2.x  | 6.2.x  | 5.6.x-6.8.x | 5.6.x-6.8.x | 6.0.x-6.8.x       | 6.2.x-6.8.x | N/A        |
| 6.3.x         | 6.3.x  | N/A**  | 5.6.x-6.8.x | 5.6.x-6.8.x | 6.0.x-6.8.x       | 6.2.x-6.8.x | N/A        |
| 6.4.x         | 6.4.x  | N/A**  | 5.6.x-6.8.x | 5.6.x-6.8.x | 6.0.x-6.8.x       | 6.2.x-6.8.x | N/A        |
| 6.5.x         | 6.5.x  | N/A**  | 5.6.x-6.8.x | 5.6.x-6.8.x | 6.0.x-6.8.x       | 6.2.x-6.8.x | N/A        |
| 6.6.x         | 6.6.x  | N/A**  | 5.6.x-6.8.x | 5.6.x-6.8.x | 6.0.x-6.8.x       | 6.2.x-6.8.x | N/A        |
| 6.7.x         | 6.7.x  | N/A**  | 5.6.x-6.8.x | 5.6.x-6.8.x | 6.0.x-6.8.x       | 6.2.x-6.8.x | N/A        |
| 6.8.x         | 6.8.x  | N/A**  | 5.6.x-6.8.x | 5.6.x-6.8.x | 6.0.x-6.8.x       | 6.2.x-6.8.x | N/A        |
| 7.0.x         | 7.0.x  | N/A**  | 6.8.x-7.5.x | 6.8.x-7.5.x | 6.8.x-7.5.x       | 7.0.x-7.5.x | N/A        |
| 7.1.x         | 7.1.x  | N/A**  | 6.8.x-7.5.x | 6.8.x-7.5.x | 6.8.x-7.5.x       | 7.0.x-7.5.x | N/A        |
| 7.2.x         | 7.2.x  | N/A**  | 6.8.x-7.5.x | 6.8.x-7.5.x | 6.8.x-7.5.x       | 7.0.x-7.5.x | 7.2.x      |
| 7.3.x         | 7.3.x  | N/A**  | 6.8.x-7.5.x | 6.8.x-7.5.x | 6.8.x-7.5.x       | 7.0.x-7.5.x | 7.3.x      |
| 7.4.x         | 7.4.x  | N/A**  | 6.8.x-7.5.x | 6.8.x-7.5.x | 6.8.x-7.5.x       | 7.0.x-7.5.x | 7.4.x      |
| 7.5.x         | 7.5.x  | N/A**  | 6.8.x-7.5.x | 6.8.x-7.5.x | 6.8.x-7.5.x       | 7.0.x-7.5.x | 7.5.x      |

Logstash:

```text
说明
*：此兼容性适用于监控和管理Elasticsearch集群，以及包含xpack.monitoring.elasticsearch.url和xpack.management.elasticsearch.url指定的任意集群。强烈建议您使用相同小版本的Elasticsearch、Kibana和Logstash集群，以获得最佳的监控和管理性能；对于6.2及之前的旧版本集群，X-Pack必须安装在所有产品中。
**：7.4版本之前，Functionbeat仅支持Elasticsearch作为output，不支持Logstash或其他output；7.4以及之后版本，Functionbeat支持Logstash和Elasticsearch作为output。
```



| Logstash | Beats**     | Monitoring & Management Elasticsearch Cluster* |
| :------- | :---------- | :--------------------------------------------- |
| 2.4.x    | 1.0.x-5.6.x | N/A                                            |
| 5.0.x    | 1.3.x-5.6.x | N/A                                            |
| 5.1.x    | 5.0.x-5.6.x | N/A                                            |
| 5.2.x    | 5.0.x-5.6.x | 5.2.x-5.6.x                                    |
| 5.3.x    | 5.0.x-5.6.x | 5.3.x-5.6.x                                    |
| 5.4.x    | 5.0.x-5.6.x | 5.4.x-5.6.x                                    |
| 5.5.x    | 5.0.x-5.6.x | 5.5.x-5.6.x                                    |
| 5.6.x    | 5.6.x-6.8.x | 5.6.x-6.0.x                                    |
| 6.0.x    | 5.6.x-6.8.x | 6.0.x-6.8.x                                    |
| 6.1.x    | 5.6.x-6.8.x | 6.1.x-6.8.x                                    |
| 6.2.x    | 5.6.x-6.8.x | 6.2.x-6.8.x                                    |
| 6.3.x    | 5.6.x-6.8.x | 6.3.x-6.8.x                                    |
| 6.4.x    | 5.6.x-6.8.x | 6.4.x-6.8.x                                    |
| 6.5.x    | 5.6.x-6.8.x | 6.5.x-6.8.x                                    |
| 6.6.x    | 5.6.x-6.8.x | 6.6.x-6.8.x                                    |
| 6.7.x    | 5.6.x-6.8.x | 6.7.x-6.8.x                                    |
| 6.8.x    | 5.6.x-6.8.x | 6.8.x                                          |
| 7.0.x    | 6.8.x-7.5.x | 7.0.x - 7.5.x                                  |
| 7.1.x    | 6.8.x-7.5.x | 7.1.x-7.5.x                                    |
| 7.2.x    | 6.8.x-7.5.x | 7.2.x-7.5.x                                    |
| 7.3.x    | 6.8.x-7.5.x | 7.3.x-7.5.x                                    |
| 7.4.x    | 6.8.x-7.5.x | 7.4.x - 7.5.x                                  |
| 7.5.x    | 6.8.x-7.5.x | 7.5.x                                          |

Beat:

```text
说明
*：此兼容性适用于监控Elasticsearch集群，以及包含xpack.monitoring.elasticsearch设置中指定的任意集群。强烈建议您使用相同小版本的Elasticsearch、Kibana和Beats集群，以获得最佳的监控性能；对于6.2及之前的旧版本集群，X-Pack必须安装在所有产品中。
**：Functionbeat仅支持Elasticsearch作为output，不支持Logstash或其他output。
```



| Beats** | Logstash    | Monitoring Elasticsearch Cluster* |
| :------ | :---------- | :-------------------------------- |
| 1.3.x   | 2.0.x-5.0.x | N/A                               |
| 5.0.x   | 2.0.x-5.6.x | N/A                               |
| 5.1.x   | 2.0.x-5.6.x | N/A                               |
| 5.2.x   | 2.0.x-5.6.x | N/A                               |
| 5.3.x   | 2.0.x-5.6.x | N/A                               |
| 5.4.x   | 2.0.x-5.6.x | N/A                               |
| 5.5.x   | 2.0.x-5.6.x | N/A                               |
| 5.6.x   | 5.6.x-6.8.x | N/A                               |
| 6.0.x   | 5.6.x-6.8.x | N/A                               |
| 6.1.x   | 5.6.x-6.8.x | N/A                               |
| 6.2.x   | 5.6.x-6.8.x | 6.2.x                             |
| 6.3.x   | 5.6.x-6.8.x | 6.3.x-6.8.x                       |
| 6.4.x   | 5.6.x-6.8.x | 6.4.x-6.8.x                       |
| 6.5.x   | 5.6.x-6.8.x | 6.5.x-6.8.x                       |
| 6.6.x   | 5.6.x-6.8.x | 6.6.x-6.8.x                       |
| 6.7.x   | 5.6.x-6.8.x | 6.7.x-6.8.x                       |
| 6.8.x   | 5.6.x-6.8.x | 6.8.x                             |
| 7.0.x   | 6.8.x-7.5.x | 7.0.x-7.5.x                       |
| 7.1.x   | 6.8.x-7.5.x | 7.1.x-7.5.x                       |
| 7.2.x   | 6.8.x-7.5.x | 7.2.x-7.5.x                       |
| 7.3.x   | 6.8.x-7.5.x | 7.3.x-7.5.x                       |
| 7.4.x   | 6.8.x-7.5.x | 7.4.x - 7.5.x                     |
| 7.5.x   | 6.8.x-7.5.x | 7.5.x                             |

## 基本概念

集群:

一个es集群由一个或多个es节点组成,并提供集群内所有节点的联合索引和搜索能力.
一个集群被命名为唯一的名字（默认为elasticsearch），集群名称非常重要，因为节点需要通过集群的名称加入集群。

节点:

一个节点是集群中的一个服务器，用来存储数据并参与集群的索引和搜索。

索引:

一个索引是一个拥有一些相似特征的文档的集合,通常使用一个名称（字母必须小写）来标识

类型：

目前已经不支持在一个索引下创建多个类型，并且类型概念已经在后续版本中删除

文档：

一个文档是可以被索引的基本信息单元

字段：

组成文档的最小单位

映射：

定义一个文档及其所包含的字段如何被存储和索引

分片：

Elasticsearch可以把一个完整的索引分成多个分片，这样的好处是可以把一个大的索引拆分成多个，分布到不同的节点上，构成分布式搜索。分片的数量只能在索引创建前指定，并且索引创建后不能更改。
Elasticsearch 7.0以下版本默认为一个索引创建5个主分片，并分别为每个主分片创建1个副本分片，7.0及以上版本默认创建1个主分片和1个副本分片。

## Elasticsearch

1. 创建实例

2. 访问实例

   1. 在**实例列表**中，单击目标实例ID。

   2. 在左侧导航栏，单击**可视化控制**。

   3. 在**Kibana**区域中，单击**进入控制台**。

   4. 在登录页面输入账号和密码，单击登录。

   5. 在登录成功页面，单击**Explore on my own**。

   6. 在左侧导航栏，单击**Dev Tools**（开发工具），再单击**Go to work**。

   7. 在**Console**中，执行如下命令访问Elasticsearch实例。

      ```
      GET /
      ```

## Logstash

Logstash能够动态地从多个来源采集数据、转换数据，并且将数据存储到所选择的位置。通过输入、过滤和输出插件，Logstash可以对任何类型的事件加工和转换。

集成官方全部Input、Output、Filter插件。

1. 创建实例

2. 创建并运行管道任务

   1. 在左侧导航栏，单击**管道管理**。
   2. 在**管道列表**区域，单击**创建管道**。
   3. 在**Config配置**中，输入**管道ID**并配置管道。

   ```shell
   #指定输入数据源
   input {
       kafka {
       bootstrap_servers => ["192.168.xx.xx:9092,192.168.xx.xx:9092,192.168.xx.xx:9092"]
       group_id => "group_1"
       topics => ["logstash_test"]
       consumer_threads => 6
       decorate_events => true
       }
   }
   #指定对输入数据进行过滤的插件
   filter {
   }
   #指定目标数据源类型
   output {
     elasticsearch {
       #阿里云Elasticsearch服务的访问地址。
       hosts => ["http://es-cn-mp91cbxsm000c****.elasticsearch.aliyuncs.com:9200"]
       #访问阿里云Elasticsearch服务的用户名
       user => "elastic"
       #对应用户的密码。
       password => "your_password"
       index => "%{[@metadata][_index]}"
       document_type => "%{[@metadata][_type]}"
       document_id => "%{[@metadata][_id]}"
     }
     file_extend {
        path => "/ssd/1/ls-cn-v0h1kzca****/logstash/logs/debug/test"
      }
   }
   ```

3. 查看数据同步结果

   ```
   GET /my_index/_search
   {
     "query": {
       "match_all": {}
     }
   }
   ```

   ![img](https://gitee.com/c_honghui/picture/raw/master/img/20210427120956.png)

## Beat

### 采集ECS服务器日志

前提：

beat仅支持Aliyun Linux、RedHat或CentOS这三种操作系统

在目标ECS实例上安装云助手和Docker服务。

操作：

![img](https://gitee.com/c_honghui/picture/raw/master/img/20210427150608.png)

**启用Kibana Dashboard**：需要开通kibana私网访问

**Filebeat文件目录**：用docker部署beat，会将采集的目录映射到docker，建议与filebeat.yml中的`input.path`保持一致。

如果ECS实例生效成功，则两边数字相等。

![已生效采集器状态](https://gitee.com/c_honghui/picture/raw/master/img/20210427151659.png)

## 通过filebeat采集apache日志

使用Filebeat采集日志数据，并通过阿里云Logstash对采集后的日志数据进行过滤，最终传输到Elasticsearch（简称ES）中进行分析。

1. 配置采集器

![img](https://gitee.com/c_honghui/picture/raw/master/img/20210427153628.png)

2. 配置logstash管道

   ```
   input {
     beats {
         port => 8000
       }
   }
   filter {
     json {
           source => "message"
           remove_field => "@version"
           remove_field => "prospector"
           remove_field => "beat"
           remove_field => "source"
           remove_field => "input"
           remove_field => "offset"
           remove_field => "fields"
           remove_field => "host"
           remove_field => "message"
         }
   
   }
   output {
     elasticsearch {
       hosts => ["http://es-cn-mp91cbxsm00******.elasticsearch.aliyuncs.com:9200"]
       user => "elastic"
       password => "<your_password>"
       index => "<your_index>"
     }
   }
   ```

3. 查看采集数据

   ![img](https://gitee.com/c_honghui/picture/raw/master/img/20210427160646.png)

## Filebeat+Kafka+Logstash+Elasticsearch构建日志分析系统

为了满足大数据实时检索的需求，您可以使用Filebeat采集日志数据，将Kafka作为Filebeat的输出端。Kafka实时接收到Filebeat采集的数据后，以Logstash作为输出端输出。输出到Logstash中的数据在格式或内容上可能不能满足您的需求，此时可以通过Logstash的filter插件过滤数据。最后将满足需求的数据输出到ES中进行分布式检索，并通过Kibana进行数据分析与展示。

1. 配置filebeat

2. 配置logstash管道

   ```shell
   input {
     kafka {
       #消息队列Kafka实例的接入点
       bootstrap_servers => ["172.**.**.92:9092,172.16.**.**:9092,172.16.**.**:9092"]
       #指定为您已创建的Consumer Group的名称。
       group_id => "es-test"
       #指定为您已创建的Topic的名称，需要与Filebeat中配置的Topic名称保持一致。
       topics => ["estest"]
       #设置为json，表示解析json格式的字段，便于在Kibana中分析。
       codec => json
    }
   }
   filter {
   
   }
   output {
     elasticsearch {
       hosts => "http://es-cn-n6w1o1x0w001c****.elasticsearch.aliyuncs.com:9200"
       user =>"elastic"
       password =>"<your_password>"
       index => "kafka‐%{+YYYY.MM.dd}"
    }
   }
   ```

3. 查看日志消费状态

4. 通过kibana过滤日志数据

   创建一个索引模式。

   1. 在左侧导航栏，单击**Management**。
   2. 在Kibana区域，单击**Index Patterns**。
   3. 单击**Create index pattern**。
   4. 输入**Index pattern**（本文使用kafka-*），单击**Next step**。
   5. 选择**Time Filter field name**（本文选择**@timestamp**），单击**Create index pattern**。

   