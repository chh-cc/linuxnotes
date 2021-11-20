# Docker

## 简介

go语言开发的

通过部署容器方式实现，每个容器之间互相隔离，每个容器有自己的文件系统 ，容器之间进程不会相互影响，能区分计算资源。

**docker可以实现虚拟机隔离应用环境的功能，并且开销比虚拟机小**。

把应用程序（1.jar）根据某个依赖的（java）基础镜像，生成一个应用程序镜像，这个应用程序镜像可以在任何部署了docker环境的机器上运行

docker容器镜像官网：

```text
dockerhub.com
download.docker.com
```

docker国内加速镜像站：

```text
阿里云
清华大学镜像站：mirrors.tuna.tsinghua.edu.cn
```

### 用途

1.提供一次性的环境。比如，本地测试他人的软件、持续集成的时候提供单元测试和构建的环境。

2.提供弹性的云服务。因为Docker容器可以随开随关，很适合动态扩容和缩容。

3.组建微服务架构。通过多个容器，一台机器可以跑多个服务，因此在本机就可以模拟出微服务架构。

### Docker应用建议

1）不要在容器中存储数据 – 容器可能被停止，销毁，或替换。一个运行在容器中的程序版本1.0，应该很容易被1.1的版本替换且不影响或损失数据。有鉴于此，如果你需要存储数据，**请存在卷中**，并且注意如果两个容器在同一个卷上写数据会导致崩溃。确保你的应用被设计成在共享数据存储上写入。

 2）不要将你的应用发布两份 – 一些人将容器视为虚拟机。他们中的大多数倾向于认为他们应该在现有的运行容器里发布自己的应用。在开发阶段这样是对的，此时你需要不断地部署与调试；但对于质量保证与生产中的一个连续部署的管道，你的应用本该成为镜像的一部分。记住：容器应该保持不变。

 3）不要创建超大镜像 – 一个超大镜像只会难以分发。确保你仅有运行你应用/进程的必需的文件和库。不要安装不必要的包或在创建中运行更新（yum更新）。

 4）不要使用单层镜像 – 要对分层文件系统有更合理的使用，始终为你的操作系统创建你自己的基础镜像层，另外一层为安全和用户定义，一层为库的安装，一层为配置，最后一层为应用。这将易于重建和管理一个镜像，也易于分发。

 5）不要为运行中的容器创建镜像 – 换言之，**不要使用“docker commit”命令来创建镜像**。这种创建镜像的方法是不可重现的也不能版本化，应该彻底避免。始终使用Dockerfile或任何其他的可完全重现的S2I（源至镜像）方法。

 6）不要只使用“最新”标签 – 最新标签就像Maven用户的“快照”。标签是被鼓励使用的，尤其是当你有一个分层的文件系统。你总不希望当你2个月之后创建镜像时，惊讶地发现你的应用无法运行，因为最顶的分层被非向后兼容的新版本替换，或者创建缓存中有一个错误的“最新”版本。在生产中部署容器时应避免使用最新。

 7）不要在单一容器中运行超过一个进程－容器能完美地运行单个进程（http守护进程，应用服务器，数据库），但是如果你不止有一个进程，管理、获取日志、独立更新都会遇到麻烦。

 8）不要在镜像中存储凭据。使用环境变量 –不要将镜像中的任何用户名/密码写死。使用环境变量来从容器外部获取此信息。

 9）**使用非root用户运行进程** – “docker容器默认以root运行。（…）随着docker的成熟，更多的安全默认选项变得可用。现如今，请求root对于其他人是危险的，可能无法在所有环境中可用。你的镜像应该使用USER指令来指令容器的一个非root用户来运行。”

```shell
让用户拥有docker权限
usermod -G docker 用户名
```

 10）不要依赖IP地址 – 每个容器都有自己的内部IP地址，如果你启动并停止它地址可能会变化。如果你的应用或微服务需要与其他容器通讯，使用任何命名与（或者）环境变量来从一个容器传递合适信息到另一个。

## 架构和原理

### **镜像（Image）**

镜像是一个**只读**的文件和文件夹组合。它包含了容器运行时所需要的所有基础文件和配置信息，是容器启动的基础。

实际上image是由**一层层的文件系统组成**的，这种层级的文件系统称为UnionFS。

Docker image来源：

```text
（1）可以基于Dockerfile从无到有的构建；
（2）也可以基于Dockerfile从已有的image创建新的image；
（3）也可以基于容器生成新的image；
（4）甚至也可以直接下载别人的image。
```

镜像名称组成：

```text
registry/repo:tag
```

基础镜像：

```text
⼀个没有任何⽗镜像的镜像，谓之基础镜像
```

镜像ID：

```text
所有镜像都是通过⼀个 64 位⼗六进制字符串 （内部是⼀个 256 bit 的值）来标识的。 为简化使⽤，前12 个字符可以组成⼀个短ID，可以在命令⾏中使⽤。短ID还是有⼀定的碰撞机率，所以服务器总是返回⻓ID
```

分层存储机制：

```text
***docker采用分层构建机制，最底层为bootfs，其之为rootfs***

***bootfs***：用于系统引导的文件系统，包括bootloader和kernel，容器启动完成后会被卸载以节约内存资源；

***rootfs***：位于bootfs之上，表现为docker容器的根文件系统。传统模式中，系统启动时，内核挂载rootfs时会首先将其挂载为只读模式，完整性自检完成后将其重新挂载为只读模式：docker中，rootfs由内核挂载为只读模式，而后通过联合挂载技术额外挂载一个可写层。

位于下层的镜像称之为父镜像，最底层的称之为基础镜像；最上层为可读写层，其下的均为只读层。

为了能启动容器（Containers），需要在本地拥有镜像文件（images）,镜像文件存储在本地特殊的存储位置，此存储位置必须支持一种特殊的文件系统分层挂载或称为联合挂载技术（因为镜像是分层构建的）

***目前支持分层存放的文件系统有：***

1. aufs：advanced multi-layered unification filesystem 高级多层统一文件系统，aufs是之前的UnionFS的重新实现，2006年由Junjiro Okajima开发

2. overlayfs2：高级叠加文件系统。从3.18版后被合并到linux内核

3. dm：devicemapper，稳定性和性能不佳

此外，docker的分层镜像还支持btrfs、vfs等。

在Ubuntu系统下，docker默认Ubuntu的aufs，而在Centos7上用的是devicemapper
```

### **容器（Container）**

