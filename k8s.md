## k8s

## 架构和组件

1. k8s与docker的关系？

   docker提供了容器的生命周期管理和镜像构建运行时容器，主要优点是将应用和所需设置、依赖打包到一个容器中，提高可移植性。k8s用于关联和编排多主机上运行容器。

2. k8s如何实现集群管理？（哪些组件及功能？）

   k8s划分为控制平面和工作节点，控制平面包括apiserver、controller-manager、scheduler、etcd，这些组件实现了整个集群的安全控制、故障恢复、自动扩缩、Pod调度等。

   apiserver：资源增删查改的唯一入口，各个组件通过apiserver进行通信：各个组件通过apisever**将信息存入etcd中**，当需要获取这些数据时再通过apiserver提供的**REST接口**（用get、list、watch）去请求，从而实现各个组件的通信。

   controller-manager：**维护集群的状态**，执行各种控制器，故障检测、自动扩展、滚动更新、保证pod数量达到期望值等

   scheduler：通过API Server的Watch接口监听到新建Pod副本的信息后，把pod调度到一个或一批合适的节点

   etcd：保存了整个集群的状态

   

   kubelet：负责节点上**pod的**创建、修改、删除等**全生命周期管理**，定期汇报node状态给apiserver

   kube-proxy：**负责转发**，一旦发现service关联的pod信息发生改变，就会创建ipvs规则，完成serive对后端pod的转发

   容器运行时：负责镜像管理和pod容器的真正运行，比如docker

3. k8s的网络模型？

   Kubernetes网络模型中每个Pod都拥有一个独立的IP地址，并假定所有Pod都在一个可以直接连通的、扁平的网络空间中。所以不管它们是否运行在同一个Node（宿主机）中，都要求它们可以直接通过对方的IP进行访问。

   一个 Pod 内部的所有容器共享一个网络堆栈。

4. k8s的CNI模型？

   k8s的扁平网络空间是由CNI插件建立的，常见的有flannel和calico

   flannel能协助k8s给每一个节点的容器都分配不相互冲突的IP地址，它能在这些ip地址建立一个覆盖网络，容器可以在这个网络中通信。

5. k8s的网络策略？

   网络策略的主要功能是对pod间的网络通信进行限制和准入控制，设置方式是将pod的label作为查询条件，设置允许或禁止访问的pod列表；默认所有pod没有隔离；网络策略功能由第三方网络插件提供，如calico

## pod

1. kube-proxy ipvs原理

   创建了service后会在宿主机生成一个虚拟网卡（kube-ipvs0），并给它分配一个service vip，kube-proxy为这个地址设置ipvs虚拟主机，并设置这几个虚拟主机使用轮询模式来作为负载策略。这时候任何发往这个IP地址的请求都会转发到某一个后端pod。

   ⼆者有着本质的差别：iptables是为防⽕墙⽽设计的；IPVS则专门⽤于⾼性能负载均衡，并使⽤更⾼效的数据结构（Hash表），允许⼏乎⽆限的规模扩张。

2. pod有哪些状态

   pending：pod的yaml文件已经提交给k8s，api对象已经被保存在etcd中，但pod可能因为一些原因无法顺利创建，比如**pod调度失败**

   failed：pod至少有一个容器不正常退出

   unknown：意味pod的状态不能被kubelet汇报给apiserver，可能是master和node之间通信出了问题

   succeeded：pod内所有容器正常退出没有重启

   running：pod被调度成功，并且至少有一个容器正常运行

3. pod重启策略

   always：容器失效时，kubelet自动重启该容器

   onfailed：容器异常退出时，kubelet自动重启该容器

   never：都不重启

   不同控制器的重启策略：deployment、statefulset、daemonset都要always；job为onfailed或never

4. pod的健康检测

   livenessprobe存活探针：用于判断容器是否running，如果检测到不健康则**杀掉容器**，然后根据**重启**策略做相应处理

   readnessprobe就绪探针：判断容器是否ready，如果检测失败则把pod**从endpoint中移除**，不然外部流量进入pod

   startupprobe启动探针：配置了startupprobe会**先禁用其他探针**，直到检测成功为止，成功后就不再探测，适合启动慢的程序

   

   ExecAction：在容器内执行一个命令，如果返回值为0，则认为容器健康

   TCPSocketAction：对指定端口的容器IP进行TCP检查，如果端口打开则认为容器健康

   HttpGetAction：对指定端口和路径的容器IP进行HTTP Get请求，如果状态码在200~400之间则认为容器健康

5. 如何选择探针

   容器启动时间较长（超出 `initialDelaySeconds + failureThreshold × periodSeconds`）则用启动探针

   希望容器在探测失败时被杀死并重新启动，那么请指定一个存活态探针

   如果要仅在探测成功时才开始向 Pod 发送请求流量，请指定就绪态探针。

6. master如何将pod调度到指定节点上

   该工作由scheduler组件完成，scheduler通过一系列复杂的算法算出pod的最佳node节点，可以通过**nodeselector、节点亲和性**来调度到指定的节点

7. 创建一个pod的流程

   客户端提交pod的配置信息到apiserver

   apiserver收到指令后通知controller-manager创建一个资源对象

   controller-manager通过apiserver将pod的配置信息存储到etcd

   scheduler检测到pod信息会开始调度预选，先过滤掉不符合pod资源配置要求的节点，然后开始调度调优，主要是挑选更合适运行pod的节点，然后将pod配置单发给节点的kubelet组件，kubelet运行pod

