# Docker入门

## 简介

不同的应用程序可能会有不同的应用环境，比如.net开发的网站和java开发的网站依赖的软件就不一样，如果把他们依赖的软件都安装在一个服务器上就要调试很久，而且很麻烦，还会造成一些冲突。比如IIS和tomcat访问端口冲突。这个时候你就要隔离.net开发的网站和tomcat开发的网站。常规来讲，我们可以在服务器上创建不同的虚拟机在不同的虚拟机上放置不同的应用，但是虚拟机开销比较高。**docker可以实现虚拟机隔离应用环境的功能，并且开销比虚拟机小**，小就意味着省钱了。

Docker将应用程序与该程序的依赖，打包在一个文件里面。运行这个文件，就会生成一个虚拟容器。程序在这个虚拟容器里运行，就好像在真实的物理机上运行一样。有了 Docker，就不用担心环境问题。

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

 10）不要依赖IP地址 – 每个容器都有自己的内部IP地址，如果你启动并停止它地址可能会变化。如果你的应用或微服务需要与其他容器通讯，使用任何命名与（或者）环境变量来从一个容器传递合适信息到另一个。

## 架构

### **镜像（Image）**

Image是docker的基石，是一个可运行的基本单元。image是只读的，包括container运行所需要的数据，主要用来创建container。实际上image是由一层层的文件系统组成的，这种层级的文件系统称为UnionFS。Docker image来源：

```text
（1）可以基于Dockerfile从无到有的构建；

（2）也可以基于Dockerfile从已有的image创建新的image；

（3）也可以基于容器生成新的image；

（4）甚至也可以直接下载别人的image。
```

镜像名称组成：registry/repo:tag

基础镜像：⼀个没有任何⽗镜像的镜像，谓之基础镜像

镜像ID：所有镜像都是通过⼀个 64 位⼗六进制字符串 （内部是⼀个 256 bit 的值）来标识的。 为简化使⽤，前
12 个字符可以组成⼀个短ID，可以在命令⾏中使⽤。短ID还是有⼀定的碰撞机率，所以服务器总是返回
⻓ID

分层存储机制：

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

### **容器（Container）**

⼀个Docker容器包含了所有的**某个应⽤运⾏所需要的环境**。每⼀个Docker 容器都是从 Docker 镜像创建的。

容器的实质是进程，但与直接在宿主执行的进程不同，容器进程运行于**属于自己的独立的 命名空间**。因此容器可以拥有自己的 root 文件系统、自己的网络配置、自己的进程空间，甚至自己的用户 ID 空间。容器内的进程是运行在一个隔离的环境里，使用起来，就好像是在一个独立于宿主的系统下操作一样。

**容器有两个状态**：运行（running）和退出（exited）

我们可以⽤同⼀个镜像启动多个Docker容器，这些容器启动后都是活动的，彼此还是相互隔离的。我们
对其中⼀个容器所做的变更只会局限于那个容器本身。

如果对容器的底层镜像进⾏修改，那么当前正在运⾏的容器是不受影响的，不会发⽣⾃动更新现象。

如果想更新容器到其镜像的新版本，那么必须当⼼，确保我们是以正确的⽅式构建了数据结构，否则我
们可能会导致损失容器中所有数据的后果。

### **仓库（Repository）**

如百度网盘一样，我们需要一个仓库来存放镜像，Docker官方提供了公共的镜像仓库；从安全和效率的角度考虑我们也可以部署私有环境的Registry或者是Harbor。

----------------------------------------

layer: 在Dockerfile中每一步都会产生一层layer，每一步的结果产出变成文件。

dockerfile: 一种构建image的文件的DSL。

docker-compose: Python写的一个docker编排工具。

docker swarm: docker公司推出的容器调度平台。

kubernetes: google主导的容器调度平台。

### 核心底层技术

Docker使用的核心底层技术：Namespace、Control Groups和Union FS。

**Namespaces**

每个docker主机上可以起很多container，这些container之间是相互隔离，互不影响的。Docker正是借助Linux kernel namespace(命名空间)来实现这一点。具体包括pid、net、ipc、mnt、uts、user等namespace将container的进程、网络、消息、文件系统、UTS("UNIX Time-sharing System")和用户空间隔离开。

**Control groups**

Control groups（Cgroups）中文称为控制组。Docker利用Cgroups实现了对资源的配额和度量。Cgroups可以限制CPU、内存、磁盘读写速率、网络带宽等系统资源。

**Union file systems**（联合文件系统，UnionFS）

Docker目前支持的UnionFS种类包括AUFS,btrfs,vfs和 DeviceMapper。

