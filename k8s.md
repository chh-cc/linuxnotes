## k8s

## docker

1. docker用了linux什么技术

   namespace：linux内核中对进程进行资源的隔离，通过隔离网络、PID、系统信号量、文件挂载系统、主机名和域名，来实现同一个宿主机的容器互不干扰

   cgroup：linux内核中对进程使用资源的上限进行限制，包括CPU、内存、磁盘读写速率、网络带宽等

2. docker网络模型

   bridge：网桥，默认类型，会自动在主机上创建一个 `docker0` 虚拟网桥

   host：和宿主机共享一个网络namespace

   container：和别的容器共享一个网络namespace

   none：不参与网络通信

3. 本地的镜像文件都存放在哪里

   /var/lib/docker/目录下，其中container目录存放容器信息，graph目录存放镜像信息，aufs目录下存放具体的镜像底层文件

4. 如何临时退出一个正在交互的容器的终端，而不终止它？

   ctrl+p+q

5. 可以在一个容器中同时运行多个应用进程吗？

   一般不推荐一个容器运行多个进程，如果有类似需求可以通过额外的进程管理机制比如supervisord
   
6. docker常用命令

   docker images/ps

   docker pull/push

   docker build

   docker tag

   docker run

   docker cp 宿主机和容器互相拷贝文件

   docker exec 进入容器

   docker logs

   docker start/stop

7. 让容器随着 Docker 服务启动而自动启动？

   docker run --restart=always

   创建容器后则通过修改容器配置文件的RestartPolicy参数或docker update更新--restart参数

8. 指定容器的端口映射

   docker run -p 宿主机端口:容器端口

   创建容器后则通过修改容器配置文件

9. 容器异常排查

   通过启动容器返回的错误提示排查

   通过docker logs -f获取日志排查

10. docker注入环境变量

   运行容器时使用-e ..=..或者在dockerfile写入环境变量ENV ... ...

11. namespace不能隔离什么

    磁盘IO、时间

12. docker有几种状态

    starting运行、exited退出、paused暂停、healthy健康、unhealthy不健康

## 架构和组件

1. k8s与docker的关系？

   docker提供了容器的生命周期管理和镜像构建运行时容器，主要优点是将应用和所需设置、依赖打包到一个容器中，提高可移植性。k8s用于关联和编排多主机上运行容器。

2. k8s有哪些组件及功能？

   k8s划分为控制平面和工作节点，控制平面包括apiserver、controller-manager、scheduler、etcd，这些组件实现了整个集群的安全控制、故障恢复、自动扩缩、Pod调度等。

   apiserver：**资源增删查改的唯一入口，各个组件通过apiserver进行通信**：各个组件通过apisever**将信息存入etcd中**，当需要获取这些数据时再通过apiserver提供的**REST接口**（用get、list、watch）去请求，从而实现各个组件的通信。

   controller-manager：**维护集群的状态**，执行各种控制器，故障检测、自动扩展、滚动更新、保证pod数量达到期望值等

   scheduler：通过API Server的Watch接口监听到新建Pod副本的信息后，把pod调度到一个或一批合适的节点

   etcd：保存了整个集群的状态

   

   kubelet：负责节点上**pod的**创建、修改、删除等**全生命周期管理**，定期汇报node状态给apiserver

   kube-proxy：**负责转发**，一旦发现service关联的pod信息发生改变，就会创建ipvs规则，完成serive对后端pod的转发

   容器运行时：负责镜像管理和pod容器的真正运行，比如docker
   
3. 升级kubadm部署的集群

   升级前必须备份所有组件及数据，例如etcd

   不要跨多个小版本进行升级

   1.升级管理节点

   2.升级工作节点

4. 证书续签

   **容易过期的是ca证书派发出来的证书文件**，工作节点证书有效期限都是1年


## pod

1. kube-proxy ipvs原理

   创建了service后会在宿主机生成一个虚拟网卡（kube-ipvs0），并给它分配一个service vip，kube-proxy为这个地址设置ipvs虚拟主机，并设置这几个虚拟主机使用轮询模式来作为负载策略。这时候任何发往这个IP地址的请求都会转发到某一个后端pod。

   ⼆者有着本质的差别：iptables是为防⽕墙⽽设计的；IPVS则专门⽤于⾼性能负载均衡，并使⽤更⾼效的数据结构（Hash表），允许⼏乎⽆限的规模扩张。

2. pod有哪些状态

   pending：pod的yaml文件已经提交给k8s，api对象已经被保存在etcd中，但pod可能因为一些原因无法顺利创建，比如**pod调度失败**/pvc有问题

   failed：pod至少有一个容器不正常退出

   unknown：意味pod的状态不能被kubelet汇报给apiserver，可能是master和node之间通信出了问题

   succeeded：pod内所有容器正常退出没有重启

   running：pod被调度成功，并且至少有一个容器正常运行

