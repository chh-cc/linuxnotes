## docker

1. docker用了linux什么技术

   namespace：linux内核对进程进行资源的隔离，通过隔离网络、PID、文件系统、主机名等，实现同一个宿主机的容器之间互不干扰

   cgroup：linux内核对进程的使用资源上限进行限制，如cpu、内存、网络带宽等

2. docker网络模型

   bridge：网桥，默认类型，会自动在主机上创建一个docker0虚拟网桥

   host：和宿主机共享一个网络namespace

   container：和别的容器共享一个网络namespace

   none：不参与网络通信

3. 本地镜像文件存放在哪里

   /var/lib/docker/目录下，其中container目录存放容器信息，graph目录存放镜像信息，aufs目录下存放具体的镜像底层文件

4. 一个容器可以运行多个进程吗

   一般不推荐运行多个进程，如果有类似需求可以通过额外的进程管理机制比如supervisor

5. 说下常用的docker命令

   制作镜像docker build

   镜像打标签docker tag

   上传拉取镜像docker push/pull

   运行容器docker run

   查看容器日志docker logs

   进入容器docker exec

6. 让容器随着docker服务启动而自动启动

   第一次启动容器：

   docker run --restart=always

   已经运行的容器：

   修改容器配置文件的RestartPolicy参数或docker update --restart=always

7. 指定容器的端口映射

   docker run -p 宿主机端口:容器端口

8. docker注入环境变量

   运行容器时使用-e ..=..或者在dockerfile写入环境变量ENV ... ...

9. 容器异常排查

   通过docker logs查看容器日志

   docker exec进入容器内部查看情况（可以进入容器的情况下）

   检查宿主机的运行环境、资源情况

   重启容器docker restart（如果容器挂了）

## k8s

### 架构、组件

1. docker和k8s关系？

   docker提供容器生命周期管理，k8s用于关联和编排多主机上运行容器

2. k8s有哪些组件及功能？

   k8s分控制平面和工作节点，控制平面包括apiserver、controller-manager、scheduler、etcd

   apiserver：资源增删查改的唯一入口，各个组件通过apiserver通信（只有apiserver和etcd直接打交道）：各个组件通过apiserver将信息存入etcd中，当需要获取这些信息时再从apiserver的REST接口获取（get、list、watch）

   controller-manager：负责管理各种控制器，保证系统中的状态和期望状态一致

   scheduler：通过apiserver的watch接口监听新建pod信息，把pod调度到一个或一批合适的节点

   etcd：保存整个集群的状态

### pod

1. 创建一个 Pod 时，它会经历哪些状态？

   Pending（等待中）：创建一个 Pod 时，它的初始状态将是“等待中”。这意味着 Kubernetes 正在尝试为该 Pod 找一个可用的节点

   ContainerCreating（创建容器中）：一旦 Kubernetes 将 Pod 调度到节点上，它将开始为 Pod 中的每个容器创建容器运行时环境

   Running（运行中）：当所有容器都准备就绪并开始运行时，Pod 的状态将变为“运行中”。此时，所有容器都已经成功启动并在运行中。

   ---

   Succeeded（已完成）：一旦 Pod 中的所有容器都已经完成它们的任务并退出，Pod 的状态将变为“已完成”。这通常发生在批处理作业或一次性任务中。

2. pod重启策略？

   always：总是自动重启

   onfailed：异常退出才重启

   never：从不重启

   不同控制器的重启策略：deployment、statefulset、daemonset都要always；job为onfailed或never

3. pod的健康检测机制

   三种检测探针：

   livenessprobe存活探针：用于判断容器是否running，如果检测到不健康则**杀掉容器**然后**重启**

   readnessprobe就绪探针：判断容器是否ready，如果检测失败则把pod**从endpoint中移除**，不让外部流量进入pod

   startupprobe启动探针：配置了startupprobe会**先禁用其他探针**，直到检测成功为止，成功后就不再探测（容器启动时间较长（超出 `initialDelaySeconds + failureThreshold × periodSeconds`）则用启动探针）

   探针的三种检测类型：

   ExecAction：在容器内执行一个命令，如果返回值为0，则认为容器健康

   TCPSocketAction：对指定端口的容器IP进行TCP检查，如果端口打开则认为容器健康

   HttpGetAction：对指定端口和路径的容器IP进行HTTP Get请求，如果状态码在200~400之间则认为容器健康

4. 创建一个pod的流程

   客户端提交一个pod的配置信息到apiserver，apiserver收到指令后通知controller-manager创建资源对象

   controller-manager通过apiserver把pod配置信息存入etcd中

   scheduler检测到pod信息会把pod调度到合适的节点，把pod配置单发给节点的kubelet组件

   kubelet运行pod

5. 删除一个pod的流程

   apiserver收到delete pod指令后，pod状态变为“terminating“

   k8s向pod发送SIGTERM信号，信号会要求容器尽快停止正在运行的进程，如果在默认的grace period内容器没有停止，则k8s会发送SIGKILL信号，强制关闭容器

   在容器被终止之后，Kubernetes 控制平面会将该 Pod 的状态设置为“Terminated”并从集群中移除该 Pod。

6. Requests和Limits如何影响Pod的调度?

   Requests：Pod 的 CPU 和内存请求是一个容器或多个容器请求的总量。如果没有足够的请求资源可用于满足 Pod 的要求，那么该 Pod 可能会排队等待。如果等待时间过长，Pod 将会被调度器标记为未调度状态并且不会启动。因此，Pod 的请求资源限制可以影响 Pod 是否能够被调度。

   Limits：Pod 的 CPU 和内存限制是容器可以使用的资源的最大值。如果容器使用的资源超过其限制，则 Kubernetes 会将容器标记为 OutOfMemory 或 OutOfCPU，然后进行相应的处理。如果所有容器都标记为 OutOfMemory 或 OutOfCPU，Pod 将会被重启或重新调度，以确保容器能够正常运行。因此，Pod 的资源限制可以影响容器的稳定性和可用性。

   

## 控制器

1. 如何控制滚动更新的过程

   maxSurge：此参数控制滚动更新的过程中，最多可以新建多少个副本数

   maxUnavailable：此参数控制滚动更新的过程中，最多有多少个副本不可用

2. deployment升级/更新过程

   更新delpoyment时，会创建新的RS，用于管理更新后的pod

   新的RS创建后，会逐步替换旧的pod，先创建新的pod，再删除旧的pod，直到旧版pod副本数为0，新版副本数稳定，停止滚动更新

   在更新过程中，k8s会不断检测更新状态，确保所有的pod都已经更新完成

3. 版本回滚命令

   kubectl rollout undo deploy xxx [--to-reversion=?]

4. statefulset为什么要用headless service

   statefulset管理的是有状态服务，不需要通过service反向代理到一组pod，使用headless service就可以通过pod名称.servicename.namespace来访问一指定的pod

   

## service

1. 集群外部如何访问集群内部服务

   pod配置hostNetwork: true参数，可以共享宿主机的ip，直接通过宿主机ip访问

   通过nodeport类型的serivce访问pod

   通过ingress访问pod
