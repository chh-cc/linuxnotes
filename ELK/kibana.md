# kibana

是一个优秀的**前端日志展示框架**，它可以非常详细的将日志转化 为各种图表,它能够搜索、展示存储Elasticsearch 中索引数据。 

**监听端口：5601**

## 配置文件kibana.yml

```shell
#####----------kibana服务相关----------#####
#提供服务的端口，监听端口
server.port: 5601
#主机地址，可以是ip,主机名
server.host: 0.0.0.0
#在代理后面运行，则可以指定安装Kibana的路径
#使用server.rewriteBasePath设置告诉Kibana是否应删除basePath
#接收到的请求，并在启动时防止过时警告
#此设置不能以斜杠结尾
server.basePath: ""
#指定Kibana是否应重写以server.basePath为前缀的请求，或者要求它们由反向代理重写，默认false
server.rewriteBasePath: false
#传入服务器请求的最大有效负载大小,以字节为单位，默认1048576
server.maxPayloadBytes: 1048576
#该kibana服务的名称，默认your-hostname
server.name: "your-hostname"
#服务的pid文件路径，默认/var/run/kibana.pid
pid.file: /var/run/kibana.pid
#####----------elasticsearch相关----------#####
#kibana访问es服务器的URL,就可以有多个，以逗号","隔开
elasticsearch.hosts: ["http://ip1:9200","http://ip2:9200"]
#当此值为true时，Kibana使用server.host设定的主机名
#当此值为false时，Kibana使用连接Kibana实例的主机的主机名
#默认ture
elasticsearch.preserveHost: true
#Kibana使用Elasticsearch中的索引来存储已保存的搜索，可视化和仪表板
#如果索引尚不存在，Kibana会创建一个新索引
#默认.kibana
kibana.index: ".kibana"
#加载的默认应用程序
#默认home
kibana.defaultAppId: "home"
#kibana访问Elasticsearch的账号与密码(如果ElasticSearch设置了的话)
elasticsearch.username: "kibana_system"
elasticsearch.password: "pass"
#从Kibana服务器到浏览器的传出请求是否启用SSL
#设置为true时，需要server.ssl.certificate和server.ssl.key
server.ssl.enabled: true
server.ssl.certificate: /path/to/your/server.crt
server.ssl.key: /path/to/your/server.key
#从Kibana到Elasticsearch启用SSL后，ssl.certificate和ssl.key的位置
elasticsearch.ssl.certificate: /path/to/your/client.crt
elasticsearch.ssl.key: /path/to/your/client.key
#PEM文件的路径列表
elasticsearch.ssl.certificateAuthorities: [ "/path/to/your/CA.pem" ]
#控制Elasticsearch提供的证书验证
#有效值为none，certificate和full
elasticsearch.ssl.verificationMode: full
#Elasticsearch服务器响应ping的时间，单位ms
elasticsearch.pingTimeout: 1500
#Elasticsearch 的响应的时间，单位ms
elasticsearch.requestTimeout: 30000
#Kibana客户端发送到Elasticsearch的标头列表
#如不发送客户端标头，请将此值设置为空
elasticsearch.requestHeadersWhitelist: []
#Kibana客户端发往Elasticsearch的标题名称和值
elasticsearch.customHeaders: {}
#Elasticsearch等待分片响应的时间
elasticsearch.shardTimeout: 30000
#Kibana刚启动时等待Elasticsearch的时间，单位ms，然后重试
elasticsearch.startupTimeout: 5000
#记录发送到Elasticsearch的查询
elasticsearch.logQueries: false
#####----------日志相关----------#####
#kibana日志文件存储路径，默认stdout
logging.dest: stdout
#此值为true时，禁止所有日志记录输出
#默认false
logging.silent: false
#此值为true时，禁止除错误消息之外的所有日志记录输出
#默认false
logging.quiet: false
#此值为true时，记录所有事件，包括系统使用信息和所有请求
#默认false
logging.verbose: false
#####----------其他----------#####
#系统和进程取样间隔，单位ms，最小值100ms
#默认5000ms
ops.interval: 5000
#kibana web语言
#默认en
i18n.locale: "en"
```

## 访问kibana

检查kibana状态：http://localhost:5601/status

![img](https://gitee.com/c_honghui/picture/raw/master/img/20210429113408.png)

 在你开始用Kibana之前，你需要告诉Kibana你想探索哪个Elasticsearch索引。第一次访问Kibana是，系统会提示你定义一个索引模式以匹配一个或多个索引的名字。

1. 访问http://127.0.0.1:5601 查看kibana

2. 指定一个索引模式来匹配一个或多个你的Elasticsearch索引。当你指定了你的索引模式以后，任何匹配到的索引都将被展示出来。

   （画外音：*匹配0个或多个字符； 指定索引默认是为了匹配索引，确切的说是匹配索引名字）

   ![image-20211024230130390](https://gitee.com/c_honghui/picture/raw/master/img/20211024230130.png)

   ![image-20211024225501714](https://gitee.com/c_honghui/picture/raw/master/img/20211024225508.png)

3. 点击“**Next Step**”以选择你想要用来执行基于时间比较的包含timestamp字段的索引。如果你的索引没有基于时间的数据，那么选择“**I don’t want to use the Time Filter**”选项。

   ![image-20211024230213443](https://gitee.com/c_honghui/picture/raw/master/img/20211024230213.png)

4. 点击“**Create index pattern**”按钮可以再添加索引。第一个索引模式自动配置为默认的索引默认，以后当你有多个索引模式的时候，你就可以选择将哪一个设为默认。（提示：Management > Index Patterns）

   ![image-20211024230439596](https://gitee.com/c_honghui/picture/raw/master/img/20211024230439.png)

