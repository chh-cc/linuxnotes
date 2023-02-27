# alertmanager

Alertmanager处理从Prometheus服务器发来的警报。然后，Alertmanager对这些告警进行去重(Deduplicating)、分组(Grouping)、静音(silencing)、抑制(inhibition)、聚合(aggregation )，然后路由到不同的接收器，如电子邮件、短信等给对应的联系人。

端口：9093

## 部署AlertManager

### 获取并安装软件包

安装包下载
地址1： https://prometheus.io/download/
地址2： https://github.com/prometheus/alertmanager/releases  

```shell
export VERSION=0.15.2
curl -LO https://github.com/prometheus/alertmanager/releases/download/v$VERSION/alertmanager-$VERSION.darwin-amd64.tar.gz
tar xvf alertmanager-$VERSION.darwin-amd64.tar.gz
mv alertmanager-$VERSION.darwin-amd64 /usr/local/alertmanager
```

### 启动alertmanager

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

## Alertmanager配置

在Alertmanager中通过路由(Route)来定义告警的处理方式。路由是一个基于标签匹配的树状匹配结构。根据接收到告警的标签匹配相应的处理方式。

在Alertmanager配置中一般会包含以下几个主要部分：

- 全局配置（global）：用于定义一些全局的公共参数，如全局的SMTP配置，Slack配置等内容；
- 模板（templates）：用于定义通知模板；
- 告警路由（route）：根据标签匹配，确定当前告警应该如何处理；
- 接收人（receivers）：接收人是一个抽象的概念，它可以是一个邮箱也可以是微信，Slack或者Webhook等，接收人一般配合告警路由使用；
- 抑制规则（inhibit_rules）：合理设置抑制规则可以减少垃圾告警的产生

其完整配置格式如下：

```yaml
global:
  # 当Alertmanager持续多长时间未接收到告警后标记告警状态为resolved（已解决）
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

# - 从中读取自定义通知模板定义的文件,最后一个组件可以使用通配符匹配器,例如“templates/*.tmpl”。 
templates:
  [ - <filepath> ... ]

# - 路由块定义路由树中的节点及其子节点。如果未设置则从其父节点继承其可选配置参数。 
route: <route>

# 通知接收者列表
receivers:
  - <receiver> ...

# - 抑制规则列表: 当存在与另一组匹配器匹配的警报（源）时，禁止规则会使与一组匹配器匹配的警报（目标）静音。对于相等列表中的标签名称，目标警报和源警报必须具有相同的标签值。 
inhibit_rules:
  [ - <inhibit_rule> ... ]
```

## 告警分组

将类似的指标分组，比如说监控linux服务器会监控其负载，资源负载就是一个分组，这个分组下面会有cpu的指标，内存的指标。

每一个告警都会从配置文件中顶级的route进入路由树，需要注意的是顶级的route必须匹配所有告警(即不能有任何的匹配设置match和match_re)，每一个路由都可以定义自己的接受人以及匹配规则。

```yaml
# 所有报警信息进入后的根路由，用来设置报警的分发策略
route:
  receiver: 'default-receiver' # 默认情况下所有的告警都会发送给集群管理员default-receiver
  group_wait: 10s # 组告警等待时间。也就是告警产生后等待10s，如果有同组告警一起发出
  group_interval: 5m # 如果有不同的组，两组告警的间隔时间
  repeat_interval: 4h # 重复告警的间隔时间，减少相同邮件的发送频率
  group_by: ['alertname', 'cluster'] # 这里的标签列表是接收到报警信息后的重新分组标签，例如，接收到的报警信息里面有许多具有 cluster=A 和 alertname=LatncyHigh 这样的标签的报警信息将会批量被聚合到一个分组里面
  
  # 上面所有的属性都由所有子路由继承，并且可以在每个子路由上进行覆盖。
  routes:
  - receiver: 'database-pager' # 如果告警中包含service标签，并且service为MySQL或者Cassandra,则向database-pager发送告警通知，由于这里没有定义group_by等属性，这些属性的配置信息将从上级路由继承，database-pager将会接收到按cluster和alertname进行分组的告警通知。
    group_wait: 10s
    match_re: #可以用正则
      service: mysql|cassandra
  - receiver: 'frontend-pager'
    group_by: [product, environment] #按照标签product和environment对告警进行分组
    match:
      team: frontend
```

![img](assets/7efb9a62d6bf40b7b5aa7c90fc060ef8.png)

## 在prometheus中创建告警规则

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

cat node_rules.yml

```yaml
groups:
- name: node_rules #告警规则组名称
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

## 告警状态

Prometheus Alert 告警状态有三种状态：Inactive、Pending、Firing。

**Inactive**：这里什么都没有发生。

**Pending**：已触发阈值，但未满足告警持续时间，也就是for字段

**Firing**：已触发阈值且满足告警持续时间。警报发送给接受者。

## 使用Receiver接收告警信息

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

### 集成邮箱系统

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
#全局配置
global:
  resolve_timeout: 5m
  #邮箱服务器
  smtp_smarthost: smtp.gmail.com:587
  smtp_from: <smtp mail from>
  smtp_auth_username: <usernae>
  smtp_auth_identity: <username>
  smtp_auth_password: <password>

#配置路由树
route:
  group_by: ['alertname'] # 用于分组聚合，对告警通知按标签(label)进行分组，将具有相同标签或相同告警名称(alertname)的告警通知聚合在一个组，然后作为一个通知发送。
  group_wait: 10s # 分组内第一个告警等待时间，10s内如有第二个告警会合并一个告警
  group_interval: 10s # 发送新告警间隔时间
  repeat_interval: 1h # 重复告警间隔发送时间
  receiver: 'default-receiver'

#接收人
receivers:
  - name: default-receiver
    email_configs:
      - to: <mail to address>
        send_resolved: true
```

### 集成企业微信

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

配置自定义告警模板：

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

分组（group） ： 将类似性质的警报分类为单个通知
抑制（Inhibition） ： 当警报发出后， 停止重复发送由此警报引发的其他警报
静默（Silences） ： 是一种简单的特定时间静音提醒的机制  

![image-20210604151014911](https://gitee.com/c_honghui/picture/raw/master/img/20210604151036.png)

![image-20210604151047311](https://gitee.com/c_honghui/picture/raw/master/img/20210604151047.png)

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

