# Docker入门

## 简介

Docker属于Linux容器的一种封装，提供简单易用的容器使用接口。它是目前最流行的 Linux 容器解决方案。

Docker将应用程序与该程序的依赖，打包在一个文件里面。运行这个文件，就会生成一个虚拟容器。程序在这个虚拟容器里运行，就好像在真实的物理机上运行一样。有了 Docker，就不用担心环境问题。

## 用途

1.提供一次性的环境。比如，本地测试他人的软件、持续集成的时候提供单元测试和构建的环境。

2.提供弹性的云服务。因为Docker容器可以随开随关，很适合动态扩容和缩容。

3.组建微服务架构。通过多个容器，一台机器可以跑多个服务，因此在本机就可以模拟出微服务架构。

## 组件

**镜像（Image）**
Docker把应用程序及其依赖，打包在image文件里面。只有通过这个文件，才能生成Docker容器。同一个image文件，可以生成多个同时运行的容器实例。

Docker 镜像是 Docker 容器运⾏时的只读模板，每⼀个镜像由⼀系列的层 (layers) 组成。Docker 使⽤ 
UnionFS 来将这些层联合到单独的镜像中。UnionFS 允许独⽴⽂件系统中的⽂件和⽂件夹(称之为分⽀)
被透明覆盖，形成⼀个单独连贯的⽂件系统。正因为有了这些层的存在，Docker 是如此的轻ᰁ。当你
改变了⼀个 Docker 镜像，⽐如升级到某个程序到新的版本，⼀个新的层会被创建。因此，不⽤替换整
个原先的镜像或者᯿新建⽴(在使⽤虚拟机的时候你可能会这么做)，只是⼀个新的层被添加或升级了。
现在你不⽤᯿新发布整个镜像，只需要升级，层使得分发 Docker 镜像变得简单和快速

镜像名称组成：registry/repo:tag

基础镜像：⼀个没有任何⽗镜像的镜像，谓之基础镜像

镜像ID：所有镜像都是通过⼀个 64 位⼗六进制字符串 （内部是⼀个 256 bit 的值）来标识的。 为简化使⽤，前
12 个字符可以组成⼀个短ID，可以在命令⾏中使⽤。短ID还是有⼀定的碰撞机率，所以服务器总是返回
⻓ID

**容器（Container）**

⼀个Docker容器包含了所有的某个应⽤运⾏所需要的环境。每⼀个Docker 容器都是从 Docker 镜像创建的。

容器有两个状态：运行（running）和退出（exited）

我们可以⽤同⼀个镜像启动多个Docker容器，这些容器启动后都是活动的，彼此还是相互隔离的。我们
对其中⼀个容器所做的变更只会局限于那个容器本身。

如果对容器的底层镜像进⾏修改，那么当前正在运⾏的容器是不受影响的，不会发⽣⾃动更新现象。

如果想更新容器到其镜像的新版本，那么必须当⼼，确保我们是以正确的⽅式构建了数据结构，否则我
们可能会导致损失容器中所有数据的后果。

**仓库（Repository）**
如百度网盘一样，我们需要一个仓库来存放镜像，Docker官方提供了公共的镜像仓库；从安全和效率的角度考虑我们也可以部署私有环境的Registry或者是Harbor。

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

获取镜像：docker pull

```shell
[root@localhost ~]# docker pull library/hello-world 
#library/hello-world是image文件在仓库里面的位置，其中library是image文件所在的组，hello-world是image文件的名字。
#Docker官方提供的image文件，都放在library组里面，所以可简写为hello-world
[root@localhost ~]# docker pull ubuntu:14.04
#镜像可以发布为不同的版本，这种机制我们称之为标签（Tag）。
```

列出本地所有镜像：docker images
删除镜像，多个镜像用空格隔开：docker rmi 镜像名或id

```shell
[root@node1 ~]# docker rmi d123f4e55e12 
#如果镜像被使用想强制删除
[root@qfedu.com ~]# docker rmi docker.io/ubuntu:latest --force
```

删除所有镜像：

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

### 容器

#### 创建、运行

创建并运行一个容器：

```shell
docker run [参数] 镜像名
#参数：
-i	捕获标准输入输出
-t	分配一个终端或控制台
--restart=always	容器随着docker engine自启动
--name	为容器分配一个名字，不指定会随机分配一个名称
-d	容器运行在后台，此时所有I/O数据只能通过网络资源或共享卷组进行交互
-p	指定映射端口
-v	指定挂载一个本地的已有目录到容器中去作为数据卷
```

