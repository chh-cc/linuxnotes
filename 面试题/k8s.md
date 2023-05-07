## docker

1. docker用了linux什么技术

   namespace：linux内核对进程进行资源的隔离，通过隔离网络、PID、文件系统、主机名等，实现同一个宿主机的容器之间互不干扰

   cgroup：linux内核对进程的使用资源上限进行限制，如cpu、内存、网络带宽等

2. 宿主机能看到容器的进程吗

   能

3. docker网络模型

   bridge：默认类型，给每个容器分配ip，并将容器连接到docker0的虚拟网桥

   host：和宿主机共享一个网络namespace

   container：和别的容器共享一个网络namespace

   none：禁用网络

4. 本地镜像文件存放在哪里

   /var/lib/docker/目录下，其中container目录存放容器信息，graph目录存放镜像信息，aufs目录下存放具体的镜像底层文件

5. 一个容器可以运行多个进程吗

   一般不推荐运行多个进程，如果有类似需求可以通过额外的进程管理机制比如supervisor

6. 说下常用的docker命令

   制作镜像docker build

   镜像打标签docker tag

   上传拉取镜像docker push/pull

   运行容器docker run

   查看容器日志docker logs

   进入容器docker exec

7. 让容器随着docker服务启动而自动启动

   第一次启动容器：

   docker run --restart=always

   已经运行的容器：

   修改容器配置文件的RestartPolicy参数或docker update --restart=always

8. 指定容器的端口映射

   docker run -p 宿主机端口:容器端口

9. docker注入环境变量

   运行容器时使用-e ..=..或者在dockerfile写入环境变量ENV ... ...

10. 容器异常排查

   通过docker logs查看容器日志

   docker exec进入容器内部查看情况（可以进入容器的情况下）

   检查宿主机的运行环境、资源情况

   重启容器docker restart（如果容器挂了）

## k8s

### 架构、组件

1. docker和k8s关系？

   docker提供容器生命周期管理，k8s用于关联和编排多主机上运行容器

2. k8s有哪些组件及功能？

   k8s分控制平面和工作节点，控制平面包括apiserver、controller-manager、scheduler、etcd，工作节点包括kubelet、kube-proxy

   apiserver：资源增删查改的唯一入口，各个组件通过apiserver通信（只有apiserver和etcd直接打交道）：各个组件通过apiserver将信息存入etcd中，当需要获取这些信息时再从apiserver的REST接口获取（get、list、watch）

   controller-manager：监控Kubernetes资源对象的状态，保证系统中的状态和期望状态一致

   scheduler：通过apiserver的watch接口监听新建pod信息，把pod调度到一个或一批合适的节点

   etcd：保存整个集群的状态

   ---

   kubelet：负责pod的全生命周期的管理，定期汇报节点状态给apiserver

   kube-proxy：负责转发，一旦发现service关联的pod信息发生变化，就会创建ipvs规则，完成service对后端pod的转发

   容器运行时：负责镜像管理和容器的真正运行，比如docker

3. 简述k8s的调度有哪些方式

   基于cpu和内存资源的静态/动态调度：为每个pod指定需要使用的cpu和内存资源。基于这些资源的需求，k8s会自动将pod调度到具有足够资源的节点上

   基于节点标签的调度

   基于节点亲和性反亲和性的调度

4. 简述kube-proxy的工作模式和原理

   iptables：

   Kube-proxy为service后端的每个Pod创建对应的iptables规则，直接将发向Cluster IP的请求重定向到一个Pod IP，但不能提供灵活的LB策略，当后端Pod不可用时也无法进行重试

   ipvs：

   和iptables类似，kube-proxy监控Pod的变化并创建相应的ipvs rules。ipvs也是在kernel模式下通过netfilter实现的，但采用了hash table来存储规则，因此在规则较多的情况下，Ipvs相对iptables转发效率更高。除此以外，ipvs支持更多的LB算法

### pod

1. 创建一个 Pod 时，它会经历哪些状态？

   Pending（等待中）：创建一个 Pod 时，它的初始状态将是“等待中”。这意味着 Kubernetes 正在尝试为该 Pod 找一个可用的节点

   ContainerCreating（创建容器中）：一旦 Kubernetes 将 Pod 调度到节点上，它将开始为 Pod 中的每个容器创建容器运行时环境

   Running（运行中）：当所有容器都准备就绪并开始运行时，Pod 的状态将变为“运行中”。此时，所有容器都已经成功启动并在运行中。

   ---

   Succeeded（已完成）：一旦 Pod 中的所有容器都已经完成它们的任务并退出，Pod 的状态将变为“已完成”。这通常发生在批处理作业或一次性任务中。

2. 节点宕机后，简述pods驱逐流程

   节点宕机一段时间后会被标记为unkown状态，并且节点生命周期控制器会自动创建代表状况的污点，用于防止调度器调度 pod 到该节点。

   在节点被认定为不可用状态到删除节点上的 Pod 之间是有一段时间的，这段时间被称为容忍度，如果不配置的话默认是300秒

   当到了删除 Pod 时，污点管理器会创建污点标记事件，然后驱逐 pod

   deployment 控制器在监听到 Pod 被驱逐后会创建一个新的 Pod 出来，但是 Statefulset 控制器并不会创建出新的 Pod，原因是因为它可能会违反 StatefulSet 固有的至多一个的语义，可能出现具有相同身份的多个成员，这将可能是灾难性的，并且可能导致数据丢失。 

