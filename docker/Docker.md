# Docker

理解容器的几个基本点

- 容器技术的兴起源于 PaaS 技术的普及；
- Docker 公司发布的 Docker 项目具有里程碑式的意义；
- Docker 项目通过“容器镜像”，解决了应用打包这个根本性难题。
- 容器本身没有价值，有价值的是“容器编排”

因此容器技术生态才爆发了一场关于“容器编排”的“战争”。而这次战争，最终以 Kubernetes 项目和 CNCF 社区的胜利而告终

## 简介

容器其实是一种**沙盒技术**，沙盒就是能够像一个个集装箱，把你的应用**“装”**起来的技术，这样应用与应用之间就有了**边界**而不互相干扰，而被装进集装箱的应用也方便**搬动**，这也是PASS最理想的状态。



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

## 架构和原理

### **镜像（Image）**

镜像是一个**只读**的文件和文件夹组合。它包含了容器运行时所需要的所有基础文件和配置信息，是容器启动的基础。

实际上image是由**一层层的文件系统组成**的，这种层级的文件系统称为UnionFS。

docker相关的本地资源都存放在/var/lib/docker目录下，其中containers目录下每个序列都是一个镜像

```shell
[root@docker containers]# ll
total 0
drwx------ 4 root root 237 Apr  8 23:35 01cd68ccc61b021dba0e20e226ac7db40e62ba9233522445b29f1fa1a7669113
drwx------ 4 root root 237 Apr  8 23:35 32d661fd565451624446149dde18500f0e1ca702f93a286ad84b6c86e67985d7
drwx------ 4 root root 237 Apr  8 23:22 408136b5a5aeddef2e7368d4106f8b3040875538d9f872a8291fbbf0dcb058f6
```

Docker image来源：

```text
（1）可以基于Dockerfile从无到有的构建；
（2）也可以基于Dockerfile从已有的image创建新的image；
（3）也可以基于容器生成新的image；
（4）甚至也可以直接下载别人的image。
```

镜像名称——registry/repository:tag

```shell
docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
ubuntu     			latest              2c4c54db1c77        4 months ago        192 MB

#ubuntu是一个repository，同时这个repository下有一系列打了tag的image，iamge的标记是GUID，为了方便可以通过repository:tag来引用
#registry：存储镜像数据，提供镜像的拉取和上传功能
#registry中镜像是通过repository来组织的，而每个repository又包含了若干个image
```

基础镜像：

```text
⼀个没有任何⽗镜像的镜像，谓之基础镜像
```

### **容器（Container）**

容器是镜像的运行实体。镜像是静态的只读文件，而容器带有运行时需要的可写文件层，并且容器中的进程属于运行状态。即**容器运行着真正的应用进程。容器有初建、运行、停止、暂停和删除五种状态。**

### **仓库（Repository）**

我们需要一个仓库来存放镜像，Docker官方提供了公共的镜像仓库；从安全和效率的角度考虑我们也可以部署私有环境的Registry或者是Harbor。

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

## 核心底层技术

容器的核心在于**通过对进程的隔离和限制，从而为其创造出一个边界。**

### Namespaces（隔离）

进入一个容器，执行ps：

```shell
/ # ps
PID   USER     TIME  COMMAND
    1 root      0:00 /bin/sh
    6 root      0:00 ps
```

通过Linux的namespace机制， 容器里这两个进程已经**被docker隔离在一个跟宿主机完全不同的世界中**。实际上它们在宿主机的操作系统里，进程号不是1和6。



简单说，**namespace是linux创建进程的可选参数。**

在 Linux 系统中**创建线程**的系统调用是 **clone()**，这个系统调用就会为我们创建一个新的进程，并且返回它的进程号 pid。而当我们用 clone() 系统调用创建一个新进程时，就可以在参数中指定 CLONE_NEWPID 参数这时，**新创建的这个进程将会“看到”一个全新的进程空间**，在这个进程空间里，它的 PID 是 1。之所以说“看到”，是因为这只是一个“障眼法”，在宿主机真实的进程空间里，这个进程的 PID 还是真实的数值，比如 100。

多次执行上面的 clone() 调用，这样就会创建多个 PID Namespace，而每个 Namespace 里的应用进程，都会认为自己是当前容器里的第 1 号进程。它们既看不到宿主机里真正的进程空间，也看不到其他 PID Namespace 里的具体情况。

除了我们刚刚用到的 PID Namespace，Linux 操作系统还提供了 Mount、UTS、IPC、Network 和 User 这些 Namespace，用来对各种不同的进程上下文进行“障眼法”操作。

| namespace | 系统调用参数  | 隔离内容                   | 内核版本 |
| --------- | ------------- | -------------------------- | -------- |
| UTS       | CLONE_NEWUTS  | 主机名和域名               | 2.6.19   |
| IPC       | CLONE_NEWIPC  | 信息量，消息队列和共享内存 | 2.6.19   |
| PID       | CLONE_NEWPID  | 进程编号                   | 2.6.24   |
| Network   | CLONE_NEWNET  | 网络设备、网络栈、端口等   | 2.6.29   |
| Mount     | CLONE_NEWNS   | 挂载点（文件系统）         | 2.4.19   |
| User      | CLONE_NEWUSER | 用户和用户组               | 3.8      |

