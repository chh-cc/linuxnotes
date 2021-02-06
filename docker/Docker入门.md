# Docker入门

## 简介

Docker属于Linux容器的一种封装，提供简单易用的容器使用接口。它是目前最流行的 Linux 容器解决方案。

Docker将应用程序与该程序的依赖，打包在一个文件里面。运行这个文件，就会生成一个虚拟容器。程序在这个虚拟容器里运行，就好像在真实的物理机上运行一样。有了 Docker，就不用担心环境问题。

## 用途

1.提供一次性的环境。比如，本地测试他人的软件、持续集成的时候提供单元测试和构建的环境。

2.提供弹性的云服务。因为Docker容器可以随开随关，很适合动态扩容和缩容。

3.组建微服务架构。通过多个容器，一台机器可以跑多个服务，因此在本机就可以模拟出微服务架构。

## 组件

镜像（Image）
Docker把应用程序及其依赖，打包在image文件里面。只有通过这个文件，才能生成Docker容器。同一个image文件，可以生成多个同时运行的容器实例。

容器（Container）
docker是通过容器来运行业务的，就像运行一个kvm虚拟机是一样的。容器其实就是从镜像创建的一个实例。 我们可以对容器进行增删改查，容器之间也是相互隔离的。和虚拟机最大的区别就是一个是虚拟的一个是隔离的。

仓库（Repository）
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
```

查看镜像：docker images
删除镜像：docker rmi
[root@node1 ~]# docker rmi d123f4e55e12 

### 容器

随机启动nginx容器 随机端口
docker run -d -p nginx
#-d 在后台运行容器

指定映射端口：
docker run -d -p 127.0.0.1:81:80 --name mynginx nginx

进入容器：docker exec -it id /bin/bash
#-i 打开标准输入 -t 分配一个伪终端 exec 在容器内执行任意命令

显示一个运行的容器里面的进程信息 
docker top id

停止容器：docker stop id
[root@localhost ~]# docker stop 2b85a89be878
强制停止容器： docker kill id
重启容器：docker restart id

删除处于终止或退出状态的容器：docker rm id

强制删除还处于运行状态的容器： docker rm -f id

查看容器启动情况:docker ps

查看容器日志：docker logs  id

批量停止容器
docker kill $(docker ps -aq)

批量删除容器 docker rm -f $(docker ps -aq)

## 其他

查看 docker 的硬盘空间使用情况
docker system df