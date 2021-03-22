# Kubernetes

Kubernetes（k8s）是自动化容器操作的开源平台。这些容器操作包括：部署、调度和节点集群间扩展。

**具体功能：**

- 自动化容器部署和复制。
- 实时弹性收缩容器规模。
- 容器编排成组，并提供容器间的负载均衡。

## 架构

架构图

![image-20210319234059798](https://gitee.com/c_honghui/picture/raw/master/img/20210319234106.png)

kubectl:客户端命令行工具，作为整个系统的操作入口。

master主要包含以下组件：

- API Server（核心服务）:集群资源访问控制入口,提供restAPI及安全访问控制，是**整个集群的网关**
- Scheduler:**调度器**,负责把业务容器调度到最合适的node节点
- Controller Manager:执行整个系统的后台任务，包括节点状态状况、Pod个数、Pods和Service的关联等。
- etcd:分布式高性能键值数据库,存储整个集群的所有元数据

node组件：

- kubelet:直接接受API Server的调度，API Server控制kubelet创建容器，Kubelet控制docker创建容器，K8S的容器叫Pod，Kube-Proxy给Pod做端口映射给用户访问
- kube-proxy:运行在每个计算节点上，负责Pod网络代理。定时从etcd获取到service信息来做相应的策略。
- K8S集群通过cAdvisor监控，cAdvisor集成在Kubelet
- 容器需要跨主机通信，这时候需要网络插件Flannel

DNS：一个可选的DNS服务，用于为每个Service对象创建DNS记录，这样所有的Pod就可以通过DNS访问服务了。

## Pod

- K8S创建一个Pod资源，然后再Pod资源里运行容器。

​       Pod代表着kubernetes的部署单元和原子运行单元，即一个应用程序的单一运行实例。

- kubernetes的网络模型要求各Pod对象的ip地址在同一网段，各Pod之间使用其ip直接通信，无论它们运行在哪个节点。

- Pod对象中各进程运行于彼此隔离的容器中，并于各容器间共享两种关键资源：**网络和存储卷**
  - 网络：每个pod都会被分配一个集群内的专用ip，同一pod内的所有容器共享pod的network和uts名称空间，包括主机名、ip、端口等。因此这些容器间的通信可以基于**本地回环lo**进行。而与pod外的其他组件的通信需要使用service资源的**clusterip**及相应端口完成。

vim nginx_pod.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labele:
    app: web
spec:
  containers:
    - name: nginx
      images: nginx:1.13
      ports:
        - containerPort: 80
       
kubectl create -f nginx_pod.yaml

kubectl get pod nginx -o wide
Name	READY	STATUS	RESTARTS	AGE		IP				NODE
nginx	0/1     Running	0			2m		172.16.48.2	k8s-node2
```



k8s 创建一个pod，控制docker至少启动两个容器，一个业务容器，一个基础Pod容器

```shell
docker ps可以看到上面的pod资源启动了两个容器
docker inspect命令查看容器信息，可以看到pod容器有ip地址，而nginx容器没有ip地址，nginx容器跟pod容器共用网络
```



Pod常用操作：

```shell
#创建一个pod
kubectl create -f nginx_k8s.yaml
#查看pod资源列表
kubectl get pods
#查看某个pod的详细信息(可用于排错)
kubectl describe pod test
#删除某个pod
kubectl delete pod test [--force --grace-period=0]
```

## Replication Controller

RC确保任何时间kubernetes中都有指定数量的Pod在运行。在此基础上， RC还提供滚动升级、升级回滚等功能。

### 创建一个RC

vim nginx-rc.yaml

```yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: myweb
spec:
  replicas: 2
  selector:
    app: myweb
  template:
    metadata:
      labels:
        app: myweb
    spec:
      containers:
      - name: myweb
        images: nginx:1.13
        ports:
        - containerPort: 80
        
kubectl create -f nginx-rx.yaml

kubectl get rc
kubectl get pods
```

### rc的滚动升级

## service

- pod对象拥有ip，但是无法确保pod对象重启或重建后保持不变。

​       service被用于在被访问的pod对象中添加一个有着固定ip的中间层，客户端向此地址发起访问后由service资源调度并代理到后端的pod对象。

- service ip是一种虚拟ip，也称为cluster ip，它专用于**集群内通信**，通常使用专用的地址段：10.96.0.0/12

​       但集群网络属于私有地址，要将集群外部的访问流量引入集群内的常用方法是通过**节点网络**进行，实现方法是通过**工作节点的ip地址和某端口**（nodeport）接入请求并将其代理到相应的service的cluster ip上的服务端口，而后由**service对象**将请求代理到后端的pod对象的pod ip及应用程序监听的端口。

- service有三种类型：
  - 仅用于集群内部通信的clusterIP类型
  - 接入集群外部请求的nodeport类型
  - loadbalancer类型，把外部请求负载均衡至多个node的主机ip的nodeport上

创建一个service

vim nginx-svc.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: myweb
spec:
  type: NodePort
  ports:
    - port: 80
      nodePort: 30000
      targetPort: 80
  selector:
    app: myweb
    
kubectl create -f nginx-svc.yaml
```