```shell
[root@localhost ~]# docker run -d -p 88:80 --name web -v /src/webapp:/opt/webapp training/webapp python app.py
```



#### 查看

查看运行的容器:

```shell
[root@qfedu.com ~]# docker ps
```

查看所有的容器：

```shell
[root@qfedu.com ~]# docker ps -a
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
```

重启容器：docker restart id或名字

删除处于终止或退出状态的容器：docker rm id

强制删除还处于运行状态的容器： docker rm -f id

#### 连接容器

attach

```shell
[root@qfedu.com ~]# docker attach 容器id //前提是容器创建时必须指定了交互shell
```

exec

交互型任务：

```shell
[root@qfedu.com ~]# docker exec -it 容器id /bin/bash
root@68656158eb8e:/[root@qfedu.com ~]# ls
```

后台任务：

```shell
[root@qfedu.com ~]# docker exec 容器id touch /testfile
```

#### 监控容器的运行

logs命令来查看容器的运⾏⽇志，其中--tail选项可以指定查看最后⼏条⽇志，⽽-t选项则可以对⽇志条⽬附加时间戳。使⽤-f选项可以跟踪⽇志的输出，直到⼿动停⽌。

```shell
[root@qfedu.com ~]# docker logs App_Container //不同终端操作
```

top命令显示一个运行的容器里面的进程信息

```shell
[root@qfedu.com ~]# docker top birdben/ubuntu:v1
```

events实时输出docker服务器端的事件，包括容器的创建启动关闭等

```shell
[root@qfedu.com ~]# docker events //不同终端操作
```

#### 文件管理

容器和宿主机之间拷贝文件

```shell
[root@qfedu.com ~]# docker cp mysql:/usr/local/bin/docker-entrypoint.sh /root
#拷贝mysql容器的文件到本地的/root
```

### 其他

查看 docker 的硬盘空间使用情况
docker system df

## 容器镜像制作

### 容器文件系统打包

把正在运行的容器直接导出为tar包的镜像文件：

```shell
第⼀种：
 [root@master ~][root@qfedu.com ~]# docker export -o elated_lovelace.tar elated_lovelace
第⼆种：
 [root@master ~][root@qfedu.com ~]# docker export 容器名称 > 镜像.tar
```

导入镜像归档文件到其他宿主机：

```shell
[root@qfedu.com ~]# docker import elated_lovelace.tar elated_lovelace:v1
```

### 通过容器创建本地镜像

容器运⾏起来后，⼜在⾥⾯做了⼀些操作，并且要把操作结果保存到镜像⾥

使⽤ docker commit 指令，把⼀个正在运⾏的容器，直接提交为⼀个镜像

```shell
[root@qfedu.com ~]# docker commit 4ddf4638572d wing/helloworld:v2
#可加参数
-m 注释
-a 作者
-p 提交时暂停容器
```

### 镜像迁移

保存一台宿主机的镜像为tar包，然后导入到其他主机

打包镜像：

```shell
[root@qfedu.com ~]# docker save -o nginx.tar nginx
[root@qfedu.com ~]# docker save > nginx.tar nginx
```

导入上面打包的镜像：

```SHELL
[root@qfedu.com ~]# docker load < nginx.tar
#1.tar⽂件的名称和保存的镜像名称没有关系
#2.导⼊的镜像如果没有名称，⾃⼰打tag起名字
```

### 通过dockerfile创建镜像

docker build命令⽤于根据给定的Dockerfile和上下⽂以构建 Docker镜像。

docker build语法：

```shell
[root@qfedu.com ~]# docker build [OPTIONS] <PATH | URL | ->
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

dockerfile

Dockerfile 由一行行命令语句组成，并且支持以#开头的注释行。
一般而言，Dockerfile分为四部分：基础镜像信息、维护者信息、镜像操作指令和容器启动时执行指令。

## dockerfile封装nginx

```shell
mkdir  nginx
cd nginx
wget  http://nginx.org/download/nginx-1.15.2.tar.gz
vim Dockerfile
FROM centos	//使用官方的centos镜像作为基础镜像
RUN yum -y install gcc make pcre-devel zlib-devel tar zlib
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

## Dockerfile优化 

编译⼀个简单的nginx成功以后发现好⼏百M。 

1、RUN 命令要尽量写在⼀条⾥，每次 RUN 命令都是在之前的镜 像上封装，只会增⼤不会减⼩ 

2、每次进⾏依赖安装后使⽤yum clean all清除缓存中的rpm头⽂ 件和包⽂件 

3、选择⽐较⼩的基础镜像，⽐如：alpine