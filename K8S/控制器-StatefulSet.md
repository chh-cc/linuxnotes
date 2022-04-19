# StatefulSet

## statefulset

### 什么是statefulset

deployment认为，一个应用的所有 Pod，是完全一样的，它们没有启停顺序的。

deployment并不能满足所有应用的编排，尤其是分布式应用，它的多实例之间有依赖关系，比如主从关系；还有数据存储类应用，它的多个实例往往都会在本地磁盘保存一份数据。而这些实例一旦被杀掉，即便重建，实例与数据之间的对应关系也丢失了。所以这种实例之间有不对等关系，以及实例对外部数据有依赖关系的应用，称为“有状态应用”（Stateful Application）。

k8s基于deployment扩展出了statefulset，**用来编排有状态应用**。

### statefulset设计

statefuleset把应用状态抽象为两种情况：

- 拓扑状态。
  -  应用的多个实例之间不是完全对等的关系，这些应用实例必须按某些顺序启动（statefulset中每个Pod**有order编号**，会**按序号创建、删除、更新Pod**），比如应用主节点A要先于从节点B启动。
  - A和B两个pod删掉后，它们也要按照这个启动顺序创建，并且新创建的pod要和原来pod的网络标识一样（通过配置Headless Service，使每个pod**有一个唯一的网络标识**），这样才能访问到这个新pod。
- 存储状态。应用的多个实例分别绑定了不同的存储数据，通过配置pvc template，每个Pod有一块独享的pv存储盘



statefulset核心功能：

通过某种方式记录这些状态，然后在pod被重建时能够恢复这些状态。



## 创建一个statefulset应用

定义一个简单的statefulset：

```yaml
vim nginx-sts.yaml

apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  clusterIP: None
  selector:
    app: nginx
  ports:
  - name: web
    port: 80
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: nginx-web
spec:
  updateStrategy:
    type: RollingUpdate
    rollingUpdate:
      partition: 0
  serviceName: "nginx"
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:  
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.15.2
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www-storage
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www-storage
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 2Gi
```

创建pod

```shell
kubectl create -f nginx-sts.yaml
```

查看pods

```shell
[root@k8s-master-1 statefulset]# kubectl get pods
NAME                                      READY   STATUS    RESTARTS   AGE
nginx-web-0                                     1/1     Running   0          2m39s
nginx-web-1                                     1/1     Running   0          2m34s
nginx-web-2                                     1/1     Running   0          2m34s

#pod名字从0开始编号
#web-0创建好后才会继续创建web-1
```

查看svc

```shell
kubectl get svc
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
nginx        ClusterIP   None         <none>        80/TCP    2m48s
```

扩容

```shell
kubectl scale --replicas=4 sts web
```

## StatefulSet更新策略

### Rolling Updates

`.spec.updateStrategy.type` 字段的默认值是 RollingUpdate，该策略为 StatefulSet 实现了 Pod 的**自动滚动更新**。

在用户更新 StatefulSet 的pod模板（ `.spec.tempalte` 字段）时，StatefulSet Controller 从最后一个 Pod 开始，逐一更新这个 StatefulSet 管理的每个 Pod；当正在更新的 Pod 达到了 Running 和 Ready 的状态之后，才继续更新其前序 Pod



此外，StatefulSet 的“滚动更新”还允许我们进行更精细的控制，比如金丝雀发布(Canary Deploy)或者灰度发布：

**.spec.updateStrategy.rollingUpdate.partition**：滚动升级时，保留旧版本Pod的数量

假如Partition设为2，有4个pod（web-0~web-3），则只会更新web2~web3，可以利用这种机制实现简单的灰度发布：

```shell
[root@k8s-master01 ~]# kubectl get po -w
NAME    READY   STATUS    RESTARTS   AGE
web-0   1/1     Running   0          17m
web-1   1/1     Running   0          17m
web-2   1/1     Running   0          17m
web-3   1/1     Running   0          17m
----
web-3   1/1     Terminating   0          18m
web-3   0/1     Terminating   0          18m
web-3   0/1     Terminating   0          18m
web-3   0/1     Terminating   0          18m
web-3   0/1     Pending       0          0s
web-3   0/1     Pending       0          0s
web-3   0/1     ContainerCreating   0          0s
web-3   1/1     Running             0          1s
web-2   1/1     Terminating         0          18m
web-2   0/1     Terminating         0          18m
web-2   0/1     Terminating         0          18m
web-2   0/1     Terminating         0          18m
web-2   0/1     Pending             0          0s
web-2   0/1     Pending             0          0s
web-2   0/1     ContainerCreating   0          0s
web-2   1/1     Running             0          1s
```

###  On Delete

如果 StatefulSet 的 `.spec.updateStrategy.type` 字段被设置为 OnDelete，当您修改 `.spec.template` 的内容时，StatefulSet Controller 将不会自动更新其 Pod。**必须手工删除 Pod才会重新创建 Pod** ，使用修改过的 `.spec.template` 的内容创建新 Pod。

## 级联删除和非级联删除

### 级联删除

删除sts时同时删除pod

```shell
kubectl delete sts web
```

### 非级联删除

删除sts时不删除pod

```shell
kubectl delete sts web --cascade=false
```

删除后pod变成孤儿pod，此时删除pod后不会重建

## volumeClaimTemplates

有了PV和PVC的设计，使得StatefulSet 对存储状态的管理成为了可能。以这个statefulset为例：

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  serviceName: "nginx"
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.9.1
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes:
      - ReadWriteOnce
      resources:
        requests:
          storage: 1Gi
```

添加了volumeClaimTemplates字段后，该statefulset管理的pod都会声明一个对应的PVC，而这个PVC的定义就来自这个volumeClaimTemplates字段。

PVC的名字会被分配一个和pod名字一样的编号：

```shell
$ kubectl get pvc -l app=nginx
NAME        STATUS    VOLUME                                     CAPACITY   ACCESSMODES   AGE
www-web-0   Bound     pvc-15c268c7-b507-11e6-932f-42010a800002   1Gi        RWO           48s
www-web-1   Bound     pvc-15c79307-b507-11e6-932f-42010a800002   1Gi        RWO           48s

#<PVC 名字 >-<StatefulSet 名字 >-< 编号 >
```

用kubectl delete删除这两个pod，pod被重建后依然同pvc绑定在一起。

## 实践

