# K8S简介

Kubernetes是一个**开源的容器管理平台**，简称k8s，用于管理多个主机上的容器化应用程序，提供应用程序的快速部署，扩缩容，升级，维护和扩展等机制，利用service可以实现服务注册、发现以及四层负载均衡，通过ingress可以实现七层负载均衡等功能，Kubernetes这个名字源于希腊语，意思是舵手或飞行员

## 官网

https://kubernetes.io/

每年可能会发布三次版本，可以去官网或github的changelog查看更新信息

查看1.20.0的changelog：https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG/CHANGELOG-1.20.md#changelog-since-v1190

虽然K8s 1.20版本宣布将在1.23版本之后将不再维护dockershim，意味着K8s将不直接支持Docker，不过大家不必过于担心。一是在1.23版本之前我们仍然可以使用Docker，二是dockershim肯定会有人接盘，我们同样可以使用Docker，三是Docker制作的镜像仍然可以在其他Runtime环境中使用

## k8s架构

<img src="assets/image-20230625115935241.png" alt="image-20230625115935241" style="zoom:80%;" />


整个k8s集群分为两大部分：

- **控制平面/master节点**：负责控制并保持整个集群的正常运转，一般包括**controller plane的apiserver、controller-manager、scheduler、etcd**（如果磁盘 IO 、网络依赖较高，建议独立部署etcd）。公司生产环境**最少三台Master节点做高可用**，一般资源一次性给够，因为改动麻烦
  - kube-apiserver：负责接收所有请求，**集群内对集群的任何修改都是通过命令行、ui把请求发给apiserver才能执行**，**各个组件通过apiserver进行通信**：各个组件通过apiserver将信息存入etcd，当要获取这些信息的时候再通过apiserver的REST接口（用get、watch、list）去请求
  - kube-controller-manager：**负责维护集群的状态**，执行各种控制器，比如故障检测、自动扩展、滚动更新，保证Pod或其他资源达到期望值
  - kube-scheduler：**负责调度**，它会根据指定的调度规则，把pod调度到一个或一批最佳的node节点
  - etcd：**保存了整个集群的状态**，一般生产环境建议部署3个以上奇数个节点（防止脑裂），规模大的话跟master分开，要用ssd盘，一般资源一次性给够，etcd备份：https://github.com/kubesphere/kubekey/blob/235de693d6e68740b573362b8fd3e1226d8c574f/pkg/etcd/templates/backup_script.go

- **工作节点/Node节点**：运行容器的任务依赖于每个工作节点上运行的组件。理论上一个k8s集群可以运行5000个节点，但是节点太多master负担也大，一般100~200个节点就够了；或者用公有云的全托管集群，master节点是公有云提供的，非常大，这样一个集群上千个节点是没问题的
  - Kubelet：负责容器真正运行的核心组件。负责node节点上pod的创建、修改、删除等全生命周期管理，定期汇报node状态给apiserver，通过apiserver间接与etcd交互读取集群配置信息。
  - Kube-proxy：kube-proxy负责转发，一旦发现service关联的某个pod信息发生变化，就会把这些变化转换成ipvs规则，完成对后端pod的负载均衡
  - 容器运行时：Container runtime负责镜像管理以及Pod和容器的真正运行（CRI）

**附加组件**：

- CNI容器网络接口插件：calico, flannel（如果没有实施网络策略的需求，那么就直接用flannel，开箱即用；否则就用calico了，但要注意如果网络使用了巨型帧，那么注意calico配置里面的默认值是1440，需要根据实际情况进行修改才能达到最佳性能），terway（阿里云）
- CoreDNS：CoreDNS负责为整个集群提供DNS服务
- Ingress控制器：Ingress Controller为服务提供外网流量入口

## k8s核心功能全景图



首先从容器最基础的概念出发，遇到了容器间的“紧密协作”关系难题，于是有了**pod**；然后我们希望一次能启动多个应用实例，于是有了**deployment**；有了一组相同的pod后，我们需要一个固定ip和端口号以负载均衡的方式访问它们，于是有了**service**；

如果两个pod之间不仅有访问关系，还要密码认证，于是有了**secret**来保存认证信息挂载到容器里；

关于应用运行的形态，k8s基于pod定义了新的对象，比如**job**，一次性运行的pod；比如**daemonset**，每个宿主机必须且只能运行一个副本的守护进程；比如**cronjob**，做定时任务等。

在 Kubernetes 项目中，我们所推崇的使用方法是：

首先，通过一个“编排对象”，比如 Pod、Job、CronJob 等，来描述你试图管理的应用；

然后，再为它定义一些“服务对象”，比如 Service、Secret、Horizontal Pod Autoscaler（自动水平扩展器）等。这些对象，会负责具体的平台级功能。这种使用方法，就是所谓的“声明式 API”。这种 API 对应的“编排对象”和“服务对象”，都是 Kubernetes 项目中的 API 对象（API Object）。

这就是 Kubernetes 最核心的设计理念。

