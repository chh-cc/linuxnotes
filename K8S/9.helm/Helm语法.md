# Helm语法

## helm模板

### 内置对象

- **Release**：这个对象描述了 release 本身的一些信息。它里面有几个对象：

  - .Release.Name：release 名称
  - .Release.Time：release 的时间
  - .Release.Namespace：release 的 namespace（如果清单未覆盖）
  - .Release.Service：release 服务的名称，一般是Helm
  - .Release.Revision：此 release 的修订版本号，从1开始累加。
  - .Release.IsUpgrade：如果当前操作是升级或回滚，则将其设置为 true。
  - .Release.IsInstall：如果当前操作是安装，则设置为 true。

- **Values**：使用该对象可以获取value.yaml文件中已定义的任何变量数值。默认情况下，Values 是空的。

- Chart：用于获取`Chart.yaml`文件的内容。。chart 指南中[Charts Guide](https://github.com/kubernetes/helm/blob/master/docs/charts.md#the-chartyaml-file)列出了可用字段，可以前往查看。

- Capabilities：这提供了关于 Kubernetes 集群相关的信息，该对象有如下方法：

  - .Capabilities.APIVersions  返回k8s集群API版本信息集合
  - .Capabilities.APIVersions.Has $version  用于检测指定的版本或资源在k8s集群是否可用，可用为true，例如`.Capabilities.APIVersions.Has apps/v1/Deployment`
  - .Capabilities.KubeVersion  提供了查找 Kubernetes 版本的方法。它具有以下值：Major（主版本号,如1），Minor（小版本号,如20），GitVersion，GitCommit，GitTreeState，BuildDate，GoVersion，Compiler，和 Platform。

- Template：用于获取当前模板的信息，包含如下两个对象：

  - .Template.Name 用于获取当前模板的名称和路径（例如 mychart/templates/mytemplate.yaml）
  - .Template.BasePath：当前模板目录的路径（例如 mychart/templates）。

### 表达式

模板表达式：

{{ .Release.Name }}, 通过双括号注入,小数点开头表示从最顶层命名空间引用

{{- 模版表达式 -}} ， 表示去掉表达式输出结果前面和后面的空行，去掉前面空行可以这么写{{- 模版表达式 }}, 去掉后面空格 {{ 模版表达式 -}}

### 变量

变量的作用域：

默认情况最左面的点( . ), 代表全局作用域，用于引用全局对象，中间的点，很像是js中对json对象中属性的引用方式。

helm全局作用域中有两个重要的全局对象：Values和Release



变量的定义格式：**$name := value**

如：{{- $relname := .Release.Name }}

引用自定义变量:

{{ $relname }}

### Values

**Values**对象是为chart模板提供值，这个对象的值有4个来源：

- chart包中的value.yaml
- 父chart包中的value.yaml
- 通过helm install或helm upgrade的-f或--values参数传入的自定义yaml文件
- 通过--set参数传入的值

```yaml
cat mychart/values.yaml 
replicaCount: 1
image:
  repository: nginx
  pullPolicy: IfNotPresent
  tag: "1.16"
selectorLabels: "nginx"

cat mychart/templates/deployment.yaml 
...
metadata:
  name: {{ .Release.Name }}
...
spec:
  replicas: {{ .Values.replicaCount }}
        
helm install web1 --dry-run mychart/
metadata:
  name: web1
...
spec:
  replicas: 1
```

### 内置函数

常用函数：http://masterminds.github.io/sprig/strings.html



**函数的使用格式：**

格式1：函数名 arg1 arg2

格式2：**agr1 ｜函数名** #实际使用中更偏向于管道符来将参数传递给函数



• quote：将值转换为字符串，即**加双引号**

   ```yaml
cat mychart/values.yaml         
nodeSelector:
  gpu: true

cat mychart/templates/deployment.yaml
apiVersion: apps/v1
...
      nodeSelector:
        gpu: {{ .Values.nodeSelector.gpu | quote }}

helm install web1 --dry-run mychart/
apiVersion: apps/v1
...
      nodeSelector:
        gpu: "true"
   ```

• default：设置默认值，**如果获取的值为空则使用这个默认值**( 以防止忘记定义而导致模板文件缺少字段无法创建资源，这时可以为字段定义一个默认值。)

```yaml
cat mychart/values.yaml         
namespace: ""

cat mychart/templates/deployment.yaml
apiVersion: apps/v1
...
metadata:
  namespace: {{ .Values.namespace | default "default" }}

helm install web1 --dry-run mychart/
apiVersion: apps/v1
...
metadata:
  namespace: default
```

• toYaml：**引用一块YAML内容**

• indent和nindent：**缩进字符串**,nindent用的多，一般是结合toYaml

```yaml
#在values.yaml里写结构化数据，引用内容块，结合上面的nindent换行缩进
#像健康检查，资源配额resources，或者端口，这都是一块一块的内容，可以通过toYaml引用

cat mychart/values.yaml
resources:
  limits:
    cpu: 100m
    memory: 128Mi
  requests:
    cpu: 100m
    memory: 128Mi

cat mychart/templates/deployment.yaml
apiVersion: apps/v1
...
    spec:
      containers:
      - name: web
        resources:
          {{- toYaml .Values.resources | nindent 10 }}
          
helm install web1 --dry-run mychart/
apiVersion: apps/v1
...
    spec:
      containers:
      - name: web
        resources:
          limits:
            cpu: 100m
            memory: 128Mi
          requests:
            cpu: 100m
            memory: 128Mi
```

• printf：用于格式化输出字符串内容，并且支持使用占位符，占位符取决于传入参数的类型

<img src="assets/image-20230112173613415.png" alt="image-20230112173613415" style="zoom:67%;" />

```shell
例子：
#以占位符的方式，输出.Chart.Name-.Chart.Version
printf "%s-%s" .Chart.Name .Chart.Version #返回结果：ingress-nginx-3.6.0
```

• trunc：用于截断字符串，通过使用正整数或负整数来分别表示从左向右截取的个数和从右向左截取的个数

```shell
trunc 5
```

• trimPrefix和trimSuffix：分别用于移除字符串中指定的前缀和后缀

```shell
trimPrefix "-"
```

• contains：用于测试一个字符串是否包含在另一个字符串里面，返回布尔值，包含在里面结果为true

```shell
{{- if contains $name .Release.Name -}}
...
{{- else -}}
```

• replace：用于执行简单的字符串替换，该函数需要传递三个参数

```shell
"I Am Test" | replace "" "-" #返回结果：I-Am-Test
```

### 流程控制

• **with**：主要就是用来**修改 . 作用域**的

适用于多次引用同个多层级里的内容

```yaml
{{ .Values.a.b.c.repository }}
{{ .Values.a.b.c.pullPolicy }}
转成：
{{- with .Values.a.b.c }}
{{ .repository }}
{{ .pullPolicy }}
{{- end }}

cat values.yaml
controller:
  image:
    repository: registry.cn-beijing.aliyuncs.com/dotbalo/controller
    tag: "v0.40.2"
    pullPolicy: IfNotPresent
    runAsUser: 101
    allowPrivilegeEscalation: true

cat templates/daemonset.yaml
      containers:
        - name: controller
          {{- with .Values.controller.image }}
          image: "{{.repository}}:{{ .tag }}{{- if (.digest) -}} @{{.digest}} {{- end -}}"
          {{- end }}
          
helm install ingress-nginx --dry-run .
      containers:
        - name: controller
          image: "registry.cn-beijing.aliyuncs.com/dotbalo/controller:v0.40.2"
```

因为**with语句里不能调用父级别的变量**，所以如果要调用父级别的变量，需要声明一个变量名，将父级别的变量值赋值给声明的变量

```yaml
cat configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  # 由于下方的with语句引入相对命令空间,无法通过.Release引入,提前定义relname变量
  {{- $name := .Release.Name }}
  {{- with .Values.data  }}
  ...
  release: {{ $name }}
  # 或者可以使用$符号,引入全局命名空间
  release: {{ $.Release.Name }}
  {{- end }}
```

• **if**语法

格式：

```shell
{{- if ... }}
...
{{- else if ... }}
...
{{- else }}
...
{{- end }}
```



操作符：and/eq/or/not



什么时候空类型：

整型：0

字符串：“”

列表：[]

字典：{}

布尔：false

以及所有的nil或null



例子：

```yaml
#例子1
cat values.yaml
rbac:
  create: true
  scope: false
  
cat clusterrole.yaml
#如果.Values.rbac.create的值为真且.Values.rbac.scope不为真
{{- if and .Values.rbac.create (not .Values.rbac.scope) -}}
xxxx
{{- end }}

#例子2
cat values.yaml
controller:
  kind: DaemonSet
  
cat controller-daemonset.yaml
{{- if or (eq .Values.controller.kind "DaemonSet") (eq .Values.controller.kind "Both") -}}
xxx
{{- end }}
```

• **range**主要用于循环遍历数组类型

语法1（遍历map类型，用于遍历键值对象）:

 ```yaml
#对于字典类型的结构，可以使用range获取到每个键值对的key和value（注意字典是无序的，所以遍历出来的结果也是无序的）
#变量$key代表对象的属性名，$val代表属性值
格式：
 {{- range $key, $val := 键值对象 }}
 {{ $key }}: {{ $val | quote }}
 {{- end}} 

cat values.yaml
controller:
  containerPort:
    http: 80
    https: 443

cat templates/controller-daemonset.yaml
spec:
  template:
    spec:
      containers:
        - name: controller
          ports:
          {{- range $key, $value := .Values.controller.containerPort }}
            - name: {{ $key }}
              containerPort: {{ $value }}
              protocol: TCP
          {{- end }}

#渲染后
spec:
  template:
    spec:
      containers:
        - name: controller
          ports:
            - name: http
              containerPort: 80
              protocol: TCP
            - name: https
              containerPort: 443
              protocol: TCP
 ```

语法2（数组类型遍历）：

```yaml
格式：
{{- range 数组 }}
{{ . | title | quote }} # . (点)，引用数组元素值。
{{- end }}

cat values.yaml
pizzaToppings:
  - mushrooms
  - cheese
  - peppers
  - onions

cat templates/test.yaml
{{- range .Values.pizzaToppings }}
- {{ . | quote }} 
{{- end }}

渲染后：
- "Mushrooms"
- "Cheese"
- "Peppers"
- "Onions"
```

### 公共模板

在templates目录中默认下划线开头的文件为公共模板(\_helpers.tpl)，**通过define在_helpers.tpl文件中定义子模板,通过template或include调用子模板**

**template不能被其他函数修饰，include可以，推荐用include**



格式：

```yaml
定义模版:
{{ define "模版名字" }}
模版内容
{{ end }}

引用模版:
{{ include "模版名字" 作用域}}
```

例子：

```yaml
#公共模板
cat templates/_helpers.tpl
{{- define "ingress-nginx.labels" -}}
helm.sh/chart: {{ include "ingress-nginx.chart" . }}
{{ include "ingress-nginx.selectorLabels" . }}
{{- if .Chart.AppVersion }}
app.kubernetes.io/version: {{ .Chart.AppVersion | quote }}
{{- end }}
app.kubernetes.io/managed-by: {{ .Release.Service }}
{{- end -}}
...
{{- define "ingress-nginx.chart" -}}
#以占位符的方式，输出.Chart.Name-.Chart.Version，取前63个字符，去掉末尾的-
{{- printf "%s-%s" .Chart.Name .Chart.Version | replace "+" "_" | trunc 63 | trimSuffix "-" -}}
{{- end -}}
...
{{- define "ingress-nginx.selectorLabels" -}}
app.kubernetes.io/name: {{ include "ingress-nginx.name" . }}
app.kubernetes.io/instance: {{ .Release.Name }}
{{- end -}}
...
{{- define "ingress-nginx.name" -}}
# 如果.Values.nameOverride为空的，那么就取默认default的值 .Chart.Name
{{- default .Chart.Name .Values.nameOverride | trunc 63 | trimSuffix "-" -}}
{{- end -}}

#Chart
cat Chart.yaml
appVersion: 0.40.2
name: ingress-nginx
version: 3.6.0


cat templates/clusterrole.yaml
metadata:
  labels:
    {{- include "ingress-nginx.labels" . | nindent 4 }}

#渲染后
helm install ingress-nginx --dry-run .
metadata:
  labels:
    helm.sh/chart: ingress-nginx-3.6.0
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/version: "0.40.2"
    app.kubernetes.io/managed-by: Helm
```

## 手动编写helm模板

helm 基于java 的chart 模板开发

```shell
helm create java
cd java ;rm -fr templates/*
```

修改values.yaml

```yaml
# Default values for java. 
# This is a YAML-formatted file. 
# Declare variables to be passed into your templates. 
 
replicaCount: 1 
namespace: neighbour 
label: {} 
annotations: {} 
 
#image 
image: 
  repository: nginx 
  pullPolicy: IfNotPresent 
  imageUrl: harbor.test.com/neiour/bl-test 
  tag: "v1" 
 
imagePullSecrets: [] 
nameOverride: "" 
fullnameOverride: "" 
 
#pod更新策略 
strategy: 
  rollingUpdate: 
    maxSurge: 1 
    maxUnavailable: 0 
 
#java 应用配置参数 
jarInfo: 
  name: "/opt/neighbour-group.jar" 
  port: 10051 
  version: v1 
  args: ['-XX:+UnlockExperimentalVMOptions','-XX:+UseCGroupMemoryLimitForHeap','-XX:MaxRAMFraction=1'] 
  env: 
  - name: NACOS_CONFIG_ADDR 
    value: "nacos-headless.nacos.svc.cluster.local:8848" 
  - name: SEATA_CONFIG_ADDR 
    value: "seata-server.default.svc.cluster.local:8091" 
  - name: enable_multi_nacos 
    value: "true" 
  - name: additional_nacos_address 
    value: http://nacos-headless.nacos.svc.cluster.local:8848  
  addenv: {} 
 
# 资源限制 
resources: 
  limits: 
    cpu: 1 
    memory: 1 
  requests: 
    cpu: 100Mi 
    memory: 200Mi 
 
#探针 
readinessProbe: 
  initialDelaySeconds: 15 
  periodSeconds: 10 
 
livenessProbe: 
  initialDelaySeconds: 25 
  periodSeconds: 10 
  failureThreshold: 3 
 
#节点亲和度 
nodeSelector: {} 
##污点 
tolerations: [] 
##容忍 
affinity: {} 
 
serviceAccount: 
  # Specifies whether a service account should be created 
  create: true 
  # Annotations to add to the service account 
  annotations: {} 
  # The name of the service account to use. 
  # If not set and create is true, a name is generated using the fullname template 
  name: ""

podAnnotations: {}

podSecurityContext: {}
  # fsGroup: 2000

securityContext: {}
  # capabilities:
  #   drop:
  #   - ALL
  # readOnlyRootFilesystem: true
  # runAsNonRoot: true
  # runAsUser: 1000

service:
  type: ClusterIP
  port: 80
  protocol: TCP
  annotations: {}

ingress:
  enabled: false
  className: ""
  annotations: {}
    # kubernetes.io/ingress.class: nginx
    # kubernetes.io/tls-acme: "true"
  hosts:
    - host: chart-example.local
      paths:
        - path: /
          pathType: ImplementationSpecific
  tls: []
  #  - secretName: chart-example-tls
  #    hosts:
  #      - chart-example.local
```

