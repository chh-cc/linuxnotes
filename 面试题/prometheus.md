1. prometheus监控原理？

   prometheus定期从活跃目标主机拉取监控数据，目标主机可以通过配置静态或动态服务发现被prometheus server采集

   prometheus把采集的数据保存在本地或者云存储

   通过配置告警规则，把触发的告警发送到alertmanager

   通过配置告警接收方，alertmanager把告警发送到邮件或微信等

2. prometheus怎么监控k8s

   通过node-exporter监控节点指标，比如节点的cpu、内存利用率等
   
   通过kubelet内置的cadvisor监控pod指标，比如容器的cpu、内存利用率等
   
   通过kube-state-metrics监控k8s资源对象指标，比如deployment状态、副本数等
   
3. 如何监控微服务

   通过应用埋点监控

   如果是springcloud微服务，可以通过actuator依赖暴露监控数据

   如果是go微服务，可以通过引用 prometheus/client_golang 包暴露监控数据