# Deployment

用于部署无状态的服务，这个是最常用的控制器。一般用于管理维护企业内部无状态的微服务，比如configserver、zuul、springboot。它可以管理多个副本的Pod实现无缝迁移、自动扩缩容、自动灾难恢复、一键回滚等功能。

## 使用deployment创建pod

```shell
cat deployment.yml

```



```shell
kubectl create -f deployment.yml

kubectl get deploy -owide

NAME                               READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES         SELECTOR
nginx-deployment   					3/3     3            3           41s   nginx        nginx:1.17.1   app=nginx-pod

#NAME：deployment名称
#READY：pod状态
#UP-TO-DATE：已经达到期望状态的被更新的副本数
#AVAILABLE：已经可用的副本数
#AGE：应用程序运行的时间
#CONTAINERS：容器名称
#IMAGES：镜像名称
#SELECTOR：pod的标签
```





