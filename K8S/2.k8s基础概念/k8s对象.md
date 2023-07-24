命令太多不好记

k8s里面操作的资源实体，就是k8s的对象，可以**使用yaml来声明对象**，然后用`kubectl apply -f xxx.yaml`创建对象

常规应用里，我们把应用程序数据存储在数据库中，**k8s将其数据以k8s对象的形式通过apiserver存储在etcd中**



对象的spec和status

每一个k8s对象都包含两个重要的字段：

- `spec`必须由您来提供，描述您对该对象期望的**目标状态**
- `status`只能由k8s系统来修改，描述该对象在k8s系统中的**实际状态**

k8s通过对应的**控制器**，不断的使实际状态趋于期望的目标状态



编写一个资源对象，比如pod

```shell
#集群中挑一个同类资源获取它的yaml
kubectl get pod xxx -oyaml
#kubectl run my-tomcat --image=tomcat --dry-run -oyaml #干跑一遍

然后修改yaml
```

