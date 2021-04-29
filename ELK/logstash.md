# logstash

主要是用来日志的搜集、分析、过滤日志的工具。用于管理日志 和事件的工具，你可以用它去收集日志、转换日志、解析日志并将他们 作为数据提供给其它模块调用，例如搜索、存储等。

logstash内部主要分为三个组件，输入（input），过滤器（filter），输出（output）

logstash的工作流程：input插件 ---> filter ---> output插件，如无需对数据进行额外处理，filter可省略；

![image-20210325102556762](https://gitee.com/c_honghui/picture/raw/master/img/20210429113307.png)

## input



## filter



## output



## 配置文件

vim logstash.yml

```yaml
# If using queue.type: persisted, the interval in milliseconds when a checkpoint is forced on the head page
# Default is 1000, 0 for no periodic checkpoint.
#启用持久队列(queue.type: persisted)后，执行检查点的时间间隔，单位ms，默认1000ms
queue.checkpoint.interval: 1000
# ------------ Dead-Letter Queue Settings --------------
#是否启用插件支持的DLQ功能的标志，默认false
dead_letter_queue.enable: false
#dead_letter_queue.enable为true时，每个死信队列的最大大小
#若死信队列的大小超出该值，则被删除，默认1024mb
dead_letter_queue.max_bytes: 1024mb
#死信队列存储路径，默认path.data/dead_letter_queue
path.dead_letter_queue:
# ------------ Debugging Settings --------------
#日志输出级别，选项：fatal,error,warn,info,debug,trace,默认info
log.level: info
#日志格式，选项:json,plain,默认plain
log.format:
#日志路径，默认LOGSTASH_HOME/logs
path.logs:
# ------------ Other Settings --------------
#插件存储路径
path.plugins: []
#是否启用每个管道在不同日志文件中的日志分隔
#默认false
pipeline.separate_logs: false
```



vim logstash.conf
该文件定义了**logstash从哪里获取输入，然后输出到哪里**

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
    hosts => "192.168.71.132:9200"
    #创建的elasticsearch索引名，可以自定义也可以使用下面的默认
    index => "%{[@metadata][beat]}-%{[@metadata][version]}-%{+YYYY.MM.dd}"
  }
}

#从ELK收集日志输出到Elasticsearch
input{
  #读取文件
  file{
    path => "/var/log/*.log"
    type => "system"
  }
}

output{
  elasticsearch{
    hosts=>["127.0.0.1:9200"]
    index => "es-message-%{+YYYY.MM.dd}" #要输入的elasticsearch的索引，没有会自建
  }
  #并打印到控制台
  stdout{codec => rubydebug}
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

###  