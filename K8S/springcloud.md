Eureka：服务注册中心，其他springboot服务不需要配置service了，推荐用statefulset部署

![注册中心](assets/22cf5201f5de4aa4a8367f1fca1a76ef.png)

Zuul：网关，相当于一个网站所有后端的前门。所有的外部请求都会先经过微服务网关，用deployment部署

ConfigServer：配置中心，用来统一管理我们的配置，用deployment部署

