## Prometheus告警处理

### 简介

Prometheus发出告警时分为两部分。首先，Prometheus**按告警规则**(rule_files配置块)向Alertmanager**发送**告警（即告警规则是在Prometheus上定义的）。然后，**由Alertmanager来管理**这些告警，包括去重(Deduplicating)、分组(Grouping)、静音(silencing)、抑制(inhibition)、聚合(aggregation )，最终将面要发出的告警通过电子邮件、webhook等方式将告警通知路由(route)给对应的联系人。

端口：9093

![image-20210301232909110](https://gitee.com/c_honghui/picture/raw/master/img/20210318102955.png)

在Prometheus中一条告警规则主要由以下几部分组成：

- 告警名称：用户需要为告警规则命名，当然对于命名而言，需要能够直接表达出该告警的主要内容
- 告警规则：告警规则实际上主要由PromQL进行定义，其实际意义是当表达式（PromQL）查询结果持续多长时间（During）后出发告警

Alertmanager作为一个独立的组件，负责接收并处理来自Prometheus Server(也可以是其它的客户端程序)的告警信息。Alertmanager可以对这些告警信息进行进一步的处理，比如当接收到大量重复告警时能够消除重复的告警信息，同时对告警信息进行分组并且路由到正确的通知方，Prometheus内置了对邮件，Slack等多种通知方式的支持，同时还支持与Webhook的集成，以支持更多定制化的场景。例如，目前Alertmanager还不支持钉钉，那用户完全可以通过Webhook与钉钉机器人进行集成，从而通过钉钉接收告警信息。同时AlertManager还提供了静默和告警抑制机制来对告警通知行为进行优化。

**Alertmanager特性**

**分组**

就是**将具有相同性质的告警先分类，然后当作单个通知发送出来**。比如A和B两台主机的磁盘(CPU/内存)使用率都在告警，则磁盘的告警就可以合并在一个通知中发送出来。可以想像某个服务部署了100个节点，在一次升版后因为bug，日志中均报同样一类错误，如果不合并这类通知，那就是告警风暴。

**抑制**

就是**某些告警触发后，则抑制（禁止）另一些告警**。比如收到一条告警提示集群故障无法访问，那么在该集群上的所有其他警告应该被抑制。

**静默**

快速根据标签对告警进行静默处理。如果接收到的告警符合静默的配置，Alertmanager则**不会发送告警**通知。

### 部署AlertManager

#### 获取并安装软件包

```shell
export VERSION=0.15.2
curl -LO https://github.com/prometheus/alertmanager/releases/download/v$VERSION/alertmanager-$VERSION.darwin-amd64.tar.gz
tar xvf alertmanager-$VERSION.darwin-amd64.tar.gz
mv alertmanager-$VERSION.darwin-amd64 /usr/local/alertmanager
```

#### 创建alertmanager配置文件

Alertmanager解压后会包含一个默认的alertmanager.yml配置文件，内容如下所示：

```yaml
global:
  resolve_timeout: 5m

route:
  group_by: ['alertname']
  group_wait: 10s
  group_interval: 10s
  repeat_interval: 1h
  receiver: 'web.hook'
receivers:
- name: 'web.hook'
  webhook_configs:
  - url: 'http://127.0.0.1:5001/'
inhibit_rules:
  - source_match:
      severity: 'critical'
    target_match:
      severity: 'warning'
    equal: ['alertname', 'dev', 'instance']
```

配置说明:

```text
global：全局配置，主要配置告警方式，如邮件、webhook等。

route：Prometheus的告警先是到达alertmanager的根路由(route)，alertmanager的根路由不能包含任何匹配项，因为根路由是所有告警的入口点。另外，根路由需要配置一个接收器(receiver)，用来处理那些没有匹配到任何子路由的告警（如果没有配置子路由，则全部由根路由发送告警），即缺省接收器。告警进入到根route后开始遍历子route节点，如果匹配到，则将告警发送到该子route定义的receiver中，然后就停止匹配了。因为在route中continue默认为false，如果continue为true，则告警会继续进行后续子route匹配。如果当前告警仍匹配不到任何的子route，则该告警将从其上一级(匹配)route或者根route发出（按最后匹配到的规则发出邮件）。

group_by：用于分组聚合，对告警通知按标签(label)进行分组，将具有相同标签或相同告警名称(alertname)的告警通知聚合在一个组，然后作为一个通知发送。如果想完全禁用聚合，可以设置为group_by: [...]

group_wait: 当一个新的告警组被创建时，需要等待'group_wait'后才发送初始通知。这样可以确保在发送等待前能聚合更多具有相同标签的告警，最后合并为一个通知发送。

group_interval: 当第一次告警通知发出后，在新的评估周期内又收到了该分组最新的告警，则需等待'group_interval'时间后，开始发送为该组触发的新告警，可以简单理解为，group就相当于一个通道(channel)。

repeat_interval: 告警通知成功发送后，若问题一直未恢复，需再次重复发送的间隔。
```



启动Alertmanager

Alermanager会将数据保存到本地中，默认的存储路径为`data/`。因此，在启动Alertmanager之前需要创建相应的目录：

```shell
./alertmanager
```

`cat >/usr/lib/systemd/system/alertmanager.service   <<EOF`

```shell
[Unit]
Description=alertmanager

[Service]
ExecStart=/usr/local/alertmanager/alertmanager --config.file=/usr/local/alertmanager/alertmanager.yml --storage.path=/usr/local/alertmanager/data --web.listen-address=:9093 --data.retention=120h  

Restart=on-failure

[Install]
WantedBy=multi-user.target

EOF
# systemctl enable alertmanager    
# systemctl restart alertmanager
```



查看运行状态

Alertmanager启动后可以通过9093端口访问，http://192.168.33.10:9093

### 关联Prometheus与Alertmanager

编辑Prometheus配置文件prometheus.yml,并添加以下内容

```yaml
# Alertmanager configuration
alerting:
  alertmanagers:
    - static_configs:
        - targets:
            - monitor01: 9093
  #指定记录规则和告警规则文件
  #默认情况下Prometheus会每分钟对这些告警规则进行计算
  rule_files:
  - "/etc/prometheus/rules/*_rules.yml"
  - "/etc/prometheus/rules/*_alerts.yml"
  ...
  -	job_name: 'alertmanager'
    static_configs:
    - targets: ['localhost:9093']
      labels:
        app: alertmanager
```

重启Prometheus服务，成功后，可以从http://192.168.33.10:9090/config查看alerting配置是否生效。

### 添加记录规则和告警规则

cat node_rules.yml

```yaml
groups:
- name: node_rules
  #interval: 15s
  rules:
  - record: instance:node_cpu_usage  
    expr: 100 - avg(irate(node_cpu_seconds_total{mode="idle"}[1m])) by (nodename) * 100
    labels:
      metric_type: CPU_monitor  

  - record: instance:node_1m_load
    expr: node_load1
    labels:
      metric_type: load1m_monitor

  - record: instance:node_mem_usage
    expr: 100 - (node_memory_MemAvailable_bytes)/(node_memory_MemTotal_bytes) * 100
    labels:
      metric_type: Memory_monitor  

  - record: instance:node_root_partition_predit
    expr: round(predict_linear(node_filesystem_free_bytes{device="rootfs",mountpoint="/"}[2h],12*3600)/(1024*1024*1024), 1)
    labels:
      metric_type: root_partition_monitor
```

cat node_alerts.yml

```yaml
groups:
- name: node_alerts
  rules:
  - alert: cpu_usage_over_threshold
    expr: instance:node_cpu_usage > 80
    for: 1m
    labels:
      severity: warning
    annotations:
      summary: 主机 {{ $labels.nodename }} 的 CPU使用率持续1分钟超出阈值,当前为 {{humanize $value}} %

  - alert: system_1m_load_over_threshold
    expr: instance:node_1m_load > 20
    for: 1m
    labels:
      severity: warning
    annotations:
      summary: 主机 {{ $labels.nodename }} 的 1分负载超出阈值,当前为 {{humanize $value}}

  - alert: mem_usage_over_threshold
    expr: instance:node_mem_usage > 80
    for: 1m
    annotations:
      summary: 主机 {{ $labels.nodename }} 的 内存 使用率持续1分钟超出阈值,当前为 {{humanize $value}} %

  - alert: root_partition_usage_over_threshold
    expr: instance:node_root_partition_predit < 60
    for: 1m
    annotations:
      summary: 主机 {{ $labels.nodename }} 的 根分区 预计在12小时使用将达到 {{humanize $value}}GB,超出当前可用空间，请及时扩容!
```

在告警规则文件中，我们可以将一组相关的规则设置定义在一个group下。在每一个group中我们可以定义多个告警规则(rule)。

一条告警规则主要由以下几部分组成：

- alert：告警规则的**名称**。
- expr：基于PromQL表达式告警**触发条件**，用于计算是否有时间序列满足该条件。
- for：评估等待时间，可选参数。用于表示只有当**触发条件持续一段时间后才发送告警**。在等待期间新产生告警的状态为pending。
- labels：自定义标签，允许用户指定要附加到告警上的一组附加标签。
- annotations：用于指定一组附加信息，比如用于**描述告警详细信息**的文字等，annotations的内容在告警产生时会一同作为参数发送到Alertmanager。

**模板化**

一般来说，在告警规则文件的annotations中使用`summary`描述告警的概要信息，`description`用于描述告警的详细信息。同时Alertmanager的UI也会根据这两个标签值，显示告警信息。为了让告警信息具有更好的可读性，Prometheus支持模板化label和annotations的中标签的值。

通过`$labels.<labelname>`变量可以访问当前告警实例中指定标签的值。$value则可以获取当前PromQL表达式计算的样本值。

```
# To insert a firing element's label values:
{{ $labels.<labelname> }}
# To insert the numeric expression value of the firing element:
{{ $value }}
```

例如，可以通过模板化优化summary以及description的内容的可读性：

```yaml
groups:
- name: example
  rules:

  # Alert for any instance that is unreachable for >5 minutes.
  - alert: InstanceDown
    expr: up == 0
    for: 5m
    labels:
      severity: page
    annotations:
      summary: "Instance {{ $labels.instance }} down"
      description: "{{ $labels.instance }} of job {{ $labels.job }} has been down for more than 5 minutes."

  # Alert for any instance that has a median request latency >1s.
  - alert: APIHighRequestLatency
    expr: api_http_request_latencies_second{quantile="0.5"} > 1
    for: 10m
    annotations:
      summary: "High request latency on {{ $labels.instance }}"
      description: "{{ $labels.instance }} has a median request latency above 1s (current value: {{ $value }}s)"
```

**查看告警状态**

可以通过表达式，查询告警实例：

```
ALERTS{alertname="<alert name>", alertstate="pending|firing", <additional alert labels>}
```

样本值为1表示当前告警处于活动状态（pending或者firing），当告警从活动状态转换为非活动状态时，样本值则为0。

### Alertmanager配置

在Alertmanager中通过路由(Route)来定义告警的处理方式。路由是一个基于标签匹配的树状匹配结构。根据接收到告警的标签匹配相应的处理方式。

在Alertmanager配置中一般会包含以下几个主要部分：

- 全局配置（global）：用于定义一些全局的公共参数，如全局的SMTP配置，Slack配置等内容；
- 模板（templates）：用于定义告警通知时的模板，如HTML模板，邮件模板等；
- 告警路由（route）：根据标签匹配，确定当前告警应该如何处理；
- 接收人（receivers）：接收人是一个抽象的概念，它可以是一个邮箱也可以是微信，Slack或者Webhook等，接收人一般配合告警路由使用；
- 抑制规则（inhibit_rules）：合理设置抑制规则可以减少垃圾告警的产生

其完整配置格式如下：

```yaml
global:
  [ resolve_timeout: <duration> | default = 5m ]
  [ smtp_from: <tmpl_string> ] 
  [ smtp_smarthost: <string> ] 
  [ smtp_hello: <string> | default = "localhost" ]
  [ smtp_auth_username: <string> ]
  [ smtp_auth_password: <secret> ]
  [ smtp_auth_identity: <string> ]
  [ smtp_auth_secret: <secret> ]
  [ smtp_require_tls: <bool> | default = true ]
  [ slack_api_url: <secret> ]
  [ victorops_api_key: <secret> ]
  [ victorops_api_url: <string> | default = "https://alert.victorops.com/integrations/generic/20131114/alert/" ]
  [ pagerduty_url: <string> | default = "https://events.pagerduty.com/v2/enqueue" ]
  [ opsgenie_api_key: <secret> ]
  [ opsgenie_api_url: <string> | default = "https://api.opsgenie.com/" ]
  [ hipchat_api_url: <string> | default = "https://api.hipchat.com/" ]
  [ hipchat_auth_token: <secret> ]
  [ wechat_api_url: <string> | default = "https://qyapi.weixin.qq.com/cgi-bin/" ]
  [ wechat_api_secret: <secret> ]
  [ wechat_api_corp_id: <string> ]
  [ http_config: <http_config> ]

templates:
  [ - <filepath> ... ]

route: <route>

receivers:
  - <receiver> ...

inhibit_rules:
  [ - <inhibit_rule> ... ]
```

在全局配置中需要注意的是`resolve_timeout`，该参数定义了当Alertmanager持续多长时间未接收到告警后标记告警状态为resolved（已解决）。该参数的定义可能会影响到告警恢复通知的接收时间，读者可根据自己的实际场景进行定义，其默认值为5分钟。

### 基于标签的告警处理路由

route中则主要定义了告警的路由匹配规则，以及Alertmanager需要将匹配到的告警发送给哪一个receiver，一个最简单的route定义如下所示：

```yaml
route:
  group_by: ['alertname']
  receiver: 'web.hook'
receivers:
- name: 'web.hook'
  webhook_configs:
  - url: 'http://127.0.0.1:5001/'
```

如上所示：在Alertmanager配置文件中，我们只定义了一个路由，那就意味着所有由Prometheus产生的告警在发送到Alertmanager之后都会通过名为`web.hook`的receiver接收。这里的web.hook定义为一个webhook地址。当然实际场景下，告警处理可不是这么简单的一件事情，对于不同级别的告警，我们可能会有完全不同的处理方式，因此在route中，我们还可以定义更多的子Route，这些Route通过标签匹配告警的处理方式，route的完整定义如下：

```yaml
[ receiver: <string> ]
[ group_by: '[' <labelname>, ... ']' ]
[ continue: <boolean> | default = false ]

match:
  [ <labelname>: <labelvalue>, ... ]

match_re:
  [ <labelname>: <regex>, ... ]

[ group_wait: <duration> | default = 30s ]
[ group_interval: <duration> | default = 5m ]
[ repeat_interval: <duration> | default = 4h ]

routes:
  [ - <route> ... ]
```

**路由匹配**

每一个告警都会从配置文件中顶级的route进入路由树，需要注意的是顶级的route必须匹配所有告警(即不能有任何的匹配设置match和match_re)，每一个路由都可以定义自己的接受人以及匹配规则。默认情况下，告警进入到顶级route后会遍历所有的子节点，直到找到最深的匹配route，并将告警发送到该route定义的receiver中。但如果route中设置**continue**的值为false，那么告警在匹配到第一个子节点之后就直接停止。如果**continue**为true，报警则会继续进行后续子节点的匹配。如果当前告警匹配不到任何的子节点，那该告警将会基于当前路由节点的接收器配置方式进行处理。

其中告警的匹配有两种方式可以选择。一种方式基于字符串验证，通过设置**match**规则判断当前告警中是否存在标签labelname并且其值等于labelvalue。第二种方式则基于正则表达式，通过设置**match_re**验证当前告警标签的值是否满足正则表达式的内容。

如果警报已经成功发送通知, 如果想设置发送告警通知之前要等待时间，则可以通过**repeat_interval**参数进行设置。

**告警分组**

例如，当使用Prometheus监控多个集群以及部署在集群中的应用和数据库服务，并且定义以下的告警处理路由规则来对集群中的异常进行通知。

基于告警中包含的标签，如果满足**group_by**中定义标签名称，那么这些告警将会合并为一个通知发送给接收器。
有的时候为了能够一次性收集和发送更多的相关信息时，可以通过**group_wait**参数设置等待时间。
**group_interval**配置，则用于定义相同的Group之间发送告警通知的时间间隔。

```yaml
route:
  receiver: 'default-receiver'
  group_wait: 30s
  group_interval: 5m
  repeat_interval: 4h
  group_by: [cluster, alertname]
  routes:
  - receiver: 'database-pager'
    group_wait: 10s
    match_re:
      service: mysql|cassandra
  - receiver: 'frontend-pager'
    group_by: [product, environment]
    match:
      team: frontend
```

默认情况下所有的告警都会发送给集群管理员default-receiver，因此在Alertmanager的配置文件的根路由中，对告警信息按照集群以及告警的名称对告警进行分组。
如果告警中包含service标签，并且service为MySQL或者Cassandra,则向database-pager发送告警通知，由于这里没有定义group_by等属性，这些属性的配置信息将从上级路由继承，database-pager将会接收到按cluster和alertname进行分组的告警通知。
如果匹配到告警中包含标签team，并且team的值为frontend，Alertmanager将会按照标签product和environment对告警进行分组。

### 使用Receiver接收告警信息

每一个receiver具有一个全局唯一的名称，并且对应一个或者多个通知方式：

```yaml
name: <string>
email_configs:
  [ - <email_config>, ... ]
hipchat_configs:
  [ - <hipchat_config>, ... ]
pagerduty_configs:
  [ - <pagerduty_config>, ... ]
pushover_configs:
  [ - <pushover_config>, ... ]
slack_configs:
  [ - <slack_config>, ... ]
opsgenie_configs:
  [ - <opsgenie_config>, ... ]
webhook_configs:
  [ - <webhook_config>, ... ]
victorops_configs:
  [ - <victorops_config>, ... ]
```

目前官方内置的第三方通知集成包括：邮件、 即时通讯软件（如Slack、Hipchat）、移动应用消息推送(如Pushover)和自动化运维工具（例如：Pagerduty、Opsgenie、Victorops）。Alertmanager的通知方式中还可以支持Webhook，通过这种方式开发者可以实现更多个性化的扩展支持。

#### 集成邮箱系统

在Alertmanager使用邮箱通知，用户只需要定义好SMTP相关的配置，并且在receiver中定义接收方的邮件地址即可。

在配置文件的global中定义全局的SMTP配置：

```yaml
global:
  [ smtp_from: <tmpl_string> ]
  [ smtp_smarthost: <string> ]
  [ smtp_hello: <string> | default = "localhost" ]
  [ smtp_auth_username: <string> ]
  [ smtp_auth_password: <secret> ]
  [ smtp_auth_identity: <string> ]
  [ smtp_auth_secret: <secret> ]
  [ smtp_require_tls: <bool> | default = true ]
```

为receiver配置email_configs用于定义一组接收告警的邮箱地址即可，如下所示：

```yaml
name: <string>
email_configs:
  [ - <email_config>, ... ]
```

每个email_config中定义相应的接收人邮箱地址，邮件通知模板等信息即可，当然如果当前接收人需要单独的SMTP配置，那直接在email_config中覆盖即可：

```yaml
[ send_resolved: <boolean> | default = false ]
to: <tmpl_string>
[ html: <tmpl_string> | default = '{{ template "email.default.html" . }}' ]
[ headers: { <string>: <tmpl_string>, ... } ]
```

如果当前收件人需要接受告警恢复的通知的话，在email_config中定义`send_resolved`为true即可。

如果所有的邮件配置使用了相同的SMTP配置，则可以直接定义全局的SMTP配置。

这里，以Gmail邮箱为例，我们定义了一个全局的SMTP配置，并且通过route将所有告警信息发送到default-receiver中:

```yaml
global:
  smtp_smarthost: smtp.gmail.com:587
  smtp_from: <smtp mail from>
  smtp_auth_username: <usernae>
  smtp_auth_identity: <username>
  smtp_auth_password: <password>

route:
  group_by: ['alertname']
  receiver: 'default-receiver'

receivers:
  - name: default-receiver
    email_configs:
      - to: <mail to address>
        send_resolved: true
```

#### 集成企业微信

Alertmanager已经内置了对企业微信的支持

[prometheus官网](https://prometheus.io/docs/alerting/configuration/#wechat_config)中给出了企业微信的相关配置说明

```yaml
# Whether or not to notify about resolved alerts.
[ send_resolved: <boolean> | default = false ]

# The API key to use when talking to the WeChat API.
[ api_secret: <secret> | default = global.wechat_api_secret ]

# The WeChat API URL.
[ api_url: <string> | default = global.wechat_api_url ]

# The corp id for authentication.
[ corp_id: <string> | default = global.wechat_api_corp_id ]

# API request data as defined by the WeChat API.
[ message: <tmpl_string> | default = '{{ template "wechat.default.message" . }}' ]
[ agent_id: <string> | default = '{{ template "wechat.default.agent_id" . }}' ]
[ to_user: <string> | default = '{{ template "wechat.default.to_user" . }}' ]
[ to_party: <string> | default = '{{ template "wechat.default.to_party" . }}' ]
[ to_tag: <string> | default = '{{ template "wechat.default.to_tag" . }}' ]
```

企业微信相关概念说明请参考[企业微信API说明](https://work.weixin.qq.com/api/doc#90000/90135/90665)，可以在企业微信的后台中建立多个应用，每个应用对应不同的报警分组，由企业微信来做接收成员的划分。具体配置参考如下：

```yaml
global:
  resolve_timeout: 10m
  wechat_api_url: 'https://qyapi.weixin.qq.com/cgi-bin/'
  wechat_api_secret: '应用的secret，在应用的配置页面可以看到'
  wechat_api_corp_id: '企业id，在企业的配置页面可以看到'
templates:
- '/etc/alertmanager/config/*.tmpl'
route:
  group_by: ['alertname']
  group_wait: 30s
  group_interval: 5m
  repeat_interval: 12h
  routes:
  - receiver: 'wechat'
    continue: true
inhibit_rules:
- source_match:
receivers:
- name: 'wechat'
  wechat_configs:
  - send_resolved: false
    corp_id: '企业id，在企业的配置页面可以看到'
    to_user: '@all'
    to_party: ' PartyID1 | PartyID2 '
    message: '{{ template "wechat.default.message" . }}'
    agent_id: '应用的AgentId，在应用的配置页面可以看到'
    api_secret: '应用的secret，在应用的配置页面可以看到'
```

配置模板示例如下：

```shell
{{ define "wechat.default.message" }}
{{- if gt (len .Alerts.Firing) 0 -}}
{{- range $index, $alert := .Alerts -}}
{{- if eq $index 0 -}}
告警类型: {{ $alert.Labels.alertname }}
告警级别: {{ $alert.Labels.severity }}

=====================
{{- end }}
===告警详情===
告警详情: {{ $alert.Annotations.message }}
故障时间: {{ $alert.StartsAt.Format "2006-01-02 15:04:05" }}
===参考信息===
{{ if gt (len $alert.Labels.instance) 0 -}}故障实例ip: {{ $alert.Labels.instance }};{{- end -}}
{{- if gt (len $alert.Labels.namespace) 0 -}}故障实例所在namespace: {{ $alert.Labels.namespace }};{{- end -}}
{{- if gt (len $alert.Labels.node) 0 -}}故障物理机ip: {{ $alert.Labels.node }};{{- end -}}
{{- if gt (len $alert.Labels.pod_name) 0 -}}故障pod名称: {{ $alert.Labels.pod_name }}{{- end }}
=====================
{{- end }}
{{- end }}

{{- if gt (len .Alerts.Resolved) 0 -}}
{{- range $index, $alert := .Alerts -}}
{{- if eq $index 0 -}}
告警类型: {{ $alert.Labels.alertname }}
告警级别: {{ $alert.Labels.severity }}

=====================
{{- end }}
===告警详情===
告警详情: {{ $alert.Annotations.message }}
故障时间: {{ $alert.StartsAt.Format "2006-01-02 15:04:05" }}
恢复时间: {{ $alert.EndsAt.Format "2006-01-02 15:04:05" }}
===参考信息===
{{ if gt (len $alert.Labels.instance) 0 -}}故障实例ip: {{ $alert.Labels.instance }};{{- end -}}
{{- if gt (len $alert.Labels.namespace) 0 -}}故障实例所在namespace: {{ $alert.Labels.namespace }};{{- end -}}
{{- if gt (len $alert.Labels.node) 0 -}}故障物理机ip: {{ $alert.Labels.node }};{{- end -}}
{{- if gt (len $alert.Labels.pod_name) 0 -}}故障pod名称: {{ $alert.Labels.pod_name }};{{- end }}
=====================
{{- end }}
{{- end }}
{{- end }}
```

### 告警模板详解

默认情况下Alertmanager使用了系统自带的默认通知模板，模板源码可以从https://github.com/prometheus/alertmanager/blob/master/template/default.tmpl获得。Alertmanager的通知模板基于[Go的模板系统](http://golang.org/pkg/text/template)。Alertmanager也支持用户定义和使用自己的模板，一般来说有两种方式可以选择。

第一种，基于模板字符串。用户可以直接在Alertmanager的配置文件中使用模板字符串，例如:

```yaml
receivers:
- name: 'slack-notifications'
  slack_configs:
  - channel: '#alerts'
    text: 'https://internal.myorg.net/wiki/alerts/{{ .GroupLabels.app }}/{{ .GroupLabels.alertname }}'
```

第二种方式，自定义可复用的模板文件。例如，可以创建自定义模板文件custom-template.tmpl，如下所示：

```
{{ define "slack.myorg.text" }}https://internal.myorg.net/wiki/alerts/{{ .GroupLabels.app }}/{{ .GroupLabels.alertname }}{{ end}}
```

通过在Alertmanager的全局设置中定义templates配置来指定自定义模板的访问路径:

```
# Files from which custom notification template definitions are read.
# The last component may use a wildcard matcher, e.g. 'templates/*.tmpl'.
templates:
  [ - <filepath> ... ]
```

在设置了自定义模板的访问路径后，用户则可以直接在配置中使用该模板：

```yaml
receivers:
- name: 'slack-notifications'
  slack_configs:
  - channel: '#alerts'
    text: '{{ template "slack.myorg.text" . }}'

templates:
- '/etc/alertmanager/templates/myorg.tmpl'
```

### 屏蔽告警通知

**抑制机制**

避免当某种问题告警产生之后用户接收到大量由此问题导致的一系列的其它告警通知。

使用inhibit_rules定义一组告警的抑制规则：

```yaml
inhibit_rules:
  [ - <inhibit_rule> ... ]
```

每一条抑制规则的具体配置如下：

```yaml
target_match:
  [ <labelname>: <labelvalue>, ... ]
target_match_re:
  [ <labelname>: <regex>, ... ]

source_match:
  [ <labelname>: <labelvalue>, ... ]
source_match_re:
  [ <labelname>: <regex>, ... ]

[ equal: '[' <labelname>, ... ']' ]
```

当已经发送的告警通知匹配到target_match和target_match_re规则，当有新的告警规则如果满足source_match或者定义的匹配规则，并且已发送的告警与新产生的告警中equal定义的标签完全相同，则启动抑制机制，新的告警不会发送。

**临时静默**

直接通过Alertmanager的UI临时屏蔽特定的告警通知。通过定义标签的匹配规则(字符串或者正则表达式)，如果新的告警通知满足静默规则的设置，则停止向receiver发送通知。

进入Alertmanager UI，点击"New Silence"

用户可以通过该UI定义新的静默规则的开始时间以及持续时间，通过Matchers部分可以设置多条匹配规则(字符串匹配或者正则匹配)。填写当前静默规则的创建者以及创建原因后，点击"Create"按钮即可。

通过"Preview Alerts"可以查看预览当前匹配规则匹配到的告警信息。静默规则创建成功后，Alertmanager会开始加载该规则并且设置状态为Pending,当规则生效后则进行到Active状态。

## 