![image-20210304112100778](https://gitee.com/c_honghui/picture/raw/master/img/20210304112100.png)

容器是镜像的运行实体。镜像是静态的只读文件，而容器带有运行时需要的可写文件层，并且容器中的进程属于运行状态。即**容器运行着真正的应用进程。容器有初建、运行、停止、暂停和删除五种状态。**

容器的实质是进程，但与直接在宿主执行的进程不同，容器进程运行于**属于自己的独立的 命名空间**。因此容器可以拥有自己的 root 文件系统、自己的网络配置、自己的进程空间，甚至自己的用户 ID 空间。容器内的进程是运行在一个隔离的环境里，使用起来，就好像是在一个独立于宿主的系统下操作一样。

我们可以⽤同⼀个镜像启动多个Docker容器，这些容器启动后都是活动的，彼此还是相互隔离的。我们
对其中⼀个容器所做的变更只会局限于那个容器本身。

如果对容器的底层镜像进⾏修改，那么当前正在运⾏的容器是不受影响的，不会发⽣⾃动更新现象。

如果想更新容器到其镜像的新版本，那么必须当⼼，确保我们是以正确的⽅式构建了数据结构，否则我
们可能会导致损失容器中所有数据的后果。

### **仓库（Repository）**

我们需要一个仓库来存放镜像，Docker官方提供了公共的镜像仓库；从安全和效率的角度考虑我们也可以部署私有环境的Registry或者是Harbor。

----------------------------------------

layer: 在Dockerfile中每一步都会产生一层layer，每一步的结果产出变成文件。

dockerfile: 一种构建image的文件的DSL。

docker-compose: Python写的一个docker编排工具。

docker swarm: docker公司推出的容器调度平台。

kubernetes: google主导的容器调度平台。

### 架构

![image-20210304103656232](https://gitee.com/c_honghui/picture/raw/master/img/20210304103702.png)

Docker 整体架构采用 C/S（客户端 / 服务器）模式。客户端和服务端通信有多种方式，既可以在同一台机器上通过`UNIX`套接字通信，也可以通过网络连接远程通信。

```shell
[root@Pagerduty ~]# docker version
Client: Docker Engine – Community   ## 客户端
 Version:           19.03.6        ## Docker版本号
 API version:       1.40            ## Docker API 版本
 Go version:        go1.12.16       ## Go语言版本
 Git commit:        369ce74a3c     ## GIT commit ID
 Built:             Thu Feb 13 01:29:29 2020   ## 发布日期
 OS/Arch:           linux/amd64            ## 系统版本
 Experimental:      false 

Server: Docker Engine – Community      ### Docker 服务端
 Engine:                              ### 引擎版本号
  Version:          19.03.6
  API version:      1.40 (minimum version 1.12)   ### 服务端API版本
  Go version:       go1.12.16                  ### 服务端Go语言版本
  Git commit:       369ce74a3c                ### 服务端 commit ID
  Built:            Thu Feb 13 01:28:07 2020     ### 发布时间
  OS/Arch:          linux/amd64               ### 系统版本
  Experimental:     false                       ### 是否开启夜间实验
 containerd:                                  ### 容器版本
  Version:          1.4.3
  GitCommit:        269548fa27e0089a8b8278fc4fc781d7f65a939b
 runc:                                        ###轻量级的工具，用来运行容器
  Version:          1.0.0-rc92
  GitCommit:        ff819c7e9184c13b7c2607fe6c30ae19403a7aff
 docker-init:
  Version:          0.18.0
  GitCommit:        fec3683
```

1. Docker 相关的组件：

   docker

   ```text
   docker客户端，负责发送docker请求给Docker Daemon（dockerd）,该程序的安装路径为：/usr/bin/docker
   ```

   dockerd

   ```text
   一般称之为Docker engine，负责接收客户端请求并返回请求结果,该程序的安装路径为：/usr/bin/dockerd
   ```

2. containerd 相关的组件：

   containerd

   ```text
   Containerd 主要负责：管理容器的生命周期（从创建到销毁容器）、拉取/推送镜像、存储管理（管理镜像及容器存储）、调用Runc 运行容器（与Runc等容器运行时交互）、管理容器网络接口及网络。
   ```

   containerd-shim

   ```text
   是容器运行时的载体，我们在Docker宿主机上看到的shim也正是代表着一个个通过调用containerd启动的docker容器。
   containerd-shim 的主要作用是将 containerd 和真正的容器进程解耦，使用 containerd-shim 作为容器进程的父进程，从而实现重启 containerd 不影响已经启动的容器进程。
   ```


### 核心底层技术

Docker使用的核心底层技术：Namespace、Control Groups和Union FS。

**Namespaces**

简单来说，Namespace 是 Linux 内核的一个特性，可以实现在同一主机系统中，对进程 ID、主机名、用户 ID、文件名、网络和进程间通信等资源的隔离。Docker 利用 Linux 内核的 Namespace 特性，实现了每个容器的**资源相互隔离**，从而保证容器内部只能访问到自己 Namespace 的资源。

namespace         系统调用参数            隔离内容                  内核版本
	（1）UTS              CLONE_NEWUTS	       主机名和域名                  2.6.19
	（2）IPC               CLONE_NEWIPC         信息量，消息队列和共享内存    2.6.19
	（3）PID               CLONE_NEWPID	       进程编号                      2.6.24
	（4）Network      CLONE_NEWNET	       网络设备、网络栈、端口等      2.6.29
	（5）Mount         CLONE_NEWNS	       挂载点（文件系统）            2.4.19
	（6）User             CLONE_NEWUSER	       用户和用户组                  3.8

**Control groups**

Control groups（Cgroups）中文称为控制组。Docker利用Cgroups实现了对**资源的配额**和度量。Cgroups可以限制CPU、内存、磁盘读写速率、网络带宽等系统资源。

	blkio：块设备IO；
	cpu：CPU；
	cpuacct：CPU资源使用报告；
	cpuset：多处理器平台上的CPU集合；
	devices：设备访问；
	freezer：挂起或恢复任务；
	memory：内存用量及报告；
	perf_event：对cgroup中的任务进行统一性能测试；
	net_cls：cgroup中的任务创建的数据报文的类别标识符；
```shell
docker run -d --name mysql --memory="500m" --memory-swap="600M" --oom-kill-disabel mysql01
# 限制CPU
docker run -d --name mysql --cpus=".5"
docker run -d --name mysql --cpus="1.5
```

**Union file systems**（联合文件系统，**Contaner和image的分层**）

将镜像多层文件联合挂载到容器文件系统：

![image-20210509225853719](https://gitee.com/c_honghui/picture/raw/master/img/20210509225853.png)

Docker目前支持的UnionFS种类包括AUFS,btrfs,vfs和 DeviceMapper。

采用分层构建机制，最底层是bootfs，其之为rootfs；
		bootfs：用于系统引导的文件系统，包括bootloader和kernel，容器启动完成后会被卸载以节约内存资源；
		rootfs：位于bootfs之上，表现为docker容器的根文件系统；
		传统模式中，系统启动之时，内核挂载rootfs时会首先将其挂载为"只读"模式，完整性自检完成后将其重新挂载为读写模式；
		docker中，rootfs由内核挂载为"只读"模式，而后通过"联合挂载"技术额外挂载一个"可写"层

![img](https://gitee.com/c_honghui/picture/raw/master/img/20210217232523.webp)

在Docker中，上层的image依赖下层的image， 因此Docker中把下层的image称作父image，没有父image的image称作Base image。比如上图中Debian就是Base image，执行add emacs后生成的image就是执行add Apache后生成的image的父image。因此，当想要从一个image启动一个容器，Docker会先逐次加载其父image直到Base image，用户的进程运行在Writeable的文件系统层中。



## 安装

操作系统要求：
CentOS 7 要求系统为64位，系统内核版本为 3.10 以上
CentOS 6.5.X 或更高的版本的 CenOS 上，要求系统为64位，系统内核为2.6.32-431或者更高版本。

```shell
#先执行以下命令卸载旧版 Docker
$ sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
#添加 Docker 安装源
wget https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo -O /etc/yum.repos.d/docker-ce.repo
#正常情况下，直接安装最新版本的 Docker 即可
$ sudo yum install docker-ce docker-ce-cli containerd.io
#如果你想要安装指定版本的 Docker
$ sudo yum list docker-ce --showduplicates | sort -r
docker-ce.x86_64            18.06.1.ce-3.el7                   docker-ce-stable
docker-ce.x86_64            18.06.0.ce-3.el7                   docker-ce-stable
docker-ce.x86_64            18.03.1.ce-1.el7.centos            docker-ce-stable
docker-ce.x86_64            18.03.0.ce-1.el7.centos            docker-ce-stable
docker-ce.x86_64            17.12.1.ce-1.el7.centos            docker-ce-stable
docker-ce.x86_64            17.12.0.ce-1.el7.centos            docker-ce-stable
docker-ce.x86_64            17.09.1.ce-1.el7.centos            docker-ce-stable
$ sudo yum install docker-ce-<VERSION_STRING> docker-ce-cli-<VERSION_STRING> containerd.io
#配置阿里云镜像加速
mkdir /etc/docker
vim /etc/docker/daemon.json
{
    "registry-mirrors": ["https://plqjafsr.mirror.aliyuncs.com"]
    #"registry-mirrors": ["https://registry.docker-cn.com"]
}
systemctl enable docker && systemctl start docker
docker version
```

这里有一个国际惯例，安装完成后，我们需要使用以下命令启动一个 hello world 的容器

```shell
$ sudo docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
0e03bdcc26d7: Pull complete
Digest: sha256:7f0a9f93b4aa3022c3a4c147a449bf11e0941a1fd0bf4a8e6c9408b2600777c5
Status: Downloaded newer image for hello-world:latest
Hello from Docker!
```

## 命令

### 镜像

![image-20210304105957479](https://gitee.com/c_honghui/picture/raw/master/img/20210304105957.png)

搜索镜像：docker search

```shell
[root@localhost ~]# docker search centos
NAME                      DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED
-----显示内容描述—----
NAME：          - 镜像名称
DESCRIPTION：   - 镜像描述说明
STARS：         - 收藏数量
OFFICIAL：      - “OK”表示官方镜像，一般使用官方的镜像或者自己做的
AUTOMATED：     - “OK” 表示自助构建
```

获取镜像：docker pull [选项] [Docker Registry 地址[:端口]]/仓库名/[image]:[tag]

```text
Docker 镜像仓库地址：地址的格式一般是 <域名/IP>[:端口号]，默认地址是 Docker Hub。
在library的镜像也就是官方镜像需要：名称
如果是个人的镜像需要：用户名/软件名
标签：如果不显式指定TAG，则默认会选择latest标签，这会下载仓库中最新版本的镜像
```

一般来说，镜像的latest标签意味着该镜像的内容会跟踪最新的非稳定版本而发布，内容是不稳定的。从稳定性上考虑，不要在生产环境中忽略镜像的标签信息或使用默认的latest标记的镜像。

```shell
[root@localhost ~]# docker pull centos
#该命令相当于docker pull registry.docker-cn.com/library/centos:latest命令，即从默认的注册服务器Docker Hub Registry中的centos仓库来下载标记为latest的镜像。
[root@localhost ~]# docker pull ubuntu:14.04
#镜像可以发布为不同的版本，这种机制我们称之为标签（Tag）。
```

上传镜像：docker  push  [OPTIONS]  NAME[:TAG]

```shell
docker tag docker.io/nginx:last 10.1.100.11:5000/nginx:1.15
docker push 10.1.100.11:5000/nginx:1.15
```

列出本地所有镜像：docker images

```text
仓库名称，标签，镜像 ID、创建时间，镜像大小。镜像ID是唯一标识
dockerhub显示的镜像大小是压缩的
```

查询指定镜像：docker image ls 镜像名 或 docker images | grep 镜像名

查看镜像构建历史：docker history

```shell
[root@localhost ~]# docker history 67f7ad418fdf
```

删除镜像，多个镜像用空格隔开：docker rmi 镜像名或id：

**使用标签删除镜像**

当同一个镜像拥有多个标签的时候，docker rmi命令**只是删除该镜像多个标签中的指定标签**而已，并不影响镜像文件。但当镜像只剩下一个标签的时候就要小心了，此时再使用docker rmi命令会彻底删除镜像。

```shell
[root@node1 ~]# docker rmi nginx:v1
```

**使用镜像ID删除镜像**

删除所有指向该镜像的标签，然后删除该镜像文件本身。注意，当有该镜像创建的容器存在时，镜像文件默认是无法被删除的

```shell
[root@node1 ~]# docker rmi d123f4e55e12 
#如果镜像被使用想强制删除
[root@qfedu.com ~]# docker rmi docker.io/ubuntu:latest --force
```

**删除所有镜像**

```shell
[root@qfedu.com ~]# docker rmi $(docker images -q)
```

查看镜像详情信息：

```shell
[root@qfedu.com ~]# docker image inspect 镜像id
```

重命名镜像：

```shell
[root@qfedu.com ~]# docker tag daocloud.io/ubuntu daocloud.io/ubuntu:v1
#这两个镜像id一样，指向同一个镜像文件，只是别名不同
```

打包镜像：

```shell
[root@node1 ~]# docker save centos > /opt/centos.tar.gz    #导出镜像
或
[root@node1 ~]# docker save centos -o /opt/centos.tar.gz

[root@node1 ~]# docker load < /opt/centos.tar.gz           #导入镜像
或
[root@node1 ~]# docker load -i /opt/centos.tar.gz
```

### 容器

![image-20210304112246217](https://gitee.com/c_honghui/picture/raw/master/img/20210304112246.png)

#### 创建、运行

创建并运行一个容器：

```shell
docker run [参数] 镜像名
#参数：
-i	捕获标准输入输出,交互式操作
-t	分配一个终端或控制台
-d	容器运行在后台，此时所有I/O数据只能通过网络资源或共享卷组进行交互

--entrypoint=""	覆盖imgae的入口点
--restart=always	自动重启，这样每次docker重启后仓库容器也会自动启动
--name="mycontainer"	为容器分配一个名字，不指定会随机分配一个名称,容器的名称是唯一的
--rm	在容器退出时自动清理容器并移除文件系统
--net="bridge": 指定容器的网络连接类型，支持如下：
     bridge / host / none / container:<name|id>

-p	指定本地端口映射到容器端口
-u	指定容器的用户
-h	指定主机名
-v	指定挂载一个本地的已有目录到容器中去作为数据卷

注：修改docker启动参数
1. 停止docker容器
2. docker update --restart=no 容器名字
```

```shell
[root@localhost ~]# docker run -d -p 88:80 --name web -v /src/webapp:/opt/webapp training/webapp python app.py
```

容器的第一个进程（初始命令）必须一直处于前台运行的状态（夯住），否则这个容器就会处于退出状态

业务在容器中运行：夯住，启动服务

```shell
#容器秒停止
docker run -d nginx:last /bin/bash
#容器能保持运行
docker run -d nginx:last
```

某些时候，执行docker run会出错，因为命令无法正常执行容器会直接退出，此时可以查看退出的错误代码。默认情况下，常见错误代码包括：

125： Docker daemon执行出错，例如指定了不支持的Docker命令参数；

126：所指定命令无法执行，例如权限出错；

127：容器内命令无法找到。

#### 查看

查看运行的容器:

```shell
[root@qfedu.com ~]# docker ps
```

查看所有的容器：

```shell
[root@qfedu.com ~]# docker ps -a
```

查看容器的详情：

```shell
[root@localhost ~]# docker inspect 8be128d85d8d
```

查看容器的端口和宿主机端口的映射情况：

```shell
[root@qfedu.com ~]# docker port blog
 80/tcp -> 0.0.0.0:80
 #容器blog的内部端⼝80映射到宿主机的80端⼝，这样可通过宿主机的80端⼝查看容器blog提供的服务
```

#### 启动、停止、删除

强制停止容器： docker kill id或名字

杀死所有running状态的容器

```shell
docker kill $(docker ps -q)
```

停止容器：docker stop id或名字

```shell
[root@localhost ~]# docker stop 2b85a89be878
#当Docker容器中指定的应用终结时，容器也会自动终止。
```

重启容器：docker restart id或名字

删除处于终止或退出状态的容器：docker rm id

强制删除还处于运行状态的容器： docker rm -f id

#### 连接、退出容器

exec(最推荐方式)

交互型任务：

```shell
[root@qfedu.com ~]# docker exec -it 容器id /bin/bash
root@68656158eb8e:/[root@qfedu.com ~]# ls

以root身份进入容器
[root@qfedu.com ~]# docker exec -it -u root 容器id /bin/bash
```

后台任务：

```shell
[root@qfedu.com ~]# docker exec 容器id touch /testfile
```

退出容器的终端：按Ctrl-p Ctrl-q

#### 监控容器的运行

logs命令来查看容器的运⾏⽇志，其中--tail选项可以指定查看最后⼏条⽇志，⽽-t选项则可以对⽇志条⽬附加时间戳。使⽤-f选项可以跟踪⽇志的输出，直到⼿动停⽌。

```shell
[root@qfedu.com ~]# docker logs App_Container
```

top命令显示一个运行的容器里面的进程信息

```shell
[root@qfedu.com ~]# docker top birdben/ubuntu:v1
```

events实时输出docker服务器端的事件，包括容器的创建启动关闭等

```shell
[root@qfedu.com ~]# docker events
```

#### 文件管理

容器和宿主机之间拷贝文件

```shell
[root@qfedu.com ~]# docker cp mysql:/usr/local/bin/docker-entrypoint.sh /root
#拷贝mysql容器的文件到本地的/root
```

#### 导入导出容器

导出命令

```shell
docker export busybox > busybox.tar
```

导入上一步导出的容器

```shell
docker import busybox.tar busybox:test
```

此时，busybox.tar 被导入成为新的镜像，镜像名称为 busybox:test ，然后用docker run启动容器，就实现了容器的迁移

### 其他

查看docker版本:docker version

查看docker状态信息：

```shell
[root@Pagerduty ~]# docker info
Client:            ## docker 客户端信息
 Debug Mode: false
Server:           ## 代表服务端
 Containers: 1     ## 容器数量
  Running: 0      ## 运行容器数量
  Paused: 0       ## 暂停容器数量
  Stopped: 1      ## 停止容器数量
 Images: 1        ## 镜像数量
 Server Version: 19.03.6   ## docker服务版本
 Storage Driver: overlay2   ## docker存储驱动程序
  Backing Filesystem: xfs    ## 文件系统
  Supports d_type: true     ## 
  Native Overlay Diff: true
 Logging Driver: json-file    ## 日志驱动程序
 Cgroup Driver: cgroupfs     ## Cgroup 驱动程序
 Plugins:                   ## 插件信息
  Volume: local
  Network: bridge host ipvlan macvlan null overlay
  Log: awslogs fluentd gcplogs gelf journald json-file local logentries splunk syslog
 Swarm: inactive   ## Swarm 状态
Runtimes: runc    ## runtimes 信息
 Default Runtime: runc   ## 默认 runtimes 
 Init Binary: docker-init
 containerd version: 269548fa27e0089a8b8278fc4fc781d7f65a939b
 runc version: ff819c7e9184c13b7c2607fe6c30ae19403a7aff
 init version: fec3683
 Security Options:       ## 安全选项
  seccomp
   Profile: default
 Kernel Version: 3.10.0-1160.11.1.el7.x86_64  ## Linux内核版本
 Operating System: CentOS Linux 7 (Core)    ## Linux操作系统
 OSType: linux                     ## 操作系统类型
 Architecture: x86_64         ## 系统架构
 CPUs: 4                       ## CPU 数量
 Total Memory: 1.777GiB         ## 宿主机内存
 Name: Pagerduty             ## 宿主机名称
 ID: 4RDT:F4PO:L57F:3DLQ:JVPK:26C4:LD3W:J3VV:ZX4M:JG3S:SM4R:7EJL
 Docker Root Dir: /var/lib/docker   ## Docker目录
 Debug Mode: false
 Registry: https://index.docker.io/v1/   ## 镜像仓库
 Labels:
 Experimental: false
 Insecure Registries:         ## 非安全镜像仓库
  127.0.0.0/8
 Registry Mirrors:           ## 镜像加速
  https://plqjafsr.mirror.aliyuncs.com/
 Live Restore Enabled: false
```

获取容器端口：docker port 690123a26237

查看 docker 的硬盘空间使用情况：docker system df

更新容器启动项：docker container update --restart=always nginx

登录登出仓库：docker login/docker logout



## dockerfile

Dockerfile 是一个文本文件，其内包含了一条条的指令(Instruction)，每一条指令构建一层，因此每一条指令的内容，就是描述该层应当如何构建。

**生产实践中一定优先使用 Dockerfile 的方式构建镜像。**用commit创建的镜像会比较大。 使用 Dockerfile 构建镜像可以带来很多好处：

- 易于版本化管理，Dockerfile 本身是一个文本文件，方便存放在代码仓库做版本管理，可以很方便地找到各个版本之间的变更历史；
- 过程可追溯，Dockerfile 的每一行指令代表一个镜像层，根据 Dockerfile 的内容即可很明确地查看镜像的完整构建过程；
- 屏蔽构建环境异构，使用 Dockerfile 构建镜像无须考虑构建环境，基于相同 Dockerfile 无论在哪里运行，构建结果都一致。

制作小镜像：

- 不要使用centos镜像，首选apline，如果用到Glibc，可以是node:slim、python:slim
- 使用多阶段构建（多个from）

### dockerfile语法

Dockerfile 由一行行命令语句组成，并且支持以#开头的注释行。
一般而言，Dockerfile分为四部分：基础镜像信息、维护者信息、镜像操作指令和容器启动时执行指令。

**FROM**

```dockerfile
指定所创建镜像的基础镜像，如果本地不存在，则默认会去 Docker Hub下载指定镜像
任何Dockerfile 中的第一条指令必须为 FROM指令
FROM <image> [AS <name>] 
FROM centos:latest                                       #or
FROM <image>[:<tag>] [AS <name>]                        
FROM centos:7.5                                          #or
FROM <image>[@<digest>] [AS <name>]

注意：如果是从私有仓库拉取镜像，如果有配置权限，那么需要先登录到私有仓库。

多阶段构建：
#build
FROM golang:1.14.4-apline
RUN go build /opt/main.go
CMD "./opt/main"
#create image
FROM aline:3.8
COPY --from=0 /opt/main /
CMD "./main"
```

**MAINTAINER**

```dockerfile
用于将image的制作者相关的信息写入image中
LABEL maintainer="SvenDowideit@home.org.au"
LABEL version=0.1
LABEL deacription="Dev Basic PHP 5.3/5.6/7.2"
```

USER

```dockerfile
指定运行容器的用户，默认是root

指令memcached的运行用户
USER = su – user11(centos)
ENTRYPOINT [“memcached”]
USER daemon
ENTRYPOINT [“memached”,”-u”,”daemon”]
```

ENV

```dockerfile
指定环境变量，后续 RUN 指令可以使用，container启动后，可以通过docker inspect查看这个环境变量，也可以通过docker run - -env key=value时设置或修改环境变量。
环境变量可用于ADD、COPY、ENV、EXPOSE、FROM、LABEL、USER、VOLUME、WORKDIR、ONBUILD指令中。
ENV <key> <value>
ENV <key>=<value> ...

假如你安装了JAVA程序，需要设置JAVA_HOME，那么你可以在Dockerfile中这样写：
ENV JAVA_HOME /path/to/java/dirent
ENV JAVA_HOME /usr/java/latest
ENV PATH $JAVA_HOME/bin:$PATH
ENV LANG en_us.UTF-8
```

**RUN**

```dockerfile
构建指令，用来执行shell命令。 这些命令包括安装软件、创建文件和目录，以及创建环境配置等。
RUN <command>或RUN ["executable"，"param1"，"param2"]
前者将在shell终端中运行命令，即/bin/sh -c；后者则使用exec执行，可以用来指定其它形式的shell来运行指令。
RUN yum update && yum install -y vim \
    python-dev #反斜线换行
    
注意初学docker容易出现的2个关于RUN命令的问题：
1.RUN代码没有合并。
2.每一层构建的最后一定要清理掉无关文件。

注：容器在启动时，会挂载三个配置文件
/etc/hostname
/etc/hosts
/etc/resolv.conf
Dockerfile每执行一个run会临时创建一个容器（镜像层），每次从头创建都会重新挂载这三个配置文件。所以有对于次三个配置文件有依赖操作的命令需要处于同一个RUN
```

CMD（容易被替换）

```dockerfile
用来指定启动容器时默认执行的命令。它支持三种格式：
CMD ["executable","param1","param2"]      #使用exec执行，推荐方式；exec 可以保证我们的业务进程就是 1 号进程，这对于需要处理 SIGTERM 信号量实现优雅终止十分重要。
CMD ["param1","param2"]                   #提供给ENTRYPOINT的默认参数； 
CMD command param1 param2                 #在/bin/sh中执行，提供给需要交互的应用；

每个Dockerfile只能有一条CMD命令
如果用户启动容器时手动指定了运行的命令（作为 run 的参数），则会覆盖掉CMD指定的命令。
```

**ENTRYPOINT**（无法被替换）

```dockerfile
启动容器真正执行的命令，并且不可被docker run提供的参数覆盖。每个Dockerfile中只能有一个ENTRYPOINT
ENTRYPOINT ["executable", "param1", "param2"] (exec form, preferred)
例：ENTRYPOINT ["/bin/echo", "hello Ding"]
   ENTRYPOINT ["/bin/sh/", "-c", "/bin/echo hello $name" ]
ENTRYPOINT command param1 param2 (shell form)

该指令的使用分为两种情况一种独自使用，另一种和CMD指令配合使用。
当独自使用时，如果你还使用了CMD命令且CMD是一种完整的可执行的命令，那么CMD指令和ENTRYPOINT会相互覆盖只有最后一个CMD或者ENTRYPOINT有效。并且被覆盖的CMD指令将不会被执行，只有ENTRYPOINT指令被执行。
```

LABEL

```text
给镜像添加信息。使用docker inspect可查看镜像的相关信息
LABEL <key>=<value> <key>=<value> <key>=<value> ...
LABEL version="1.0"
LABEL maintainer="394498036@qq.com"
```

**EXPOSE**

```dockerfile
暴露的端口号
EXPOSE <port> [<port>/<protocol>...]
EXPOSE 22 80 8443
注意，该指令只是起到声明作用，并不会自动完成端口映射。
```

**ADD**

```dockerfile
将复制指定的 <src>路径下的内容到容器中的<dest>路径下，<src>可以是dockerfile所在目录的相对路径、一个URL、还可以是tar文件（会自动解压）
ADD [--chown=<user>:<group>] <src>... <dest>
ADD [--chown=<user>:<group>] ["<src>",... "<dest>"] (this form is required for paths containing whitespace)
```

**COPY**

```dockerfile
复制本地主机的<src>（为 Dockerfile 所在目录的相对路径、文件或目录）下的内容到镜像中的 <dest> 下。目标路径不存在时，会自动创建。
尽量使用COPY不使用ADD.
COPY [--chown=<user>:<group>] <src>... <dest>
COPY [--chown=<user>:<group>] ["<src>",... "<dest>"] (this form is required for paths containing whitespace)
```

**WORKDIR**

```text
为后续的RUN、CMD和ENTRYPOINT 指令配置工作目录，类似cd，可多次切换
用WORKDIR，不要用RUN cd 尽量使用绝对目录！
WORKDIR /path/to/workdir
```

VOLUME

```dockerfile
创建一个可以从本地主机或其他容器挂载的挂载点，一般用来存放数据库和需要保持的数据等
VOLUME ["/data"]

注意：
host机器的目录路径必须为全路径(准确的说需要以/或~/开始的路径)，不然docker会将其当做volume而不是volume处理
如果host机器上的目录不存在，docker会自动创建该目录
如果container中的目录不存在，docker会自动创建该目录
如果container中的目录已经有内容，那么docker会使用host上的目录将其覆盖掉
```

### docker build语法

```shell
[root@qfedu.com ~]# docker build [OPTIONS] dockerfile所在路径
#选项说明
-t, --tag list # 镜像名称
-f, --file string # 指定Dockerfile文件位置
docker build -t shykes/myapp .
docker build -t shykes/myapp -f /path/Dockerfile /path
docker build -t shykes/myapp http://www.example.com/Dockerfile
```

编译jdk dockerfile

```dockerfile
[root@Pagerduty ~]# mkdir /jdk
[root@Pagerduty /jdk]#  cat Dockerfile 
FROM docker.io/jeanblanchard/alpine-glibc
MAINTAINER Mengfei
RUN echo "https://mirror.tuna.tsinghua.edu.cn/alpine/v3.4/main/" > /etc/apk/repositories
RUN apk add --no-cache bash
ADD jre1.8.0_211.tar.gz /usr/java/jdk/
ENV JAVA_HOME /usr/java/jdk/jre1.8.0_211
ENV PATH ${PATH}:${JAVA_HOME}/bin
RUN chmod +x /usr/java/jdk/jre1.8.0_211/bin/java
RUN mkdir /zz
USER root
VOLUME ["/tmp/data"]
WORKDIR /opt
#EXPOSE 22
CMD ["java","-version"]
# ENTRYPOINT ["java", "-version"]
```

编译镜像

```shell
[root@Pagerduty /jdk]# docker build -t jre8:1.5 .
//注意：需要把jre8:1.5的软件包，放入当前目录中
```

启动容器

```shell
[root@Pagerduty /jdk]# docker run -it --name=jre-V1.1  jre8:1.5 bash
bash-4.3#
```

### 为镜像添加ssh服务

创建工作目录,并编写run.sh脚本和authorized_keys文件

```shell
[root@localhost ~]# mkdir sshd_ubuntu
[root@localhost ~]# cd sshd_ubuntu/
[root@localhost sshd_ubuntu]# vim run.sh
#!/bin/bash
/usr/sbin/sshd -D
[root@localhost sshd_ubuntu]# ssh-keygen -t rsa
[root@localhost sshd_ubuntu]# cat ~/.ssh/id_rsa.pub >authorized_keys
```

编写dockerfile

```shell
[root@localhost sshd_ubuntu]# vim Dockerfile
#设置继承镜像
FROM ubuntu:14.04
#提供一些作者的信息
MAINTAINER docker_user (user@docker.com)
#下面开始运行更新命令
RUN apt-get update
#安装ssh服务
RUN apt-get install -y openssh-server
RUN mkdir -p /var/run/sshd
RUN mkdir -p /root/.ssh
#取消pam限制
RUN sed -ri 's/session    required     pam_loginuid.so/#session    required     pam_loginuid.so/g' /etc/pam.d/sshd
#复制配置文件到相应位置,并赋予脚本可执行权限
ADD authorized_keys /root/.ssh/authorized_keys
ADD run.sh /run.sh
RUN chmod 755 /run.sh
#开放端口
EXPOSE 22
#设置自启动命令
CMD ["/run.sh"]
```

创建镜像

```shell
[root@localhost sshd_ubuntu]# docker build -t sshd:Dockerfile .
```

运行镜像

```shell
[root@localhost sshd_ubuntu]# docker run -d -p 10122:22 sshd:Dockerfile
```

### dockerfile封装nginx

```shell
mkdir  nginx
cd nginx
wget  http://nginx.org/download/nginx-1.15.2.tar.gz
vim Dockerfile
FROM centos	//使用官方的centos镜像作为基础镜像
MAINTAINER NGINX Docker Maintainers "docker-maint@nginx.com"	//指定维护者信息
RUN yum -y install gcc make pcre-devel zlib-devel tar zlib	//运行命令
ADD nginx-1.15.2.tar.gz /usr/src/	//把nginx压缩包复制到/usr/src/
RUN cd /usr/src/nginx-1.15.2 \
	&& mkdir /usr/local/nginx \
    && ./configure --prefix=/usr/local/nginx && make && make install \
    && ln -s /usr/local/nginx/sbin/nginx /usr/local/sbin/ \
    && nginx
RUN rm -rf /usr/src/nginx-1.15.2
EXPOSE 80	//允许外界访问容器的 80 端⼝
ENTRYPOINT [ "nginx", "-g", "daemon off;"]	#nginx默认是以后台模式启动的，Docker未执行自定义的CMD之前，nginx的pid是1，执行到CMD之后，nginx就在后台运行，bash或sh脚本的pid变成了1。所以一旦执行完自定义CMD，nginx容器也就退出了。为了保持nginx的容器不退出，应该关闭nginx后台运行
#构建镜像
docker build -t nginx:2020 .
#启动镜像
docker run -itd -p 88:80  -v /home/anhao1226/:/usr/local/nginx/html nginx:20201020
```

## 网络

网络模式主要有四种，这四种网络模式基本满足了我们单机容器的所有场景。：

```text
null 空网络模式：可以帮助我们构建一个没有网络接入的容器环境，以保障数据安全。
bridge 桥接模式：可以打通容器与容器间网络通信的需求。
host 主机网络模式：可以让容器内的进程共享主机网络，从而监听或修改主机网络。
container 网络模式：可以将两个容器放在同一个网络命名空间内，让两个业务通过 localhost 即可实现访问。
```

#### （1）null 空网络模式

获取独立的network namespace，但不为容器进行任何网络配置，需要我们手动配置。

添加 --net=none 参数启动一个空网络模式的容器

```shell
$ docker run --net=none -it busybox
/ #
```

查看一下容器内网络配置信息

```shell
/ # ifconfig
lo        Link encap:Local Loopback
          inet addr:127.0.0.1  Mask:255.0.0.0
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)
/ # route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
```

#### （2）bridge 桥接模式（docker默认的网络模式）

实现容器与容器的互通，主机与容器的互通

Bridge 模式是Docker默认的网络设置，此模式会为每一个容器分配Network Namespace、设置IP等，并将一个主机上的Docker容器连接一个虚拟网卡。

![image-20210304163014916](https://gitee.com/c_honghui/picture/raw/master/img/20210304163015.png)



#### （3）host 主机网络模式（共享主机的网络模式）

容器不会获得一个独立的network namespace，而是与宿主机共用一个。这就意味着容器不会有自己的网卡信息，而是使用宿主 机的。容器除了网络，其他都是隔离的。

注意：如果是host模式，命令行参数不能带-p/-P主机端口：容器端口

场景：组建集群，对网络要求比较高的话，可以用host模式。

```shell
$ docker run -it --net=host busybox
/ #

/ # ip a
1: lo: <LOOPBACK,UP,LOWER\_UP> mtu 65536 qdisc noqueue qlen 1000
link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
inet 127.0.0.1/8 scope host lo
valid\_lft forever preferred\_lft forever
inet6 ::1/128 scope host
valid\_lft forever preferred\_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER\_UP> mtu 1500 qdisc pfifo\_fast qlen 1000
link/ether 02:11:b0:14:01:0c brd ff:ff:ff:ff:ff:ff
inet 172.20.1.11/24 brd 172.20.1.255 scope global dynamic eth0
valid\_lft 85785286sec preferred\_lft 85785286sec
inet6 fe80::11:b0ff:fe14:10c/64 scope link
valid\_lft forever preferred\_lft forever
3: docker0: \<NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue
link/ether 02:42:82:8d:a0:df brd ff:ff:ff:ff:ff:ff
inet 172.17.0.1/16 scope global docker0
valid\_lft forever preferred\_lft forever
inet6 fe80::42:82ff:fe8d:a0df/64 scope link
valid\_lft forever preferred\_lft forever
```

可以看到容器内的网络环境与主机完全一致。

#### （4）container 网络模式（容器之间的共享模式，学习k8s这个很重要）

container 网络模式允许一个容器共享另一个容器的网络命名空间。当两个容器需要共享网络，但其他资源仍然需要隔离时就可以使用 container 网络模式

```shell
$ docker exec -it busybox1 sh
/ # ifconfig
eth0 Link encap:Ethernet HWaddr 02:42:AC:11:00:02
inet addr:172.17.0.2 Bcast:172.17.255.255 Mask:255.255.0.0
UP BROADCAST RUNNING MULTICAST MTU:1500 Metric:1
RX packets:11 errors:0 dropped:0 overruns:0 frame:0
TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
collisions:0 txqueuelen:0
RX bytes:906 (906.0 B) TX bytes:0 (0.0 B)
$ docker run -it --net=container:busybox1 --name=busybox2 busybox sh
/ # ifconfig
eth0 Link encap:Ethernet HWaddr 02:42:AC:11:00:02
inet addr:172.17.0.2 Bcast:172.17.255.255 Mask:255.255.0.0
UP BROADCAST RUNNING MULTICAST MTU:1500 Metric:1
RX packets:14 errors:0 dropped:0 overruns:0 frame:0
TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
collisions:0 txqueuelen:0
RX bytes:1116 (1.0 KiB) TX bytes:0 (0.0 B)
```

container模式存在如下特点：

1、两个容器间通过127.0.0.1可以实现高效快速通信。

2、可能存在端口冲突情况。

**自定义网络**

创建新的bridge网络

```shell
docker network create --driver bridge --subnet 172.50.0.0/16 --gateway 172.50.0.1 --opt"com.docker.network.bridge.name"="docker1000" my_bridge

docker network inspect my_bridge1
```

使用创建的网络

```shell
docker run -itd --name mysql1 --network=my_bridge mysql /bin/bash
```

不同网络中的容器如何访问？

把容器加到同一个网络中来

```shell
docker network connect my_bridge mysql2
```

自定义的bridge网络可以删除，默认bridge网络不可删除

```shell
docker network rm my_bridge
```

默认bridge网络中所有容器间只能用IP相互访问。使用自定义bridge网络的容器间既可以通过ip访问，也可以通过容器名访问。原因在于Docker从1.10版本内嵌了了一个DNS服务，使得容器间可以直接通过容器名通信。

docker run指定容器ip启动时仅适用于自定义网络。

## 数据卷管理

默认情况下，容器内创建的所有文件都存储在可写容器层上。

数据卷是一个或多个容器专门指定绕过Union File System，为持续性或共享数据提供一些有用的功能：

```text
1.数据卷 可以在容器之间共享和重用。
2.对 数据卷 的修改会立马生效。
3.对 数据卷 的更新，不会影响镜像。
4.数据卷 默认会一直存在，即使容器被删除。保护数据不被删除。
注意：数据卷 的使用，类似于 Linux 下对目录或文件进行 mount，镜像中的被指定为挂载点的目录中的文件会隐藏掉，能显示和看的是挂载的 数据卷，而不是被作为挂载点儿的目录中原来的内容。
```

从容器拷贝数据到主机上：

```shell
[root@Pagerduty /]# docker cp 92592755c544:/docker.txt /opt
[root@Pagerduty /]# ls /opt/
containerd  docker.txt
```

从宿主机拷贝数据到容器目录：

```shell
[root@Pagerduty ~]# docker cp /opt/centos.txt 92592755c544:/tmp/
```



创建一个名为myvolume的数据卷

```shell
$ docker volume create myvolume
```

在这里要说明下，默认情况下 ，Docker 创建的数据卷为 local 模式，仅能提供本主机的容器访问。如果想要实现远程访问，需要借助网络存储来实现。Docker 的 local 存储模式并未提供配额管理，因此在生产环境中需要手动维护磁盘存储空间。

还可以在 Docker 启动时使用 -v 的方式指定容器内需要被持久化的路径，Docker 会自动为我们创建卷，并且绑定到容器中:

```shell
$ docker run -d --name=nginx-volume -v /usr/share/nginx/html nginx
-v host-dir:container-dir:[rw|wo]
-v container-dir:[rw|wo]
-v volume-name:container-dir:[rw|wo]
host-dir：表示主机上的目录，如果不存在，Docker 会自动在主机上创建该目录。必须是绝对路径。
container-dir：表示容器内部对应的目录，如果该目录不存在，Docker 也会在容器内部创建该目录。
volume-name：表示卷名，如果该卷不存在，docker将自动创建。
rw|ro：用于控制volume的读写权限。
```

查看下主机上的卷：

```shell
$ docker volume ls
DRIVER              VOLUME NAME
local               eaa8a223eb61a2091bf5cd5247c1b28ac287450a086d6eee9632d9d1b9f69171
```

查看某个数据卷的详细信息:

```shell
$ docker volume inspect myvolume
[
    {
        "CreatedAt": "2020-09-08T09:10:50Z",
        "Driver": "local",
        "Labels": {},
        "Mountpoint": "/var/lib/docker/volumes/myvolume/_data",
        "Name": "myvolume",
        "Options": {},
        "Scope": "local"
    }
]
```

使用数据卷

使用上一步创建的卷来启动一个 nginx 容器，并将 /usr/share/nginx/html 目录与卷关联:

```shell
$ docker run -d --name=nginx --mount source=myvolume,target=/usr/share/nginx/html nginx
```

删除数据卷

这里需要注意，正在被使用中的数据卷无法删除，如果你想要删除正在使用中的数据卷，需要先删除所有关联的容器。

```shell
$ docker volume rm myvolume
```

容器与容器之间数据共享

有时候，两个容器之间会有共享数据的需求，很典型的一个场景就是容器内产生的日志需要一个专门的日志采集程序去采集日志内容

```shell
$ docker volume create log-vol
$ docker run --mount source=log-vol,target=/tmp/log --name=log-producer -it busybox
docker run -it --name consumer --volumes-from log-producer  busybox
```

**主机与容器之间数据共享**

卷作为一种统一封装，不仅可以支持本地目录，也可以支持 NFS 等网络文件系统。如果你的需求仅仅需要本地目录，使用 `-v 宿主机目录:容器`目录挂载也完全可以的。

Docker 卷的目录默认在 **/var/lib/docker** 下，当我们想把主机的其他目录映射到容器内时，就需要用到主机与容器之间数据共享的方式了，例如我想把 MySQL 容器中的 /var/lib/mysql 目录映射到主机的 /var/lib/mysql 目录中，我们就可以使用主机与容器之间数据共享的方式来实现。

挂载主机的 /data 目录到容器中的 /usr/local/data 中，可以使用以下命令来启动容器:

```shell
$ docker run -v /data:/usr/local/data -it busybox
```

容器启动后，便可以在容器内的 /usr/local/data 访问到主机 /data 目录的内容了，并且容器重启后，/data 目录下的数据也不会丢失。

**修改docker存储目录**

```shell
[root@Pagerduty ~]# vi /usr/lib/systemd/system/docker.service 
-----------修改内容截取--------------
    9	[Service]
    10	Type=notify
    11	# the default is not to use systemd for cgroups because the delegate issues still
    12	# exists and systemd currently does not support the cgroup feature set required
    13	# for containers run by docker
    14	#ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
    15	ExecStart=/usr/bin/dockerd --graph=/data/docker -H fd:// --containerd=/run/containerd/containerd.sock
    16	ExecReload=/bin/kill -s HUP $MAINPID
#######################################################
修改内容如下，将官方内容注释，在15行处，添加一下内容：
ExecStart=/usr/bin/dockerd --graph=/data/docker -H fd:// --containerd=/run/containerd/containerd.sock
############################################
随后重新加载配置文件，并重启docker：
[root@Pagerduty ~]# systemctl daemon-reload
[root@Pagerduty ~]# systemctl restart docker
执行docker info 查看，是否修改成功：
[root@Pagerduty ~]# docker info
-------------内容截取，红字处，修改成功-----------
Total Memory: 1.777GiB
 Name: Pagerduty
 ID: 4RDT:F4PO:L57F:3DLQ:JVPK:26C4:LD3W:J3VV:ZX4M:JG3S:SM4R:7EJL
 Docker Root Dir: /data/docker
 Debug Mode: false
 Registry: https://index.docker.io/v1/
-----------------内容截取---------------
```

## 私有仓库

在实际的生产过程中会使用到Harbor



私有仓库工作流程：

1)    用户本地构建镜像，将镜像推送到Registry仓库

2)    Docker用户使用的时候，直接从Registry下载，无须从Docker Hub下载



搭建私有仓库：

镜像名称：Registry，默认使用最新版本。

挂载宿主机的/opt/myregistry目录到容器目录的/var/lib/registry

```shell
[root@Pagerduty ~]# mkdir -p  /opt/myregistry  
[root@Pagerduty ~]# docker run -d -p  5000:5000 --restart=always --name=registry -v  /opt/myregistry/:/var/lib/registry registry
```



上传本地镜像到私有仓库：

```shell
打标签
docker image tag centos:latest 192.168.13:5000/centos7:V1.0
上传
docker push 192.168.1.13:5000/centos7:V1.0
```

出现htpps报错的解决方式：

1)    修改Docker节点配置文件

2)    添加nginx反向代理

```shell
[root@Pagerduty ~]# cat /etc/docker/daemon.json
{
  "registry-mirrors": ["https://plqjafsr.mirror.aliyuncs.com"],
  "data-root": "/data/docker",
  "insecure-registries": ["192.168.1.13:5000"],
  "storage-driver": "overlay2",
  "storage-opts":[
     "overlay2.override_kernel_check=true",
     "overlay2.size=1G"
 ]
}
```



