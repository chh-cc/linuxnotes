# logstash

Logstash主要对日志进行**过滤处理**，也能用来做日志收集。**但日志采集一般不用**logstash

logstash内部主要分为三个组件，输入（input），过滤器（filter），输出（output）

logstash的工作流程：input插件 ---> filter ---> output插件，如无需对数据进行额外处理，filter可省略；

默认logstash用logstash用户启动，日志需要给logstash用户读权限

![image-20210325102556762](https://gitee.com/c_honghui/picture/raw/master/img/20210429113307.png)

Input：**输入**，输出数据可以是Stdin、File、TCP、Redis、Syslog等。 

Filter：**过滤**，将日志格式化。有丰富的过滤插件：Grok正则捕获、Date时间处理、Json编解码、Mutate数据修改等。 

Output：**输出**，输出目标可以是Stdout、File、TCP、Redis、ES等。

## input插件

https://www.elastic.co/guide/en/logstash/current/input-plugins.html

stdin示例

```shell
input {
    #从控制台输入来源
    stdin {
    }
}
filter {
}
output {
    stdout {
        #将日志输出到当前终端上显示
        codec => rubydebug
	}
}

[root@logstash logstash]# bin/logstash -f config/logstash.conf
输入：
123
输出：
{
      "@version" => "1",
          "host" => "logstash",
    "@timestamp" => 2021-10-24T08:51:08.634Z,
       "message" => "123"
}
```

file示例

```shell
input {
    #从文件中来
    file {
        #读取messages文件
    	path =>"/var/log/messages"
    	#排除不想读取的文件
    	exclude =>"1.log"
    	#增加标签
    	tags =>"123"
    	#定义类型
    	type =>"syslog"
    	#读取文件的起始位置，默认是end
    	start_position => beginning
    }
}
filter {
}
output {
    stdout {
        codec => rubydebug
	}
}

[root@logstash logstash]# bin/logstash -f config/logstash.conf
输出：
{
          "path" => "/var/log/messages",
    "@timestamp" => 2021-10-24T08:55:20.108Z,
      "@version" => "1",
          "host" => "logstash",
       "message" => "Oct 24 16:53:32 logstash journal: g_simple_action_set_enabled: assertion 'G_IS_SIMPLE_ACTION (simple)' failed",
          "type" => "syslog",
          "tags" => [
        [0] "123"
    ]
}
```

beats示例

```shell
#从beats输入
input {
    beats {
    	port =>5044
    }
}
filter {
}
output {
    stdout {
        codec => rubydebug
	}
}
```

redis示例

```shell
input {
	redis {
		host => "192.168.10.10"
		port => 6379
		password => "123456"
		db => "0"
		data_type => "list"
		key => "filebeat"
	}
}
```



## codec插件

Logstash处理流程：input->decode->filter->encode->output

codec是基于数据流的过滤器，它可以作为input，output的一部分配置。Codec可以帮助你轻松的分割发送过来已经被序列化的数据。

json编码

```shell
input {
	file {
		path => "/var/log/nginx/access.log_json""
		#输入预定好的json数据
		codec => "json"
	}
}
```

Multiline合并多行数据

```shell
input {
	stdin {
		codec => multiline {
			pattern => "^\["
			negate => true
			what => "previous"
		}
	}
}

输入：
[Aug/08/08 14:54:03] hello world
[Aug/08/09 14:54:04] hello logstash
hello best practice
hello raochenlin
[Aug/08/10 14:54:05] the end

输出：
{
"@timestamp" => "2014-08-09T13:32:03.368Z",
"message" => "[Aug/08/08 14:54:03] hello world\n",
"@version" => "1",
"host" => "raochenlindeMacBook-Air.local"
}
{
"@timestamp" => "2014-08-09T13:32:24.359Z",
"message" => "[Aug/08/09 14:54:04] hello logstash\n\n hello best practice\n\n hello raochenlin\n",
"@version" => "1",
"tags" => [
[0] "multiline"
],
"host" => "raochenlindeMacBook-Air.local"
}
```

## fileter插件

https://www.elastic.co/guide/en/logstash/current/filter-plugins.html

### grok

复杂

匹配非结构化的日志然后结构化输出，使用一整行日志无法分析，需要提取单独的字段，分析哪个IP访问量大，分析Nginx的响应状态码

https://github.com/logstash-plugins/logstash-patterns-core/blob/master/patterns

[Grok Debugger](http://grokdebug.herokuapp.com/)

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

示例：

```shell
input {
    stdin {
    }
}
filter {
	grok {
		match => { 
			"message" => "%{IP:client} %{WORD:method} %{URIPATHPARAM:request} %{NUMBER:bytes} %{NUMBER:duration}" 
		}
	}
}
output {
    stdout {
        codec => rubydebug
	}
}

输入：
223.72.85.86 GET /index.html 15824 0.043
输出：
{
	"@version" => "1",
	"bytes" => "15824",
	"client" => "223.72.85.86",
	"@timestamp" => 2018-05-27T08:30:08.06Z,
	"message" => "223.72.85.86 GET /index.html 15824 0.043",
	"host" => "localhost.localdomain",
	"request" => "/index.html",
	"method" => "GET",
	"duration" => "0.043",
}
```

自定义模式：

```shell
# vi /opt/patterns
ID [0-9A-Z]{10,11}

filter {
	grok {
		patterns_dir =>"/opt/patterns"
		match => {
			"message" => "%{IP:client} %{WORD:method} %{URIPATHPARAM:request} %{NUMBER:bytes} %{NUMBER:duration} %{ID:id}" 
		}
	}
}
```

多模式匹配：

```shell
filter {
	grok {
	patterns_dir =>"/opt/patterns"
	#match多模式匹配
	match => [
		"message", "%{IP:client} %{WORD:method} %{URIPATHPARAM:request} %{NUMBER:bytes} %{NUMBER:duration} %{ID:id}",
		"message", "%{IP:client} %{WORD:method} %{URIPATHPARAM:request} %{NUMBER:bytes} %{NUMBER:duration} %{TAG:tag}"
	]
	}
}
```

## output插件

发送到elasticsearch

```shell
output {
  elasticsearch {
    #elasticsearch地址，多个以','隔开
    hosts => "192.168.71.132:9200"
    #创建的elasticsearch索引名，可以自定义也可以使用下面的默认
    index => "%{[@metadata][beat]}-%{[@metadata][version]}-%{+YYYY.MM.dd}"
  }
}
```

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

