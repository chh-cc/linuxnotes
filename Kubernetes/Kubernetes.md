# Kubernetes

**Kubernetes**的名字来自希腊语，意思是“*舵手”* 或 “*领航员”*。

Kubernetes是容器集群管理系统，是一个开源的平台，可以实现容器集群的自动化部署、自动扩缩容、维护等功能。

**具体功能：**

- 自动化容器部署和复制。
- 实时弹性收缩容器规模。
- 容器编排成组，并提供容器间的负载均衡。

在Kubenetes中，所有的容器均在pod中运行,一个Pod可以承载一个或者多个相关的容器，在后边的案例中，同一个Pod中的容器会部署在同一个物理机器上并且能够共享资源。

用户可以自己创建并管理Pod,Kubernetes将这些操作简化为两个操作：基于相同的Pod配置文件部署多个Pod复制品；创建可替代的Pod当一个Pod挂了或者机器挂了的时候。而Kubernetes API中负责来重新启动，迁移等行为的部分叫做“replication controller”，它根据一个模板生成了一个Pod,然后系统就根据用户的需求创建了许多冗余，这些冗余的Pod组成了一个整个应用，或者服务，或者服务中的一层。一旦一个Pod被创建，系统就会不停的监控Pod的健康情况以及Pod所在主机的健康情况，如果这个Pod因为软件原因挂掉了或者所在的机器挂掉了，replication controller 会自动在一个健康的机器上创建一个一摸一样的Pod,来维持原来的Pod冗余状态不变，一个应用的多个Pod可以共享一个机器。

## 架构

架构图

![image-20210319234059798](https://gitee.com/c_honghui/picture/raw/master/img/20210319234106.png)

kubectl:客户端命令行工具，作为整个系统的操作入口。

master主要包含以下组件：

- API Server（核心服务）：资源操作的唯一入口,提供restAPI及安全访问控制，是**整个集群的网关**
- Scheduler：负责资源的调度，按照预定的调度策略将Pod调度到相应的机器上
- Controller Manager：负责维护集群的状态，比如故障检测、自动扩展、滚动更新等。
- etcd：保存了整个集群的状态；分布式高性能键值数据库,存储整个集群的所有元数据

node组件：

- kubelet：负责维护容器的生命周期，同时也负责Volume（CVI）和网络（CNI）的管理；
- kube-proxy：负责为Service提供cluster内部的服务发现和负载均衡
- K8S集群通过cAdvisor监控，cAdvisor集成在Kubelet
- 容器需要跨主机通信，这时候需要网络插件Flannel

DNS：一个可选的DNS服务，用于为每个Service对象创建DNS记录，这样所有的Pod就可以通过DNS访问服务了。

