收集SpringBoot项目的日志输出，有几个地方需要注意：

- 项目日志输出格式化
- Filebeat将多行日志汇集成一行
- Logstash将项目日志格式化为json



这里的日志格式为：

2023-03-30 15:44:05,585 INFO  [  ] 。。。。

错误日志格式为：

Caused by:开头或者[2023-03-30 14:45:01] [http-nio-9501-exec-6] ERROR开头

filebeat配置

```yaml
filebeat.inputs:
- type: log
  enabled: true
  paths:
    - /xxx/config-server/*-error.*.log
  #json.keys_under_root: true
  #json.overwrite_keys: true
  multiline.pattern: '^\[|^Caused by:'
  multiline.negate: true
  multiline.match: after
  tags: ["config"]
  fields:
    app: config
    logsource: ip地址
    ...
    
- type: log
  enabled: true
  paths:
    - /xxx/config-server/config-server.log
  #json.keys_under_root: true
  #json.overwrite_keys: true
  multiline.pattern: '^\d{4}-\d{2}-\d{2}'
  multiline.negate: true
  multiline.match: after
  tags: ["config"]
  fields:
    app: config
    logsource: ip地址
    ...
    
- type: log
  enabled: true
  paths:
    - /xxx/eureka-server/*.log
  #json.keys_under_root: true
  #json.overwrite_keys: true
  tags: ["eureka"]
  fields:
    app: eureka
    
- type: log
  enabled: true
  paths:
    - /xxx/gateway-server/*.log
  #json.keys_under_root: true
  #json.overwrite_keys: true
  tags: ["gateway"]
  fields:
    app: gateway
    
...
    
output.logstash:
  hosts: ["10.10.0.100:5044"]
```

logstash配置

```yaml
input {
  beats { port => 5044 }
}

filter {
    date { match => [ "timestamp","yyyy-MM-dd HH:mm:ss,SSS" ] } #使用 date 插件解析 timestamp 字段为日期时间格式
    if [fields][app] == "config" {
        
    }
    if [fields][app] == "eureka" {
        
    }
    if [fields][app] == "gateway" {
        
    }
}

output {
    if "config" in [tags] {
        elasticsearch {
          hosts => "10.0.0.101:9200"
          index => "configlog-%{+YYYY.MM.dd}"
          #template_overwrite => true
       }
    }
    if "eureka" in [tags] {
        elasticsearch {
          hosts => "10.0.0.101:9200"
          index => "eurekalog-%{+YYYY.MM.dd}"
          #template_overwrite => true
       }
    }
    if "gateway" in [tags] {
        elasticsearch {
          hosts => "10.0.0.101:9200"
          index => "gatewaylog-%{+YYYY.MM.dd}"
          #template_overwrite => true
       }
    }
}
```

