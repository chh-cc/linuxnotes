# Prometheus

## Prometheus operator

1、prometheus operator添加监控项

创建一个servicemonitor对象，关联暴露metrics接口的service，注意可能要配置指定标签才能被Prometheus关联识别到

2、配置自动发现

通过添加一个额外的配置来自动发现并监控具有某个指定annotations的service，要把这个配置文件创建为secret，然后修改Prometheus对象配置additionalScrapeConfigs来关联这个secret

3、配置告警规则

创建一个Prometheusrule对象，配置告警规则，注意可能要配置指定标签才能被Prometheus关联识别到

4、配置alertmanager告警发送方式

创建一个alertmanager.yaml配置路由规则、接收器、告警媒介等，然后把这个文件创建为secret对象，Alertmanager实例在默认情况下会通过`alertmanager-{ALERTMANAGER_NAME}`的命名规则去查找Secret配置

## Prometheus



5、样本的格式

<--------------- metric ---------------------><-timestamp -><-value->

6、样本类型

瞬时向量：该时间序列中最新的一个样本值

区间向量：一段时间范围内的数据 http_requests_total{}[5m]

偏移修饰器：如5分钟前的样本数据 http_request_total{}[1d] offset 1d

7、指标类型

counter：只增不减

gauge：可增可减

histogram和summary：统计样本分布情况

8、 Prometheus原理

Prometheus读取配置文件的监控项，定期拉取metrics，经过relabel后存储在TSDB中

如果触发告警规则就发送alert到alertmanager组件，alertmanager组件根据路由配置通过媒介如企业微信来发送告警