AUFS是一种 Union FS, 简单来说就是“支持将不同目录挂载到同一个虚拟文件系统下的文件系统”, AUFS支持为每一个成员目录设定只读(Rreadonly)、读写(Readwrite)和写(Whiteout-able)权限。

Linux在启动后，首先将rootfs置为 Readonly，进行一系列检查后将其切换为Readwrite供用户使用。在Docker中，也是利用该技术，然后利用Union Mount在Readonly的rootfs文件系统之上挂载Readwrite文件系统。并且向上叠加, 使得一组Readonly和一个Readwrite的结构构成一个容器的运行目录、每一个被称作一个文件系统Layer。

AUFS的特性, 使得每一个对Readonly层文件/目录的修改都只会存在于上层的Writeable层中。这样使得多个容器可以共享Readonly文件系统层。在Docker中，将**Readonly的层称作image**，将**Writeable层称作container**。对于容器整体而言，整个rootfs变得是read-write的，但事实上所有的修改都写入最上层的container中，image不保存用户状态，可以用于模板、重建和复制。

![img](https://gitee.com/c_honghui/picture/raw/master/img/20210217232523.webp)

在Docker中，上层的image依赖下层的image， 因此Docker中把下层的image称作父image，没有父image的image称作Base image。比如上图中Debian就是Base image，执行add emacs后生成的image就是执行add Apache后生成的image的父image。因此，当想要从一个image启动一个容器，Docker会先逐次加载其父image直到Base image，用户的进程运行在Writeable的文件系统层中。

## 网络

网络四种模式：

```text
bridge模式：使用–net =bridge指定，默认设置；
host模式：使用–net =host指定；
none模式：使用–net =none指定；
container模式：使用–net =container:NAMEorID指定。
```

**bridge模式（docker默认的网络模式）**

```text
在默认情况下，docker 会在 host 机器上新创建一个 docker0 的 bridge：可以把它想象成一个虚拟的交换机，所有的容器都是连到这台交换机上面的。docker 会从私有网络中选择一段地址来管理容器，比如 172.17.0.1/16，这个地址根据你之前的网络情况而有所不同。
```

自定义网络：

```text
建议大家不要使用link的方式，如果容器千千万都link，人就受不了了。还是自定网络比较靠谱。
测试网络通信，创建容器，进行通信
不需要ip的方式两个容器都是通的。
```

```shell
docker run --name test3 --network net-test -d busybox /bin/sh -c "while true;do echo hello docker;sleep 10;done"
docker run --name test4 --network net-test -d busybox /bin/sh -c "while true;do echo hello docker;sleep 10;done"
docker exec -it test3 /bin/sh
ping test4
exit
docker exec -it test4 /bin/sh
ping test3
exit
```

**host模式（共享主机的网络模式）**

```text
docker 不会为容器创建单独的网络 namespace，而是共享主机的 network namespace，也就是说：容器可以直接访问主机上所有的网络信息。在实际微服务的环境中不建议使用这种。
```

**none模式（空网络模式）**

```text
这种none的也就自己通过exec的方式访问。
```

**container 模式（容器之前的共享模式，学习k8s这个很重要）**

```text
一个容器直接使用另外一个已经存在容器的网络配置：ip 信息和网络端口等所有网络相关的信息都是共享的。需要注意的是：这两个容器的计算和存储资源还是隔离的。
```

```shell
# test7_container 依赖a1的网络模式
 docker run --name test7_container --network container:a1 -d busybox /bin/sh -c "while true;do echo hello docker;sleep 10;done" 
# 分别进入test7_container 和a1查看ifconfig 发现两个是一样的
docker exec -it test7_container /bin/sh
ifconfig
exit
docker exec -it a1 /bin/sh
ifconfig
exit
```

## 安装

```shell
wget https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo -O /etc/yum.repos.d/docker-ce.repo
yum install docker-ce -y
#指定版本安装
#yum -y install docker-ce-18.06.1.ce-3.el7
#配置镜像加速
mkdir /etc/docker
vim /etc/docker/daemon.json
{
    "registry-mirrors": ["https://registry.docker-cn.com"]
}
systemctl enable docker && systemctl start docker
docker version
```

## 命令

### 镜像

搜索镜像：docker search

```shell
[root@localhost ~]# docker search centos
```

获取镜像：docker pull [选项] [Docker Registry 地址[:端口]/]仓库名[:标签]

```text
Docker 镜像仓库地址：地址的格式一般是 <域名/IP>[:端口号]，默认地址是 Docker Hub。
在library的镜像也就是官方镜像需要：名称
如果是个人的镜像需要：用户名/软件名
标签：如果不显式指定TAG，则默认会选择latest标签，这会下载仓库中最新版本的镜像
```

一般来说，镜像的latest标签意味着该镜像的内容会跟踪最新的非稳定版本而发布，内容是不稳定的。从稳定性上考虑，不要在生产环境中忽略镜像的标签信息或使用默认的latest标记的镜像。

```shell
[root@localhost ~]# docker pull centos
#该命令相当于docker pull registry.docker-cn.com/library/ centos:latest命令，即从默认的注册服务器Docker Hub Registry中的centos仓库来下载标记为latest的镜像。
[root@localhost ~]# docker pull ubuntu:14.04
#镜像可以发布为不同的版本，这种机制我们称之为标签（Tag）。
```

上传镜像：docker  push  [OPTIONS]  NAME[:TAG]

列出本地所有镜像：docker images [OPTIONS]  [REPOSITORY[:TAG]]

```text
仓库名称，标签，镜像 ID、创建时间，镜像大小。镜像ID是唯一标识
dockerhub显示的镜像大小是压缩的
```

查看镜像层次：docker history

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

给镜像打tag：

```shell
[root@qfedu.com ~]# docker tag daocloud.io/ubuntu daocloud.io/ubuntu:v1
```

打包镜像：

```shell
[root@node1 ~]# docker save centos > /opt/centos.tar.gz    #导出镜像
[root@node1 ~]# docker load < /opt/centos.tar.gz           #导入镜像
```

### 容器

#### 创建、运行

创建并运行一个容器：

```shell
docker run [参数] 镜像名
#参数：
-i	捕获标准输入输出,交互式操作
-t	分配一个终端或控制台
--restart=always	自动重启，这样每次docker重启后仓库容器也会自动启动
--name="mycontainer"	为容器分配一个名字，不指定会随机分配一个名称,容器的名称是唯一的
--net="bridge": 指定容器的网络连接类型，支持如下：
     bridge / host / none / container:<name|id>
-d	容器运行在后台，此时所有I/O数据只能通过网络资源或共享卷组进行交互
-p	指定本地端口映射到容器端口
-h	指定主机名
-v	指定挂载一个本地的已有目录到容器中去作为数据卷
```

```shell
[root@localhost ~]# docker run -d -p 88:80 --name web -v /src/webapp:/opt/webapp training/webapp python app.py
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

attach

使用attach命令有时候并不方便。当多个窗口同时用attach命令连到同一个容器的时候，所有窗口都会同步显示。当某个窗口因命令阻塞时，其他窗口也无法执行操作了。

```shell
[root@qfedu.com ~]# docker attach 容器id //前提是容器创建时必须指定了交互shell
```

exec(最推荐方式)

交互型任务：

```shell
[root@qfedu.com ~]# docker exec -it 容器id /bin/bash
root@68656158eb8e:/[root@qfedu.com ~]# ls
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

#### 文件数据管理

默认情况下，容器内创建的所有文件都存储在可写容器层上。

数据卷是一个或多个容器专门指定绕过Union File System，为持续性或共享数据提供一些有用的功能：

```text
1.数据卷 可以在容器之间共享和重用。
2.对 数据卷 的修改会立马生效。
3.对 数据卷 的更新，不会影响镜像。
4.数据卷 默认会一直存在，即使容器被删除。保护数据不被删除。
注意：数据卷 的使用，类似于 Linux 下对目录或文件进行 mount，镜像中的被指定为挂载点的目录中的文件会隐藏掉，能显示和看的是挂载的 数据卷，而不是被作为挂载点儿的目录中原来的内容。
```

操作：

Docker启动的时候可以通过-v选项添加数据卷，实现将主机上的目录或者文件挂载到容器中

```text
-v host-dir:container-dir:[rw|wo]
-v container-dir:[rw|wo]
-v volume-name:container-dir:[rw|wo]
host-dir：表示主机上的目录，如果不存在，Docker 会自动在主机上创建该目录。必须是绝对路径。
container-dir：表示容器内部对应的目录，如果该目录不存在，Docker 也会在容器内部创建该目录。
volume-name：表示卷名，如果该卷不存在，docker将自动创建。
rw|ro：用于控制volume的读写权限。
```

（1）

创建一个容器

```shell
在docker run 命令中使用-v选项可以在容器中创建数据卷。多次使用-v选项可以创建多个数据卷
[root@docker01 ~]# docker run -itd -P -v /test:/data --name myhttp httpd
```

进入容器

```shell
[root@docker01 ~]# docker exec -it myhttp /bin/bash
[root@28d56ec39d28 /]# df -h
```

在宿主机/text目录创建文件，观察容器内/data目录下内容变化

查看容器挂载的数据卷

```shell
[root@docker01 ~]# docker inspect myhttp
```

删除容器，宿主机/text目录下未发生变化

（2）

```shell
docker run -itd -P -v /data --name myhttp httpd
docker exec -it myhttp /bin/bash
df -h
```

看到容器内出现了/data,使用docker volume ls查询，发现多了一个本地卷：

f343bc68303155c111bea58a907131d9fe1751bc8ef6528cc53d5b7dce292dee

使用docker volume inspect查询到如下的挂下点目录：

/var/lib/docker/volumes/f343bc68303155c111bea58a907131d9fe1751bc8ef6528cc53d5b7dce292dee/_data

当在上述目录下创建test.txt文件后，容器内也查询到该新增文件。

删除容器后，宿主机上的目录及内容也未发生任何变化。

(3)

```shell
docker run -itd -P -v my_volume:/data --name myhttp httpd
docker exec -it myhttp /bin/bash
ll -a /data
```

docker自动创建了卷：my_volume，并且这个卷对应的宿主机的挂载点是：

/var/lib/docker/volumes/my_volume/_data

```shell
docker volume ls
docker volume inspect my_volume
docker inspect myhttp查看mount参数
```

（4）

在dockerfile指定数据卷

```shell
FROM debian:wheezy
VOLUME  /data
```

删除一个数据卷

```shell
[root@localhost ~]# docker volume rm yangge_vol
```

对于docker数据卷的总结：

（1） 以上几种方式都可以将宿主机目录或者文件挂载到容器。

（2） Docker提供了docker volume命令专门对volume进行管理。对于第一种方式Type为bind，是无法使用docker volume进行管理的。我们也可以使用docker volume create命令创建volume。

（3） 删除容器是如果使用docker rm container将不会删除对应的Volume。如果想要删除可以使用docker rm -v container。另外也可以单独使用docker volume rm volume_name删除volume。

（4） 对于已运行的数据卷容器，不能动态的调整其卷的挂载。Docker官方提供的方法是先删除容器，然后启动时重新挂载。



容器和宿主机之间拷贝文件

```shell
[root@qfedu.com ~]# docker cp mysql:/usr/local/bin/docker-entrypoint.sh /root
#拷贝mysql容器的文件到本地的/root
```

### 其他

查看docker信息：docker info

查看 docker 的硬盘空间使用情况：docker system df

更新容器启动项：docker container update --restart=always nginx

## 通过dockerfile创建镜像

Dockerfile 是一个文本文件，其内包含了一条条的指令(Instruction)，每一条指令构建一层，因此每一条指令的内容，就是描述该层应当如何构建。

### docker build语法

```shell
[root@qfedu.com ~]# docker build [OPTIONS] dockerfile所在路径
#选项说明
--build-arg，设置构建时的变量
--no-cache，默认false。设置该选项，将不使⽤Build Cache构建镜像
--pull，默认false。设置该选项，总是尝试pull镜像的最新版本
--compress，默认false。设置该选项，将使⽤gzip压缩构建的上下⽂
--disable-content-trust，默认true。设置该选项，将对镜像进⾏验证
--file, -f，Dockerfile的完整路径，默认值为‘PATH/Dockerfile’
--isolation，默认--isolation="default"，即Linux命名空间；其他还有process或hyperv
--label，为⽣成的镜像设置metadata
--squash，默认false。设置该选项，将新构建出的多个层压缩为⼀个新层，但是将⽆法在多个镜像之间共享新层；设置该选项，实际上是创建了新image，同时保留原有image。
--tag, -t，镜像的名字及tag，通常name:tag或者name格式；可以在⼀次构建中为⼀个镜像设置多个tag
--network，默认default。设置该选项，Set the networking mode for the RUN instructions during build
--quiet, -q ，默认false。设置该选项，Suppress the build output and print image ID on success
--force-rm，默认false。设置该选项，总是删除掉中间环节的容器
--rm，默认--rm=true，即整个构建过程成功后删除中间环节的容器
```

### dockerfile语法

Dockerfile 由一行行命令语句组成，并且支持以#开头的注释行。
一般而言，Dockerfile分为四部分：基础镜像信息、维护者信息、镜像操作指令和容器启动时执行指令。

FROM

```text
指定所创建镜像的基础镜像，如果本地不存在，则默认会去 Docker Hub下载指定镜像
任何Dockerfile 中的第一条指令必须为 FROM指令
FROM <image> [AS <name>]                                 #or
FROM <image>[:<tag>] [AS <name>]                         #or
FROM <image>[@<digest>] [AS <name>]
```

MAINTAINER

```text
LABEL maintainer="SvenDowideit@home.org.au"
```

USER

```text
指定运行容器的用户名
```

RUN

```text
指令指定将要运行并捕获到新容器映像中的命令。 这些命令包括安装软件、创建文件和目录，以及创建环境配置等。基本就是shell脚本。
RUN <command>或RUN ["executable"，"param1"，"param2"]
RUN yum update && yum install -y vim \
    python-dev #反斜线换行
注意初学docker容易出现的2个关于RUN命令的问题：
1.RUN代码没有合并。
2.每一层构建的最后一定要清理掉无关文件。
```

CMD

```text
用来指定启动容器时默认执行的命令。它支持三种格式：
CMD ["executable","param1","param2"](exec form,this is the preferred form)
CMD ["param1","param2"] (as default parameters to ENTRYPOINT)
CMD command param1 param2 (shell form)
每个Dockerfile只能有一条CMD命令
如果用户启动容器时手动指定了运行的命令（作为 run 的参数），则会覆盖掉CMD指定的命令。
```

LABEL

```text
给镜像添加信息。使用docker inspect可查看镜像的相关信息
LABEL <key>=<value> <key>=<value> <key>=<value> ...
LABEL version="1.0"
LABEL maintainer="394498036@qq.com"
```

EXPOSE

```text
声明镜像内服务所监听的端口
EXPOSE <port> [<port>/<protocol>...]
EXPOSE 22 80 8443
注意，该指令只是起到声明作用，并不会自动完成端口映射。
```

ENV

```text
指定环境变量，在镜像生成过程中会被后续 RUN 指令使用，在镜像启动的容器中也会存在
ENV <key> <value>
ENV <key>=<value> ...
```

ADD

```text
将复制指定的 <src>路径下的内容到容器中的<dest>路径下，如果是tar文件会自动解压
ADD [--chown=<user>:<group>] <src>... <dest>
ADD [--chown=<user>:<group>] ["<src>",... "<dest>"] (this form is required for paths containing whitespace)
```

COPY

```text
复制本地主机的<src>（为 Dockerfile 所在目录的相对路径、文件或目录）下的内容到镜像中的 <dest> 下。目标路径不存在时，会自动创建。
尽量使用COPY不使用ADD.
COPY [--chown=<user>:<group>] <src>... <dest>
COPY [--chown=<user>:<group>] ["<src>",... "<dest>"] (this form is required for paths containing whitespace)
```

ENTRYPOINT

```text
设置容器启动时运行的命令，所有传入值作为该命令的参数。
ENTRYPOINT ["executable", "param1", "param2"] (exec form, preferred)
ENTRYPOINT command param1 param2 (shell form)
```

WORKDIR

```text
为后续的RUN、CMD和ENTRYPOINT 指令配置工作目录
用WORKDIR，不要用RUN cd 尽量使用绝对目录！
WORKDIR /path/to/workdir
```

ENV

```text
指定环境变量
ENV <key> <value>
ENV <key>=<value> ...
```

VOLUME

### 操作系统

BusyBox

```text
BusyBox是一个集成了一百多个最常用Linux命令和工具（如cat、echo、grep、mount、telnet等）的精简工具箱，它只有几 MB的大小，很方便进行各种快速验证，被誉为“Linux系统的瑞士军刀”。BusyBox可运行于多款POSIX 环境的操作系统中，如Linux（包括Android）、Hurd、FreeBSD等。
```

Alpine

```text
Alpine镜像适用于更多常用场景，并且是一个优秀的可以适用于生产的基础系统/环境。
Alpine Docker镜像的容量非常小，仅仅只有5MB左右（Ubuntu系列镜像接近200MB），且拥有非常友好的包管理机制。官方镜像来自docker-alpine项目。
目前Docker官方已开始推荐使用Alpine替代之前的Ubuntu作为基础镜像环境。这样会带来多个好处，包括镜像下载速度加快，镜像安全性提高，主机之间的切换更方便，占用更少磁盘空间等。
ubuntu/debian -> alpine
python:2.7 -> python:2.7-alpine
ruby:2.3 -> ruby:2.3-alpine
```

Debian

Ubuntu

CentOS

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
ENTRYPOINT [ "nginx", "-g", "daemon off;"]
#构建镜像
docker build -t nginx:2020 .
#启动镜像
docker run -itd -p 88:80  -v /home/anhao1226/:/usr/local/nginx/html nginx:20201020
```

