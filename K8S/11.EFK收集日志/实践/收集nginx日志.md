修改nginx日志格式为json

```shell
log_format json '{"timestamp":"$time_local",'
                 '"clientip":"$Real",'
                 '"remote_addr":"$remote_addr",'
                 '"remote_user":"$remote_user",'
                 '"request":"$request",'
                 '"status":"$status",'
                 '"domain":"$host",'
                 '"size":"$body_bytes_sent",'
                 '"response_time":"$request_time",'
                 '"referer":"$http_referer",'
                 '"user_agent":"$http_user_agent",'
                 '"upstream_addr":"$upstream_addr",'
                 '"request_time":"$request_time",'
                 '"upstream_response_time":"$upstream_response_time",'
                 '"upstream_status":"$upstream_status",'
                 '"server_port":"$server_port"'
           '}';
```

filebeat配置

```yaml
filebeat.inputs:
- type: log
  enable: true
  paths:
    - /var/log/nginx/access_json.log
  #json.keys_under_root: true
  #json.overwrite_keys: true
  tags: ["nginx-access"]
  fields:
    app: nginx

- type: log
  enable: true
  paths:
    - /var/log/nginx/error.log
  tags: ["nginx-error"]
  fields:
    app: nginx

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
    #判断日志中的 fields.app 字段是否等于 nginx
    if [fields][app] == "nginx" {
        #使用 json 插件从日志中提取JSON格式的数据，并且移除 message, @version, timestamp, tags 和 prospector 字段
        json {
                source => "message"
                remove_field => ["message"]
                remove_field => ["@version"]
                remove_field => ["timestamp"]
                remove_field => ["tags"]
                remove_field => ["prospector"]
        }
        #使用 geoip 插件从日志中提取 clientip 字段的IP地址，并且查询 GeoLite2-City.mmdb 数据库，将IP地址转换为地理位置信息，并将结果放入 geoip 字段中。此外，还将 geoip.longitude 和 geoip.latitude 组合为 geoip.coordinates 字段。
        geoip {
                source => "clientip"
                target => "geoip"
                database => "/usr/local/elk/logstash/config/GeoLite2-City.mmdb"
                add_field => [ "[geoip][coordinates]", "%{[geoip][longitude]}" ]
                add_field => [ "[geoip][coordinates]", "%{[geoip][latitude]}" ]
        }
        #使用 mutate 插件，将 geoip.coordinates 字段转换为浮点数类型，将 response 和 bytes 字段转换为整数类型，将 type 字段的值替换为 nginx，并删除 message 字段
        mutate {
                convert => [ "[geoip][coordinates]", "float" ]
                convert => [ "response","integer" ]
                convert => [ "bytes","integer" ]
                replace => { "type" => "nginx" }
                remove_field => "message"
        }
    }
}

output {
    if "nginx-access" in [tags] {
        elasticsearch {
          hosts => "10.0.0.101:9200"
          index => "nginx-accesslog-%{+YYYY.MM.dd}"
          #template_overwrite => true
       }
    }
    if "nginx-error" in [tags] {
        elasticsearch {
          hosts => "10.0.0.101:9200"
          index => "nginx-errorlog-%{+YYYY.MM.dd}"
          #template_overwrite => true
       }
    }   
}
```

