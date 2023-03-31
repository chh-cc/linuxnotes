官方说明：https://www.elastic.co/guide/en/beats/filebeat/current/index.html



## 重要配置参数

```yaml
# ============================== Filebeat inputs ===============================
filebeat.inputs: 
- type: log #默认log，从日志文件读取每一行。包括log(具体路径的日志),stdin(键盘输入),redis,udp,docker,tcp,syslog,可以同时配置多个(包括相同类型的)
  enabled: true #启用手工配置filebeat，而不是采用模块方式配置filebeat
  paths:
    - "/root/applog/boe-wms/*"
  include_lines: [‘ERR’, ‘WARN’] #只采集某个字段，include_lines执行完毕之后会执行exclude_lines
  exclude_lines: [“^DBG”] #在输入中排除符合正则表达式列表的那些行
  exclude_files: ['.gz$'] # 忽略掉符合正则表达式列表的文件
  fields:
    log_type: "wms" #这个得意思就是会在es中多添加一个字段，格式为 "filelds":{"log_type":"wms"}，方便后续对日志进行分组统计。
  tags: ["wms"] #添加额外的tag标签，使用tag来区分不同的日志
  multiline: #适用于日志中每一条日志占据多行的情况，比如java堆栈
    pattern: ^[0-9]{4} #如果是以4个任意数字开头的，则这一行是一条日志的开头行
    negate: true
    match: after #取值为after或before。该值与上面的pattern与negate值配合使用
    timeout: 3s
  # ----------------------------------------------------------------------------------------------------

  #|multiline.negate|multiline.match|                      结论                      |

  # ----------------------------------------------------------------------------------------------------
                                                                              
  #|    true        |    before    |表示匹配行是结尾,和前面不匹配的组成一行完整的日志|

  # ----------------------------------------------------------------------------------------------------

  #|    true        |    after    |表示匹配行是开头,和后面不匹配的组成一行完整的日志|

  # ----------------------------------------------------------------------------------------------------

  #|    false      |    before    |匹配模式的连续行被附加到下一个不匹配的 |

  # ----------------------------------------------------------------------------------------------------

  #|    false      |    after    |匹配模式的连续行被附加到不匹配的前一行 |

  # ----------------------------------------------------------------------------------------------------
  #multiline.flush_pattern # 表示符合该正则表达式的，将从内存刷入硬盘。
  #multiline.max_lines: 500 # 表示如果多行信息的行数超过该数字，则多余的都会被丢弃。默认值为500行
  max_bytes: 10485760 #单文件最大收集的字节数，单文件超过此字节数后的字节将被丢弃，默认10MB，需要增大，保持与日志输出配置的单文件最大值一致即可
  
# ---------------------------- Output ----------------------------
# -------------------------- Elasticsearch output ------------------------------
output.elasticsearch:
  hosts: ["10.251.74.113:9200"]
  index: "filebeat-%{[beat.version]}-%{+yyyy.MM.dd}" #ES索引
  protocol: "https" #协议
  username: "elastic" # ES用户名
  password: "changeme" # ES密码
 
# ----------------------------- Logstash output --------------------------------
output.logstash:
  hosts: ["10.10.0.100:5044"]
  #ssl.certificate_authorities: ["/etc/pki/root/ca.pem"]
  #ssl.certificate: "/etc/pki/client/cert.pem"
  #ssl.key: "/etc/pki/client/cert.key"
  
#================================ Procesors =====================================
processors:

  #主机相关 信息

  - add_host_metadata: ~

# 云服务器的元数据信息,包括阿里云ECS 腾讯云QCloud AWS的EC2的相关信息 

  - add_cloud_metadata: ~

  #k8s元数据采集

  #- add_kubernetes_metadata: ~

  # docker元数据采集

  #- add_docker_metadata: ~

  # 执行进程的相关数据

  #- - add_process_metadata: ~
#================================ Logging =====================================

# Sets log level. The default log level is info.

# Available log levels are: error, warning, info, debug

#logging.level: debug

# At debug level, you can selectively enable logging only for some components.

# To enable all selectors use ["*"]. Examples of other selectors are "beat",

# "publish", "service".

#logging.selectors: ["*"]

#============================== Xpack Monitoring ===============================

# filebeat can export internal metrics to a central Elasticsearch monitoring

# cluster.  This requires xpack monitoring to be enabled in Elasticsearch.  The

# reporting is disabled by default.

# Set to true to enable the monitoring reporter.

#xpack.monitoring.enabled: false

# Uncomment to send the metrics to Elasticsearch. Most settings from the

# Elasticsearch output are accepted here as well. Any setting that is not set is

# automatically inherited from the Elasticsearch output configuration, so if you

# have the Elasticsearch output configured, you can simply uncomment the

# following line.

#xpack.monitoring.elasticsearch:
```

## 合并多行日志

<img src="assets/webp" alt="img" style="zoom:67%;" />

**例1:**

Java堆栈跟踪由多行组成：

```java
Exception in thread "main" java.lang.NullPointerException
        at com.example.myproject.Book.getTitle(Book.java:16)
        at com.example.myproject.Author.getBookTitles(Author.java:25)
        at com.example.myproject.Bootstrap.main(Bootstrap.java:14)
```

将这些行整合到Filebeat中的单个事件中:

```yaml
multiline.pattern: '^[[:space:]]'
multiline.negate: false
multiline.match: after
```

**例2:**

Java堆栈跟踪日志：

```java
Exception in thread "main" java.lang.IllegalStateException: A book has a null property
       at com.example.myproject.Author.getBookIds(Author.java:38)
       at com.example.myproject.Bootstrap.main(Bootstrap.java:14)
Caused by: java.lang.NullPointerException
       at com.example.myproject.Book.getId(Book.java:22)
       at com.example.myproject.Author.getBookIds(Author.java:35)
       ... 1 more
```

将这些行整合到Filebeat中的单个事件中：

```yaml
multiline.pattern: '^[[:space:]]+(at|\.{3})\b|^Caused by:'
multiline.negate: false
multiline.match: after
```

**例3：**

以时间戳开头

```java
[2019-08-24 11:49:14,389][INFO ][env                      ] [Letha] using [1] data paths, mounts [[/
(/dev/disk1)]], net usable_space [34.5gb], net total_space [118.9gb], types [hfs]
```

将这些行整合到Filebeat中的单个事件中

```yaml
multiline.pattern: '^\[[0-9]{4}-[0-9]{2}-[0-9]{2}'
multiline.negate: true
multiline.match: after
```

