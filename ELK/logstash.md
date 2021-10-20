# logstash

Logstash主要对日志进行**过滤处理**，也能用来做日志收集。**但日志采集一般不用**logstash

logstash内部主要分为三个组件，输入（input），过滤器（filter），输出（output）

logstash的工作流程：input插件 ---> filter ---> output插件，如无需对数据进行额外处理，filter可省略；

默认logstash用logstash用户启动，日志需要给logstash用户读权限

![image-20210325102556762](https://gitee.com/c_honghui/picture/raw/master/img/20210429113307.png)

Input：输入，输出数据可以是Stdin、File、TCP、Redis、Syslog等。 

Filter：过滤，将日志格式化。有丰富的过滤插件：Grok正则捕获、Date时间处理、Json编解码、Mutate数据修改等。 

Output：输出，输出目标可以是Stdout、File、TCP、Redis、ES等。

https://www.elastic.co/guide/en/logstash/current/input-plugins.html

## input插件



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

```
Logstash最简单配置/etc/logstash/conf.d/logstash.conf
input{
  stdin{}
}
output{
  stdout{
    codec=>rubydebug
  }
}
Logstash的启动和测试
/usr/share/logstash/bin/logstash -f /etc/logstash/conf.d/logstash.conf
输入字符，查看输出
```

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

#从本地收集日志输出到Elasticsearch
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
  stdout{
    codec => rubydebug
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

## logstash正则提取日志

为什么需要提取？使用一整行日志无法分析，需要提取单独的字段
 分析哪个IP访问量大
 分析Nginx的响应状态码

Grok提取利器，需要掌握正则表达式。借助Kibana的Grok工具验证提取
 自写正则提取(建议)
 内置规则提取（简化）：/usr/share/logstash/vendor/bundle/jruby/2.5.0/gems/logstash-patterns-core-4.1.2/patterns/grok-patterns

```
Grok自写正则提取语法：(?<字段名>自写正则表达式)
(?<remote_addr>\d+\.\d+\.\d+\.\d+)

内置正则提取语法：%{内置正则表达式:字段名}
%{IP:remote_addr} - (%{WORD:remote_user}|-) \[%{HTTPDATE:time_local}\] "%{WORD:method} %{NOTSPACE:request} HTTP/%{NUMBER}" %{NUMBER:status} %{NUMBER:body_bytes_sent} %{QS} %{QS:http_user_agent}

混合语法提取
(?<remote_addr>\d+\.\d+\.\d+\.\d+) - (%{WORD:remote_user}|-) \[%{HTTPDATE:time_local}\]
```

```
普通正则表达式符号
 . 表示任意一个字符，* 表示前面一个字符出现0次或者多次
 [abc]表示中括号内任意一个字符，[^abc]表示非中括号内的字符
 [0-9]表示数字，[a-z]表示小写字母，[A-Z]表示大写字母，[a-zA-Z]表示所有字母，[a-zA-Z0-9]表示所有字母+数字
 [^0-9]表示非数字
 ^xx表示以xx开头，xx$表示以xx结尾
 \s表示空白字符，\S表示非空白字符，\d表示数字
```

```
扩展正则表达式，在普通正则基础上再进行扩展
 ?表示前面字符出现0或者1次，+前面字符出现1或者多次
 {a}表示前面字符匹配a次，{a,b}表示前面字符匹配a到b次
 {,b}表示前面字符匹配0次到b次，{a,}前面字符匹配a或a+次
 string1|string2表示匹配string1或者string2
```

Logstash正则提取Nginx:

```
filter {
  grok {
    match => {
      "message" => '%{IP:remote_addr} - (%{WORD:remote_user}|-) \[%{HTTPDATE:time_local}\] "%{WORD:method} %{NOTSPACE:request} HTTP/%{NUMBER}" %{NUMBER:status} %{NUMBER:body_bytes_sent} %{QS} %{QS:http_user_agent}'
    }
    remove_field => ["message"]
  }
}
```

Logstash字段特殊处理-替换或转类型:

```
http_user_agent包含双引号，需要去除
filter {
  grok {
    match => {
   "message" => '%{IP:remote_addr} - (%{WORD:remote_user}|-) \[%{HTTPDATE:time_local}\] "%{WORD:method} %{NOTSPACE:request} HTTP/%{NUMBER}" %{NUMBER:status} %{NUMBER:body_bytes_sent} %{QS} %{QS:http_user_agent}'
    }
    remove_field => ["message"]
  }
  mutate {
    gsub => [ "http_user_agent",'"',"" ]
  }
}
```

Logstash字符串转整形:

```
mutate{
  gsub => [ "http_user_agent",'"',"" ]
  convert => { "status" => "integer" }
  convert => { "body_bytes_sent" => "integer" }
}
```

Logstash分析Linux系统日志:

```
日志格式
2020-08-03 18:47:34 sjg1 sshd[1522]: Accepted password for root from 58.101.14.103 port 49774 ssh2
%{TIMESTAMP_ISO8601:timestamp} %{NOTSPACE} %{NOTSPACE:procinfo}: (?<secinfo>.*)
 
只读权限添加
chmod +r secure
 
提取secure日志，messages等其它日志提取原理类似
input {
  file {
    path => "/var/log/secure"
  }
}
filter {
  grok {
    match => {
      "message" => '%{TIMESTAMP_ISO8601:timestamp} %{NOTSPACE} %{NOTSPACE:procinfo}: (?<secinfo>.*)'
    }
    remove_field => ["message"]
  }
  date {
    match => ["timestamp", "yyyy-MM-dd HH:mm:ss"]
    target => "@timestamp"
  }
  mutate {
    remove_field => ["timestamp"]
  }
}
output {
  elasticsearch {
    hosts => ["http://192.168.238.90:9200", "http://192.168.238.92:9200"]
    user => "elastic"
    password => "sjgpwd"
    index => "sjgsecure-%{+YYYY.MM.dd}"
  }
}
```