​                       

Docker 容器其实就是**在创建容器进程时，指定了这个进程所需要启用的一组 Namespace 参数**。这样，容器就只能“看”到**当前 Namespace 所限定的资源、文件、设备、状态，或者配置**。而对于宿主机以及其他不相关的程序，它就完全看不到了。

**所以说，容器，其实是一种特殊的进程而已。**一般不推荐在一个容器运行多个进程，如果有类似需求，可以通过额外 的进程管理机制，比如 supervisord 来管理所运行的进程。

在 Linux 内核中，有很多资源和对象是不能被 Namespace 化的，最典型的例子就是：时间。

### Control groups（限制）

**Linux Cgroups 就是 Linux 内核中用来限制进程能使用的资源上限**，包括CPU、内存、磁盘读写速率、网络带宽等。

- Cgroups以文件系统的方式暴露给用户使用，/sys/fs/cgroup 下面有很多诸如 cpuset、cpu、 memory 这样的子目录，也叫子系统
- 使用时，在每个子系统下新建一个控制组（子目录下新建一个目录，操作系统会自动为其填充资源限制文件），将需要限制的进程的PID填写入tasks文件中

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

### rootfs（文件系统）

**关于mount namespace：**

Mount Namespace和其他Namespace的不同：如果只是启用Mount Namespace，容器进程仍能看到宿主机上全部文件

因为Mount Namespace 修改的，是容器进程对文件系统“挂载点”的认知。而宿主机无法感知容器的挂载行为。**它对容器进程视图的改变，一定是伴随着挂载操作（mount）才能生效。**

但总不能每个目录都使用mount namespace然后再挂载一遍？ 这个太麻烦了



**关于容器镜像：**

- 容器进程启动后，使用pivot_root或者chroot命令重新挂载进程的根目录"/"，一般会在根目录下挂载**完整操作系统的\*文件系统\***
- 挂载在容器根目录上、用来为容器进程提供隔离后执行环境的文件系统就是**容器镜像**，也叫做**根文件系统（rootfs）**
- rootfs只包含操作系统的文件、配置和目录，不含操作系统内核
- rootfs提供了容器运行的所有依赖，故才有它的的重要特性——一致性。



所以，一个最常见的 rootfs，或者说容器镜像，会包括如下所示的一些目录和文件

```shell
#在容器中执行ls
$ ls /
bin dev etc home lib lib64 mnt opt proc root run sbin sys tmp usr var
```



**通过结合使用 Mount Namespace 和 rootfs，容器就能够为进程构建出一个完善的文件系统隔离环境。**

对docker来说，它最核心的原理实际上就是为待创建的用户进程：

1. 启用 Linux Namespace 配置；
2. 设置指定的 Cgroups 参数；
3. 切换进程的根目录（Change Root）。



**关于联合文件系统：**

在 rootfs 的基础上，Docker 公司创新性地提出了使用多个增量 rootfs 联合挂载一个完整 rootfs 的方案，这就是容器镜像中**“层”**的概念。



联合挂载(union mount)：将多个不同位置目录A、B挂载到同一个目录C下，原目录A、B下所有内容都会合并到新目录C下，在C中的修改会在A、B中生效

- `mount -t aufs -o dirs=./A:./B none ./C`

常见的有vfs、deviceMapper、overlay、overlay2、aufs，通过`docker info`命令可以查看docker采用的是哪种union fs，目前新版本docker一般都用overlay2



容器镜像的分层结构：

- Dockerfile中的每一条指令，都会为镜像生成一个层，即一个增量rootfs

- 用命令`docker image insepct <image-name>:<tag-name>`可以看到“RootFS”字段下镜像的分层信息

```shell
$ docker image inspect ubuntu:latest
...
     "RootFS": {
      "Type": "layers",
      "Layers": [
        "sha256:f49017d4d5ce9c0f544c...",
        "sha256:8f2b771487e9d6354080...",
        "sha256:ccd4d61916aaa2159429...",
        "sha256:c01d74f99de40e097c73...",
        "sha256:268a067217b5fe78e000..."
      ]
    }
    
#可以看到，这个 Ubuntu 镜像，实际上由五个层组成。这五个层就是五个增量 rootfs，每一层都是 Ubuntu 操作系统文件与目录的一部分；而在使用镜像时，Docker 会把这些增量联合挂载在一个统一的挂载点上，这个挂载点就是 /var/lib/docker/aufs/mnt/
```



镜像的多个层会被联合挂载成为一个完整的文件系统，分为三部分：

<img src="https://gitee.com/c_honghui/picture/raw/master/img/20220322003004.png" alt="在这里插入图片描述" style="zoom:67%;" />

**第一部分，只读层（镜像层）。**

它是这个容器的 rootfs 最下面的五层，对应的正是 ubuntu:latest 镜像的五层。不希望被程序修改的内容，**以只读方式挂载**（ro+wh，即 readonly+whiteout，至于什么是 whiteout，我下面马上会讲到）。

