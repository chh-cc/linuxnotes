# K8S入门

## 官网

https://kubernetes.io/

每年可能会发布三次版本，可以去官网或github的changelog查看更新信息

查看1.20.0的changelog：https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG/CHANGELOG-1.20.md#changelog-since-v1190

## 为什么用k8s

随着项目越来越多，管理的容器越来越多，裸容器部署的方式越来越难管理

宕机没有自动恢复机制

健康检查

扩缩容方便

虽然K8s 1.20版本宣布将在1.23版本之后将不再维护dockershim，意味着K8s将不直接支持Docker，不过大家不必过于担心。一是在1.23版本之前我们仍然可以使用Docker，二是dockershim肯定会有人接盘，我们同样可以使用Docker，三是Docker制作的镜像仍然可以在其他Runtime环境中使用

## k8s高可用架构

![image-20210904204842750](https://gitee.com/c_honghui/picture/raw/master/img/20210904204849.png)

k8s致力于提供跨主机集群的自动部署、扩展、高可用以及运行应用程序容器的平台

master节点：整个集群的控制中枢，公司生产环境一般3个master节点就足够了，一般资源一次性给够，因为改动麻烦

- Kube-APIServer：集群的控制中枢，各个模块之间信息交互都要经过它，同时它也是集群管理、资源配置、整个集群安全机制的入口
- ControllerManager：集群的状态管理器，保证Pod或其他资源达到期望值
- Scheduler：集群的调度中心，它会根据指定的一系列条件，选择一个或一批最佳的节点，部署我们的Pod

Etcd：键值数据库，保存一些集群的信息，一般生产环境建议部署3个以上奇数个节点，规模大的话跟master分开，要用ssd盘，一般资源一次性给够

Node节点：工作节点

- Kubelet：负责监听节点上的Pod的状态，同时负责上报节点和节点上面Pod的状态，负责与Master节点通信，并管理节点上面的Pod
- Kube-proxy：负责Pod之间的通信和负载均衡，将指定的流量分发到后端正确的机器上

----------

Calico：符合CNI标准的网络插件，给每个Pod生成一个唯一的IP地址，并且把每个节点当作一个路由器

CoreDNS：用于k8s集群内部Service的解析