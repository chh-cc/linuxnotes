# Helm语法

## helm模板

### 内置对象

- **Release**：这个对象描述了 release 本身的一些信息。它里面有几个对象：

  - Release.Name：release 名称
  - Release.Time：release 的时间
  - Release.Namespace：release 的 namespace（如果清单未覆盖）
  - Release.Service：release 服务的名称
  - Release.Revision：此 release 的修订版本号，从1开始累加。
  - Release.IsUpgrade：如果当前操作是升级或回滚，则将其设置为 true。
  - Release.IsInstall：如果当前操作是安装，则设置为 true。

- **Values**：使用该对象可以获取value.yaml文件中已定义的任何变量数值。默认情况下，Values 是空的。

- Chart：用于获取`Chart.yaml`文件的内容。。chart 指南中[Charts Guide](https://github.com/kubernetes/helm/blob/master/docs/charts.md#the-chartyaml-file)列出了可用字段，可以前往查看。

- Files：这提供对 chart 中所有非特殊文件的访问。虽然无法使用它来访问模板，但可以使用它来访问 chart 中的其他文件。请参阅 "访问文件" 部分。

  - Files.Get 是一个按名称获取文件的函数（.Files.Get config.ini）
  - Files.GetBytes 是将文件内容作为字节数组而不是字符串获取的函数。这对于像图片这样的东西很有用。

- Capabilities：这提供了关于 Kubernetes 集群相关的信息，该对象有如下方法：

  - Capabilities.APIVersions  返回k8s集群API版本信息集合
  - Capabilities.APIVersions.Has $version  用于检测指定的版本或资源在k8s集群是否可用，例如apps/v1/Deployment
  - Capabilities.KubeVersion  提供了查找 Kubernetes 版本的方法。它具有以下值：Major（主版本号），Minor（小版本号），GitVersion，GitCommit，GitTreeState，BuildDate，GoVersion，Compiler，和 Platform。

- Template：用于获取当前模板的信息，包含如下两个对象：

  - .Template.Name 用于获取当前模板的名称和路径（例如 mychart/templates/mytemplate.yaml）
  - .Template.BasePath：当前模板目录的路径（例如 mychart/templates）。

  

除了系统自带的变量，我们也可以**自定义模板变量**

{{- $relname := .Release.Name }}

引用自定义变量:

{{ $relname }}