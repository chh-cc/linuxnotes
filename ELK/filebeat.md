# filebeat

https://www.elastic.co/guide/en/beats/filebeat/current/index.html

隶属于Beats,**轻量级数据收集引擎**。

不需要使用正则的时候可直接用filebeat发送日志给es，用得比较少；用得比较多filebeat -> Logstash做一些日志的分析提取

## 配置文件

filebeat.yml

```shell
#每一个prospectors，起始于一个破折号”-“
filebeat.prospectors:
#默认log，从日志文件读取每一行。stdin，从标准输入读取
- input_type: log
#日志文件路径列表，可用通配符，不递归
paths:
  - /var/log/*.log
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

## 配置发送日志

filebeat.yml

Filebeat发送日志到ES:

```
filebeat.inputs:
- type: log
  tail_files: true
  backoff: "1s"
  paths:
    - /var/log/nginx/access.log
  fields:
    type: access
  fields_under_root: true    
    
output.elasticsearch:
    hosts: ["192.168.238.90:9200", "192.168.238.92:9200"]
    username: elastic
    password: sjgpwd
    index: "sjgfb-secure-%{+YYYY.MM.dd}"
```

 Filebeat发送到Logstash

```
filebeat.inputs:
- type: log
  tail_files: true
  backoff: "1s"
  paths:
    - /var/log/secure
    
output:
  logstash:
  hosts: ["192.168.238.90:5044"]
```

Filebeat发送日志到redis:

```
filebeat.inputs:
- type: log
  tail_files: true
  backoff: "1s"
  paths:
    - /var/log/nginx/access.log
  fields:
    type: access
  fields_under_root: true
  
output.redis:
  hosts: ["192.168.10.10"]
  password: "123456"
  key: "filebeat"
  db: 0
  timeout: 5
```

