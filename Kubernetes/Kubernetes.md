# Kubernetes

## 架构

架构图

![image-20210319234059798](https://gitee.com/c_honghui/picture/raw/master/img/20210319234106.png)

master主要包含以下组件：

- 运维开发通过API Server（核心服务）创建资源，API Server通过Scheduler调度Node节点
- Controller保证每个Pod高可用，只要它挂了就立马起个新的
- etcd相当于kubernetes的数据库，所有状态信息都持久存储在etcd（etc是独立的服务组件，并不隶属于kubernetes集群）

node组件：

- kubelet直接接受API Server的调度，API Server控制kubelet创建容器，Kubelet控制docker创建容器，K8S的容器叫Pod，Kube-Proxy给Pod做端口映射给用户访问
- K8S集群通过cAdvisor监控，cAdvisor集成在Kubelet
- 容器需要跨主机通信，这时候需要网络插件Flannel

核心组件：

- KubeDNS
- Dashboard
- Heapster
- Ingress Controller

扩展组件

核心功能：

- 自我修复
- 服务发现和负载均衡
- 自动部署和回滚
- 弹性伸缩

## 网络

kubernetes的网络中主要存在三种类型：

节点网络：配置于主机的网络接口，用于各主机之间通信

Pod网络：是一个虚拟网络，配置于Pod中容器的网络接口上，需要借助插件

Service网络：是一个虚拟网络，

## Pod

K8S创建一个Pod资源，然后再Pod资源里运行容器。

### 创建一个pod

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

## service资源

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

