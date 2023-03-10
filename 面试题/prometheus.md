1. prometheus监控原理？

   prometheus定期从活跃目标主机拉取监控数据，目标主机可以通过配置静态或动态服务发现被prometheus server采集

   prometheus把采集的数据保存在本地或者云存储

   通过配置告警规则，把触发的告警发送到alertmanager

   通过配置告警接收方，alertmanager把告警发送到邮件或微信等

2. prometheus怎么监控k8s

   