3. pod重启策略？

   always：总是自动重启

   onfailed：异常退出才重启

   never：从不重启

   不同控制器的重启策略：deployment、statefulset、daemonset都要always；job为onfailed或never

4. pod的健康检测机制

   三种检测探针：

   livenessprobe存活探针：用于判断容器是否running，如果检测到不健康则**杀掉容器**然后**重启**

   readnessprobe就绪探针：判断容器是否ready，如果检测失败则把pod**从endpoint中移除**，不让外部流量进入pod

   startupprobe启动探针：配置了startupprobe会**先禁用其他探针**，直到检测成功为止，成功后就不再探测（容器启动时间较长（超出 `initialDelaySeconds + failureThreshold × periodSeconds`）则用启动探针）

   探针的三种检测类型：

   ExecAction（命令）：在容器内执行一个命令，如果返回值为0，则认为容器健康

   TCPSocketAction（tcp请求）：对指定端口的容器IP进行TCP检查，如果端口打开则认为容器健康

   HttpGetAction（http请求）：对指定端口和路径的容器IP进行HTTP Get请求，如果状态码在200~400之间则认为容器健康

5. 创建一个pod的流程

   客户端提交一个pod的配置信息到apiserver，apiserver收到指令后通知controller-manager创建资源对象

   controller-manager通过apiserver把pod配置信息存入etcd中

   scheduler检测到pod信息会把pod调度到合适的节点，把pod配置单发给节点的kubelet组件

   kubelet指示当前节点上的Container Runtime运行对应的容器

   Container Runtime下载镜像并启动容器

6. 删除一个pod的流程

   从service删除pod：如果pod正在被service使用，k8s会将其从service中删除，确保不再将流量发送到该pod

   发送SIGTERM信号：k8s向pod发送SIGTERM信号，让容器开始优雅停止

   发送SIGKILL信号：如果在默认的grace period内容器没有停止，则k8s会发送SIGKILL信号，强制关闭容器并释放其资源

   清理资源：k8s将释放pod使用的所有资源，包括节点上的网络、存储、内存等

7. Requests和Limits如何影响Pod的调度?

   Requests：Pod 的 CPU 和内存请求是一个容器或多个容器请求的总量。如果没有足够的请求资源可用于满足 Pod 的要求，那么该 Pod 可能会排队等待。如果等待时间过长，Pod 将会被调度器标记为未调度状态并且不会启动。因此，Pod 的请求资源限制可以影响 Pod 是否能够被调度。

   Limits：Pod 的 CPU 和内存限制是容器可以使用的资源的最大值。如果容器使用的资源超过其限制，则 Kubernetes 会将容器标记为 OutOfMemory 或 OutOfCPU，然后进行相应的处理。如果所有容器都标记为 OutOfMemory 或 OutOfCPU，Pod 将会被重启或重新调度，以确保容器能够正常运行。因此，Pod 的资源限制可以影响容器的稳定性和可用性。

   

## 控制器

1. deployment和rs的关系？

   创建deployment时会自动创建一个RS，deploy不是直接管理pod，而是通过管理RS来管理pod，通过修改RS的副本数和属性来实现扩缩副本和滚动更新

2. 如何控制滚动更新的过程

   maxSurge：此参数控制滚动更新的过程中，最多可以新建多少个副本数

   maxUnavailable：此参数控制滚动更新的过程中，最多有多少个副本不可用

3. deployment升级/更新过程

   更新delpoyment时，会创建新的RS，用于管理更新后的pod

   新的RS创建后，会逐步替换旧的pod，先创建新的pod，再删除旧的pod，直到旧版pod副本数为0，新版副本数稳定，停止滚动更新

   在更新过程中，k8s会不断检测更新状态，确保所有的pod都已经更新完成

4. 版本回滚命令

   kubectl rollout undo deploy xxx [--to-reversion=?]

5. statefulset为什么要用headless service

   statefulset管理的是有状态服务，不需要通过service反向代理到一组pod，使用headless service就可以通过pod名称.servicename.namespace来访问一指定的pod

6. k8s自动伸缩方式有哪些

   水平自动伸缩HPA：根据cpu、内存利用率等指标，动态增加或减少pod数量，以满足应用的负载需求

   垂直自动伸缩VPA：根据容器内部资源使用情况，动态调整容器的cpu和内存请求量

   集群自动伸缩：根据集群节点的资源利用率，动态增加或减少节点的数量

   

## 通信

1. 简单说下k8s集群内外如何互通

   使用service：使用nodeport类型或loadbalancer的service，这样可以通过service的ip和端口调用服务

   使用ingress：ingress controller可以将外部流量路由到k8s集群中特定的服务

   pod配置hostNetwork: true参数，可以共享宿主机的ip，直接通过宿主机ip访问