3. pod重启策略

   always：容器失效时，kubelet自动重启该容器

   onfailed：容器异常退出时，kubelet自动重启该容器

   never：都不重启

   不同控制器的重启策略：deployment、statefulset、daemonset都要always；job为onfailed或never

4. pod的健康检测机制

   livenessprobe存活探针：用于判断容器是否running，如果检测到不健康则**杀掉容器**然后**重启**

   readnessprobe就绪探针：判断容器是否ready，如果检测失败则把pod**从endpoint中移除**，不然外部流量进入pod

   startupprobe启动探针：配置了startupprobe会**先禁用其他探针**，直到检测成功为止，成功后就不再探测，适合启动慢的程序

   

   ExecAction：在容器内执行一个命令，如果返回值为0，则认为容器健康

   TCPSocketAction：对指定端口的容器IP进行TCP检查，如果端口打开则认为容器健康

   HttpGetAction：对指定端口和路径的容器IP进行HTTP Get请求，如果状态码在200~400之间则认为容器健康

5. 如何选择探针

   容器启动时间较长（超出 `initialDelaySeconds + failureThreshold × periodSeconds`）则用启动探针

6. master如何将pod调度到指定节点上

   该工作由scheduler组件完成，scheduler通过一系列复杂的算法算出pod的最佳node节点，可以通过**nodeselector、节点亲和性**来调度到指定的节点

7. 创建一个pod的流程

   客户端提交pod的配置信息到apiserver

   apiserver收到指令后通知controller-manager创建一个资源对象

   controller-manager通过apiserver将pod的配置信息存储到etcd

   scheduler检测到pod信息会开始调度预选，先过滤掉不符合pod资源配置要求的节点，然后开始调度调优，主要是挑选更合适运行pod的节点，然后将pod配置单发给节点的kubelet组件，kubelet运行pod

8. 删除一个pod会发生什么

   delete pod会有两个平行的过程：

   网络规则：apiserver收到delete pod后pod状态变为terminating，endpoint移除该pod的ip流量就不会进到该pod

   删除pod：kubelet向容器发送SIGTERM，如果容器没有配置优雅关闭进程，就会立刻退出；如果容器配置了，但是在默认的30秒内还没退出，kubelet就会发送SIGKILL强制退出

9. 常见调度方式

   nodeselector：定向调度，将pod调度到某个标签的节点

   亲和性：节点亲和性：pod调度到指定标签的节点，pod反亲和性：多副本的情况下把pod分散到不同节点

   污点和容忍：给节点打上污点后（master-test=test:NoSchedule），pod要想调度到该节点就必须配置容忍

9. 初始化容器？

   init容器必须先于应用容器启动，有多个init容器时要按顺序运行，并且只有前一个init容器运行成功后才会运行下一个init容器，直到所有init容器成功运行，k8s才会初始化pod的各种信息，并开始创建和运行应用容器。

10. Requests和Limits如何影响Pod的调度?

    当一个pod创建成功后，scheduler会为该pod选择一个合适的节点执行，调度器在调度时，首先要确保节点的上所有pod的cpu和内存总和不能超过该节点能提供给pod的cpu和内存的最大容量。

    在整个集群的资源分配调度中，可能某节点的实际资源使用量非常低，但是已运行 Pod 配置的总 Requests 值的总和非常高， 再加上需要调度的 Pod 的 Requests 值， 会超过该节点提供给 Pod 的资源容量上限， 这时 Kubernetes 任然不会将 Pod调度到该节点上。

## deploy、sts、ds、hpa

1. ReplicaSet作用？

   RS会尽可能保证pod以我们指定的副本数运行，假如想要运行10个pod，如果这时候有4个pod出了故障异常退出，那么RS会自动创建新的4个pod来取代故障的pod

2. 更新策略

   deployment默认rollingupdate

   statefulset默认rollingupdate，从最后一个pod开始更新，更新完成再继续更新下一个pod

   daemonset默认ondelete

3. 如何控制滚动更新过程

   maxSurge：此参数控制滚动更新过程中，最多可以创建多少副本

   maxUnavailable：此参数控制滚动更新过程中，最多不可用副本数

4. deployment升级/更新过程

   更新deployment时，**会创建新的RS，并将其扩容**，控制副本数在最大运行峰值比例，达到比例后不再扩容，直到杀死足够多的旧版pod

   接下来**对旧版本RS进行缩容操作**，控制去除Pod副本数量**满足最大不可用比例**，达到比例后不会再继续删除旧版Pod，直到创建到足够多的新版Pod

   此为一轮更新，DM不断的进行滚动更新上述操作，直到旧版Pod副本数为0，新版副本数稳定，停止滚动更新。

