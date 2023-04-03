日志格式有：

```shell
2023-04-03 11:14:56,601 INFO  [  ] c.x.e.p.s.i.XazrXcxProductServiceImpl:555 - ....
....
```

```shell
[2023-04-02 22:06:26] ...
...
        at ...
        at ...
        
#或者
[2023-04-02 22:06:26] ...
...
        at ...
        at ...
Caused by: ...
        at ...
        at ...
        ... 26 common frames omitted
```

```shell
[2023-04-03 11:31:33] [Thread-22500] INFO ...
...
```



filebeat配置

```yaml
filebeat.inputs:
- type: log
  enabled: true
  paths:
    - /xxx/picture-service-error.log
  #json.keys_under_root: true
  #json.overwrite_keys: true
  multiline.pattern: '^[[:space:]]+(at|\.{3})\b|^Caused by:'
  multiline.negate: false
  multiline.match: after
  tags: ["picture-error"]
  fields:
    app: picture
    logsource: ip地址
    ...
    
- type: log
  enabled: true
  paths:
    - /xxx/picture-service.log
  #json.keys_under_root: true
  #json.overwrite_keys: true
  multiline.pattern: '^\d{4}-\d{2}-\d{2}'
  multiline.negate: true
  multiline.match: after
  tags: ["picture-access"]
  fields:
    app: picture
    logsource: ip地址
    ...
    
- type: log
  enabled: true
  paths:
    - /xxx/user-center.log
  #json.keys_under_root: true
  #json.overwrite_keys: true
  multiline.pattern: '^\['
  multiline.negate: true
  multiline.match: after
  tags: ["user-center-access"]
  fields:
    app: user-center
    
- type: log
  enabled: true
  paths:
    - /xxx/user-center-error.log
  #json.keys_under_root: true
  #json.overwrite_keys: true
  multiline.pattern: '^[[:space:]]+(at|\.{3})\b|^Caused by:'
  multiline.negate: false
  multiline.match: after
  tags: ["uesr-center-error"]
  fields:
    app: user-center
    
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
    if [fields][app] == "picture" {
        
    }
    if [fields][app] == "user-center" {
        
    }
}

output {
    if "picture-access" in [tags] {
        elasticsearch {
          hosts => "10.0.0.101:9200"
          index => "picturelog-%{+YYYY.MM.dd}"
          #template_overwrite => true
       }
    }
    if "picture-error" in [tags] {
        elasticsearch {
          hosts => "10.0.0.101:9200"
          index => "picture-errorlog-%{+YYYY.MM.dd}"
          #template_overwrite => true
       }
    }
    if "user-center-access" in [tags] {
        elasticsearch {
          hosts => "10.0.0.101:9200"
          index => "user-centerlog-%{+YYYY.MM.dd}"
          #template_overwrite => true
       }
    }
    if "user-center-error" in [tags] {
        elasticsearch {
          hosts => "10.0.0.101:9200"
          index => "user-center-errorlog-%{+YYYY.MM.dd}"
          #template_overwrite => true
       }
    }
}
```