2. service和ep的关系

   创建一个service的同时会创建一个同名的endpoint，ep记录了svc关联的一组pod的ip和端口

3. 简述ingress？

   k8s结合ingress对象和ingress-controller，实现不同的url请求转发到对应的pod应用服务

   ingress对象其实就是一个反向代理的配置文件描述，定义了某个域名请求转发到对应的service

   ingress-controller是真正工作的组件，将请求转发到service对应的后端pod，跳过kube-proxy的转发功能

4. ingress如何配置https流量

   ingress可以通过tls证书来配置https，使用secret对象存储tls证书，然后在ingress规则中指定要使用的证书

## 存储

1. k8s数据持久化的方式有哪些

   本地存储：如emptydir、hostpath，数据保存在集群特定的节点上，不能随应用漂移

   网络存储：如ceph、glusternfs等，需要将存储服务挂载到本地使用

   PV/PVC：PV是一块抽象的存储空间，生命周期独立于pod，PVC将数据卷抽象成一个独立于pod的对象，供k8s负载挂载使用

2. pv和pvc的生命周期（如何使用pvc）

   手动创建底层存储和PV

   创建PVC，k8s根据PVC声明去寻找PV并与其绑定，如果找不到，PVC则无限处于pending状态

   PV一旦被绑定，就不会与其他PVC进行绑定

   在pod中像volume一样把PVC挂载到容器中的某个路径

   使用完毕后删除PVC，与其绑定的PV 会被标记为已释放，但还不能与其他PVC立刻绑定，k8s会根据回收策略对PV进行回收，只有PV的存储空间完成回收PV才能再次使用

3. pv与pvc的绑定关系？

   pv与pvc一一对应，pvc只有绑定了pv后才能被使用

4. pv有哪些状态

   available：可用状态，还没跟pvc绑定

   bound：已与某个pvc绑定

   released：pv已经释放了pvc，可以重新绑定给其他pvc使用

   Failed：PV 操作失败或者 PV 所在的存储出现了故障，因此 PV 不能再被使用，需要手动修复或删除。

5. 如何在pod中使用存储

   先定义一个volume，然后将volume mount到pod的某个目录，可以使用emtpydir、hostpath、pvc等类型的volume

## 安全

1. 认证方式？

   用户访问k8s集群需要先通过认证，认证方式很多种，一般为令牌认证和ssl认证

   令牌认证：k8s集群会为每个用户颁发一个令牌，用户可以使用该令牌来访问k8s集群

   ssl认证：客户端和服务端互相确认身份

2. 授权用户访问k8s集群

   1.根据ca签发用户的证书

   2.根据用户证书绑定角色（根据需求配置只读/读写指定资源权限）并生成kubeconfig文件

   3.将kubeconfig文件放到用户目录的.kube目录，切换上下文并测试权限

## 网络

1. 简述你知道的几种CNI网络插件，并详述其工作原理

   calico：

   flannel：

   Flannel实质上是一种“覆盖网络(overlay network)”，也就是将TCP数据包装在另一种网络包里面进行路由转发和通信。

   数据从源容器中发出后，经由所在主机的docker0虚拟网卡转发到flannel0虚拟网卡

   Flannel通过Etcd服务维护了一张节点间的路由表，详细记录了各节点子网网段 

   源主机的flanneld服务将原本的数据内容UDP封装后根据自己的路由表投递给目的节点的flanneld服务，数据到达以后被解包，然后直接进入目的节点的flannel0虚拟网卡，然后被转发到目的主机的docker0虚拟网卡，最后就像本机容器通信一下的有docker0路由到达目标容器

2. k8s的网络模型

   k8s网络模型中，每个pod都有一个独立的ip地址，并且假设每个pod都在一个直通的、扁平的网络空间，所以不管pod有没有在同一个宿主机，都可以通过pod的ip直接访问

   k8s的扁平网络空间是由CNI插件建立的，常见的有flannel和calico

3. 同一个pod内多容器间的通信？

   同一个pod共享网络命名空间，通过localhost加端口通信

4. 同一个节点，pod之间通信会走网络插件吗

   不会

## 上线

1. devops怎么做的？

   可以用jenkins+gitlab+sonarqube+maven+nexus+harbor+k8s 实现一套完整的 devops 系统，开发提交代码到 github -->Jenkins 检测到代码更新 --> 调用 api 在k8s 中创建 jenkins slave pod--> jenkins slave 拉取代码 -->通过 maven 把拉取的代码进行构建成jar包-->上传代码到 sonarqube 进行代码扫描-->基于jar包构建docker镜像--> 把镜像上传到 harbor 仓库 --> 基于镜像部署到应用开发环境 --> 部署应用到测试环境--> 部署应用到生产环境

## 排障

1. clusterip访问不通

   一般是service没有正确创建或者service没有正确关联到后端pod，可以用kubectl describe svc查看svc的详细信息
   
1. pod启动失败

   一般查看系统资源是否满足，然后就是查看pod日志看看原因
   
   kubectl describe查看pod失败的原因
   
   还有就是看组件日志，apiserver等组件日志，有没有异常