5. 版本回滚相关命令

   kubectl rollout undo deploy xxx [--to-reversion=?]

6. k8s自动扩容机制

   使用hpa控制器根据pod的cpu和内存使用率进行自动pod扩缩容

7. statefulset为什么要用headless service

   statefulset管理有状态服务，**不需要通过service反向代理到一组pod**，statefulset为每个pod做了唯一的编号，headless service不分配cluster ip，使用headless service就可以通过pod名称+序号.service名称.namespace名称来访问一个指定的pod

## service、ingress

1. service

   类型：cluserip、nodeport、loadbalancer

   service要定义selector，否则不会自动创建endpoint；可以利用这点来通过service访问非k8s集群的服务

   headless service：不会被分配一个vip，而是以dns的形式暴露它所代理的pod，它所代理的pod的ip都会被绑定一个这样的记录pod-name.svc-name.ns-name.svc.cluster.local

2. 集群外部如何访问集群内部服务

   pod中定义hostNetwork: true，配合nodeSelector，把pod实例化在固定节点；使用nodeport类型的service；通过ingress访问

3. 简述ingress？

   k8s结合ingress对象和ingress-controller，实现不同url的访问请求转发到后端对应的pod

   ingress对象，其实就是**一个反向代理的配置文件描述**，它定义了某个域名的请求转发到指定的service

   ingress-controller直接**将请求转发到service对应的后端pod**上，跳过kube-proxy的转发功能
   
4. 灰度发布

   创建一个金丝雀版本ingress，配置nginx.ingress.kubernetes.io/canary: "true"和nginx.ingress.kubernetes.io/canary-by-header，当请求中的hearder key和value和ingess匹配时请求流量就会分配到canary入口

## 网络

1. k8s网络模型

   Kubernetes网络模型中每个Pod都拥有一个独立的IP地址，并假定所有Pod都在一个可以直接连通的、扁平的网络空间中。所以不管它们是否运行在同一个Node（宿主机）中，都要求它们可以直接通过对方的IP进行访问。

2. 同一pod内多容器间通讯？

   同一pod内容器共享同一个网络命名空间，通过localhost加端口通信

3. flannel原理

   flannel为每一个node分配ip子网，在node创建一个名为flannel.1的虚拟网卡，为容器配置名为docker0的网桥

   源容器向目标容器发送数据，数据首先发送给docker0网桥

   docker0网桥将其转发给flannel.1虚拟网卡处理

   flannel.1收到后对数据进行封装，转发给宿主机eth0

   对封装的数据包再封装（为了在物理网络上传输），转发给目标容器node的eth0

    目标容器宿主机eth0收到后进行拆分，然后转发给flannel.1，flannel.1转发给docker0，最后数据到达目标容器

1. k8s的CNI模型？

   k8s的扁平网络空间是由CNI插件建立的，常见的有flannel和calico

   flannel能协助k8s给每一个节点的容器都分配不相互冲突的IP地址，它能在这些ip地址建立一个覆盖网络，容器可以在这个网络中通信。

2. k8s的网络策略？

   网络策略的主要功能是对pod间的网络通信进行限制和准入控制，设置方式是将pod的label作为查询条件，设置允许或禁止访问的pod列表；默认所有pod没有隔离；网络策略功能由第三方网络插件提供，如calico

## 安全

1. 认证方式

   client证书（k8s各种组件访问apiserver）和jwt token（pod访问apiserver）

2. k8s内置角色

   - **cluster-admin**  超级管理员，对集群所有权限（在部署dashboard的时候，先创建sa，然后将sa绑定到角色cluster-admin，最后获取到token，这就使用了内置的cluster-admin ）
   - **admin**   主要用于授权命名空间所有读写权限
   - **edit**   允许对命名空间大多数对象读写操作，不允许查看或者修改角色、角色绑定。
   - **view** 允许对命名空间大多数对象只读权限，不允许查看角色、角色绑定和Secret
   
3. 授权用户访问k8s资源

   1.根据ca签发用户的证书

   2.根据用户证书绑定角色（根据需求配置只读/读写指定资源权限）并生成kubeconfig文件

   3.将kubeconfig文件放到用户目录的.kube目录，切换上下文并测试权限

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

## 日志

1. 怎么收集k8s微服务日志

   为了轻量级收集日志，可以使用filebeat来收集并将日志发送给logstash，filebeat以边车模式伴随微服务进行部署

2. 监控什么日志

   有几类比较关心的日志：

   ingress日志：里面有从公网域名进来的访问记录

   k8s事件日志：kubectl describe pods中的Events:内容，有容器健康检查失败重新部署，或者pod新建、删除等事件

   业务程序的日志：比如java中常用的log4j套件输出的日志和kubectl logs命令查看的日志





