8. 删除一个pod会发生什么

   apiserver接受到删除指令，默认有30秒时间等待优雅退出，超过30秒状态会变为terminating，kubelet看到pod状态为terminating就开始关闭pod：

   pod从service的endpoint中移除；

   如果pod定义了一个停止前的钩子，其会在内部被调用，停止钩子一般定义了如何优雅的结束进程；

   进程被发送term信号kill -14

   当超过优雅退出时间后，pod中所有进程都被发送sigkill信号kill -9

9. 常见调度方式

   nodeselector：定向调度，将pod调度到某个标签的节点

   节点亲和性：

   污点和容忍：给节点打上污点后（master-test=test:NoSchedule），pod要想调度到该节点就必须配置容忍

9. 初始化容器？

   init容器必须先于应用容器启动，有多个init容器时要按顺序运行，并且只有前一个init容器运行成功后才会运行下一个init容器，直到所有init容器成功运行，k8s才会初始化pod的各种信息，并开始创建和运行应用容器。

10. Requests和Limits如何影响Pod的调度?

    当一个pod创建成功后，scheduler会为该pod选择一个合适的节点执行，调度器在调度时，首先要确保节点的上所有pod的cpu和内存总和不能超过该节点能提供给pod的cpu和内存的最大容量。

    在整个集群的资源分配调度中，可能某节点的实际资源使用量非常低，但是已运行 Pod 配置的总 Requests 值的总和非常高， 再加上需要调度的 Pod 的 Requests 值， 会超过该节点提供给 Pod 的资源容量上限， 这时 Kubernetes 任然不会将 Pod调度到该节点上。

## deploy、sts、ds、hpa

1. deployment升级/更新过程

   创建deployment时，系统创建了一个新的RS，RS创建了对应数量的pod副本；更新deployment时，**再创建一个新的RS**，然后根据更新策略**修改RS的副本数**，最后新的RS运行对应新的pod副本，旧的RS的pod副本数为0

2. 更新策略

   deployment默认rollingupdate，两个参数：maxUnavailable（最多不可用副本数）、maxSurge（最多可以新建多少个副本）

   daemonset默认ondelete

   statefulset默认rollingupdate，从最后一个pod开始更新，更新完成再继续更新下一个pod

3. k8s自动扩容机制

   使用hpa控制器根据pod的cpu和内存使用率进行自动pod扩缩容

## service、ingress

1. service

   类型：cluserip、nodeport、loadbalancer

   service要定义selector，否则不会自动创建endpoint；可以利用这点来通过service访问非k8s集群的服务

   headless service：不会被分配一个vip，而是以dns的形式暴露它所代理的pod，它所代理的pod的ip都会被绑定一个这样的记录pod-name.svc-name.ns-name.svc.cluster.local

2. 集群外部如何访问集群内部服务

   pod中定义hostNetwork: true，配合nodeSelector，把pod实例化在固定节点；使用nodeport类型的service；通过ingress访问

3. 简述ingress？

   k8s结合ingress对象和ingress-controller，实现不同url的访问请求转发到后端对应的pod

   ingress对象，其实就是一个反向代理的配置文件描述，它定义了某个域名的请求转发到指定的service

   ingress-controller直接将请求转发到service对应的后端pod上，跳过kube-proxy的转发功能

## 存储

1. k8s数据持久化的方式有哪些

   emptydir：挂载在pod的一个空目录，生命周期跟pod同步，适合容器之间临时共享文件

   hostpath：挂载宿主机的目录到pod

   configmap：存放容器的配置文件、环境变量、命令参数；secret：存放敏感信息比如密码、密钥、token等；以subpath或env挂载的话不支持热更新

   PV/PVC：PV是一块抽象的存储空间，生命周期独立于pod，PVC是申请PV的接口

2. PV和PVC的生命周期

   手动创建底层存储和PV；

   创建PVC，k8s根据PVC声明去寻找PV并与其绑定，如果找不到，PVC则无限处于pending状态

   PV一旦被绑定，就不会与其他PVC进行绑定

   在pod中像volume一样把PVC挂载到容器中的某个路径

   使用完毕后删除PVC，与其绑定的PV 会被标记为已释放，但还不能与其他PVC立刻绑定，k8s会根据回收策略对PV进行回收，只有PV的存储空间完成回收PV才能再次使用

3. PV的状态

   available：可用状态，还没跟pvc绑定

   bound：已与某个pvc绑定

   released：绑定的pvc已经删除，资源已经释放，但还没有被集群回收

   failed：自动资源回收失败

4. 存储供应模式

   静态模式：手工创建许多PV，在定义PV时需要将后端存储的特性进行设置。

   动态模式：无须手工创建PV，而是通过StorageClass的设置对后端存储进行描述，标记为某种类型。此时要求PVC对存储的类型进行声明，系统将自动完成PV的创建及与PVC的绑定。

5. k8s的CSI模型

   Kubernetes CSI是Kubernetes推出与容器对接的存储接口标准，存储提供方只需要基于标准接口进行存储插件的实现，就能使用Kubernetes的原生存储机制为容器提供存储服务。CSI使得存储提供方的代码能和Kubernetes代码彻底解耦，部署也与Kubernetes核心组件分离，显然，存储插件的开发由提供方自行维护，就能为Kubernetes用户提供更多的存储功能，也更加安全可靠。

   CSI包括CSI Controller和CSI Node：

   CSI Controller的主要功能是提供存储服务视角对存储资源和存储卷进行管理和操作。

   CSI Node的主要功能是对主机（Node）上的Volume进行管理和操作。

   

   