**第二部分，可读写层。**

用户的增删改操作都记录在这里

修改操作只作用在本层，不会实际修改只读层的内容，要修改的文件会先被复制到可读写层然后再修改。被联合挂载时，只读层的相同文件会被可读写层覆盖，这叫做copy-on-write

其中的删除动作不会真正删除文件，而只是标记某文件不可见，术语叫做whiteout，从而不影响只读层的内容

**第三部分，Init 层。**

夹在只读层和读写层之间。Init 层专门用来存放 /etc/hosts、/etc/resolv.conf 等信息。

用户执行 docker commit 只会提交可读写层，所以是不包含该层内容。

最终，这 7 个层都被联合挂载到 /var/lib/docker/aufs/mnt 目录下，表现为一个完整的 Ubuntu 操作系统供容器使用。

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

删除后台所有停止的容器

```shell
sudo docker rm $(sudo docker ps -a -q)
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

logs命令来查看后台容器的输出和⽇志，其中--tail选项可以指定查看最后⼏条⽇志，⽽-t选项则可以对⽇志条⽬附加时间戳。使⽤-f选项可以跟踪⽇志的输出，直到⼿动停⽌。

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

## 网络

所谓“网络栈”，即容器自己的network namespace，包括了：网卡(Network Interface)、回环设备(Loopback Device)、路由表(Routing Table)和 iptables 规则。

容器直接共享宿主机网络，监听的就是宿主机的80端口：

```shell
$ docker run –d –net=host --name nginx-host nginx
```

虽然可以为容器提供良好的网络性能，但也会不可避免地引入共享网络资源的问题，比如端口冲突。所以， **在大多数情况下，我们都希望容器进程能使用自己 Network Namespace 里的网络栈，即：拥有属于自己的 IP 地址和端口。**

那么这个被隔离的容器进程，该如何跟其他 Network Namespace 里的容器进程进行交互呢？



而为了实现上述目的， Docker 项目会默认在宿主机上创建一个名叫 **docker0 的网桥**，凡是连接在 docker0 网桥上的容器，就可以通过它来进行通信。

这时候，我们就需要使用一种名叫 **Veth Pair的虚拟设备**了，用作连接不同 Network Namespace 的“网线”。

当启动了一个容器后，可以在宿主机查看网络设备信息：

```shell
# 在宿主机上
$ ifconfig
...
#网桥
docker0   Link encap:Ethernet  HWaddr 02:42:d8:e4:df:c1  
          inet addr:172.17.0.1  Bcast:0.0.0.0  Mask:255.255.0.0
          inet6 addr: fe80::42:d8ff:fee4:dfc1/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:309 errors:0 dropped:0 overruns:0 frame:0
          TX packets:372 errors:0 dropped:0 overruns:0 carrier:0
 collisions:0 txqueuelen:0 
          RX bytes:18944 (18.9 KB)  TX bytes:8137789 (8.1 MB)
          
#容器的Veth Pair
veth9c02e56 Link encap:Ethernet  HWaddr 52:81:0b:24:3d:da  
          inet6 addr: fe80::5081:bff:fe24:3dda/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:288 errors:0 dropped:0 overruns:0 frame:0
          TX packets:371 errors:0 dropped:0 overruns:0 carrier:0
 collisions:0 txqueuelen:0 
          RX bytes:21608 (21.6 KB)  TX bytes:8137719 (8.1 MB)
          
$ brctl show
bridge name bridge id  STP enabled interfaces
docker0  8000.0242d8e4dfc1 no  veth9c02e56
```

同宿主机不同容器通信：

![img](https://gitee.com/c_honghui/picture/raw/master/img/20220325111520.png)



宿主机访问容器：

数据包根据路由规则转发到docker0，然后被转发到对应的Veth Pair，最后出现在容器里

![img](https://gitee.com/c_honghui/picture/raw/master/img/20220325112435.png)



容器连接另外一台宿主机：

![img](https://gitee.com/c_honghui/picture/raw/master/img/20220325113212.png)



所以说， **当你遇到容器连不通“外网”的时候，你都应该先试试 docker0 网桥能不能 ping 通，然后查看一下跟 docker0 和 Veth Pair 设备相关的 iptables 规则是不是有异常，往往就能够找到问题的答案了。**



跨主机容器间通信：

不同宿主机的docker0网桥是无法互通的，所以连接在这些网桥的容器也没法互通。

我们可以通过Overlay Network(覆盖网络)技术，创建一个整个集群”公用“的网桥，把集群所有容器都接进来，就可以互相通信。

![img](https://gitee.com/c_honghui/picture/raw/master/img/20220325142210.png)



网络模式主要有四种，这四种网络模式基本满足了我们单机容器的所有场景。：

```text
null 空网络模式：可以帮助我们构建一个没有网络接入的容器环境，以保障数据安全。
bridge 桥接模式：可以打通容器与容器间网络通信的需求。
host 主机网络模式：可以让容器内的进程共享主机网络，从而监听或修改主机网络。
container 网络模式：可以将两个容器放在同一个网络命名空间内，让两个业务通过 localhost 即可实现访问。
```

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



