# ELK

开发人员排查问题，经常上线服务器去查询项目日志

服务器越来越多，项目越来越多，日志类型越来越多

## 简介

https://www.elastic.co/cn/

https://www.elastic.co/subscriptions

Elasticsearch：搜索、分析和存储数据，分布式数据库。

Logstash：对**日志进行收集、转换**，并将其存储到数据库中

kibana ：数据可视化

Beats ：轻量型采集器的平台，从边缘机器向 Logstash 和 Elasticsearch 发送数据

Filebeat：轻量型日志采集器。

ELK6：默认安装它是开放访问的，需要xpack之类的才能启用认证
ELK7 :  默认开启安全认证功能

## 架构

### Elasticsearch + Logstash + Kibana

![image-20210221174418582](https://gitee.com/c_honghui/picture/raw/master/img/20210221174418.png)

- 这是一种最`简单`的架构。这种架构，通过logstash收集日志，Elasticsearch分析日志，然后在Kibana(web界面)中展示。这种架构虽然是官网介绍里的方式，但是往往在生产中很少使用。

### Elasticsearch + Logstash + filebeat + Kibana

- 与上一种架构相比，这种架构增加了一个`filebeat`模块。filebeat是一个轻量的日志收集代理，用来部署在客户端，优势是消耗非常少的资源(较logstash)， 所以生产中，往往会采取这种架构方式，但是这种架构有一个缺点，当logstash出现故障， 会造成日志的丢失。

![image-20210221174432189](https://gitee.com/c_honghui/picture/raw/master/img/20210221174432.png)

### Elasticsearch + Logstash + filebeat + redis（kafka）

![image-20210221174600614](https://gitee.com/c_honghui/picture/raw/master/img/20210221174600.png)

- 这种架构是上面那个架构的完善版，通过`增加中间件`，来避免数据的丢失。**当Logstash出现故障，日志还是存在中间件中，当Logstash再次启动，则会读取中间件中积压的日志。**目前我司使用的就是这种架构，我个人也比较推荐这种方式。
