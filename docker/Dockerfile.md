# Dockerfile

## dockerfile

Dockerfile 是一个文本文件，其内包含了一条条的指令(Instruction)，每一条指令构建一层，因此每一条指令的内容，就是描述该层应当如何构建。

**生产实践中一定优先使用 Dockerfile 的方式构建镜像。**用commit创建的镜像会比较大。 使用 Dockerfile 构建镜像可以带来很多好处：

- 易于版本化管理，Dockerfile 本身是一个文本文件，方便存放在代码仓库做版本管理，可以很方便地找到各个版本之间的变更历史；
- 过程可追溯，Dockerfile 的每一行指令代表一个镜像层，根据 Dockerfile 的内容即可很明确地查看镜像的完整构建过程；
- 屏蔽构建环境异构，使用 Dockerfile 构建镜像无须考虑构建环境，基于相同 Dockerfile 无论在哪里运行，构建结果都一致。

### dockerfile语法

Dockerfile 由一行行命令语句组成，并且支持以#开头的注释行。
一般而言，Dockerfile分为四部分：基础镜像信息、维护者信息、镜像操作指令和容器启动时执行指令。

**FROM**

```dockerfile
指定所创建镜像的基础镜像，如果本地不存在，则默认会去 Docker Hub下载指定镜像
任何Dockerfile 中的第一条指令必须为 FROM指令
FROM <image> [as <name>] 
FROM centos:latest                                       #or
FROM <image>[:<tag>] [as <name>]                        
FROM centos:7.5                                          #or
FROM <image>[@<digest>] [as <name>]

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
maintainer = "SvenDowideit@home.org.au"
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

用dockerfile生成镜像：

```shell
[root@qfedu.com ~]# docker build [OPTIONS] dockerfile所在路径
#选项说明
-t, --tag list # 镜像名称
-f, --file string # 指定Dockerfile文件位置
docker build -t shykes/myapp .
docker build -t shykes/myapp -f /path/Dockerfile /path
docker build -t shykes/myapp http://www.example.com/Dockerfile
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

## 优化dockerfile

- 从Docker 1.10起，`COPY`、`ADD`和`RUN`语句会在镜像中添加新层。
  - 可以把几个RUN命令合并成一句

- 使用多阶段构建（多个from）

  - 单阶段构建

    ```dockerfile
    #通过dockerfile打包应用
    FROM node:8
    
    EXPOSE 3000
    
    WORKDIR /app
    COPY package.json index.js ./
    RUN npm install
    
    CMD ["npm", "start"]
    #构建镜像
    docker build -t node-vanilla .
    #可以看到最终的镜像上添加了五个新层：Dockerfile里的每条语句一层。
    docker history node-vanilla
    IMAGE          CREATED BY                                      SIZE
    075d229d3f48   /bin/sh -c #(nop)  CMD ["npm" "start"]          0B
    bc8c3cc813ae   /bin/sh -c npm install                          2.91MB
    bac31afb6f42   /bin/sh -c #(nop) COPY multi:3071ddd474429e1…   364B
    500a9fbef90e   /bin/sh -c #(nop) WORKDIR /app                  0B
    78b28027dfbf   /bin/sh -c #(nop)  EXPOSE 3000                  0B
    b87c2ad8344d   /bin/sh -c #(nop)  CMD ["node"]                 0B
    <missing>      /bin/sh -c set -ex   && for key in     6A010…   4.17MB
    <missing>      /bin/sh -c #(nop)  ENV YARN_VERSION=1.3.2       0B
    <missing>      /bin/sh -c ARCH= && dpkgArch="$(dpkg --print…   56.9MB
    <missing>      /bin/sh -c #(nop)  ENV NODE_VERSION=8.9.4       0B
    <missing>      /bin/sh -c set -ex   && for key in     94AE3…   129kB
    <missing>      /bin/sh -c groupadd --gid 1000 node   && use…   335kB
    <missing>      /bin/sh -c set -ex;  apt-get update;  apt-ge…   324MB
    <missing>      /bin/sh -c apt-get update && apt-get install…   123MB
    <missing>      /bin/sh -c set -ex;  if ! command -v gpg > /…   0B
    <missing>      /bin/sh -c apt-get update && apt-get install…   44.6MB
    <missing>      /bin/sh -c #(nop)  CMD ["bash"]                 0B
    <missing>      /bin/sh -c #(nop) ADD file:1dd78a123212328bd…   123MB
    ```

  - 多阶段构建

    ```dockerfile
    FROM node:8 as build
    
    WORKDIR /app
    COPY package.json index.js ./
    RUN npm install
    
    FROM node:8
    
    COPY --from=build /app /
    EXPOSE 3000
    CMD ["index.js"]
    
    docker build -t node-multi-stage .
    
    docker history node-multi-stage
    IMAGE          CREATED BY                                      SIZE
    331b81a245b1   /bin/sh -c #(nop)  CMD ["index.js"]             0B
    bdfc932314af   /bin/sh -c #(nop)  EXPOSE 3000                  0B
    f8992f6c62a6   /bin/sh -c #(nop) COPY dir:e2b57dff89be62f77…   1.62MB
    b87c2ad8344d   /bin/sh -c #(nop)  CMD ["node"]                 0B
    <missing>      /bin/sh -c set -ex   && for key in     6A010…   4.17MB
    <missing>      /bin/sh -c #(nop)  ENV YARN_VERSION=1.3.2       0B
    <missing>      /bin/sh -c ARCH= && dpkgArch="$(dpkg --print…   56.9MB
    <missing>      /bin/sh -c #(nop)  ENV NODE_VERSION=8.9.4       0B
    <missing>      /bin/sh -c set -ex   && for key in     94AE3…   129kB
    <missing>      /bin/sh -c groupadd --gid 1000 node   && use…   335kB
    <missing>      /bin/sh -c set -ex;  apt-get update;  apt-ge…   324MB
    <missing>      /bin/sh -c apt-get update && apt-get install…   123MB
    <missing>      /bin/sh -c set -ex;  if ! command -v gpg > /…   0B
    <missing>      /bin/sh -c apt-get update && apt-get install…   44.6MB
    <missing>      /bin/sh -c #(nop)  CMD ["bash"]                 0B
    <missing>      /bin/sh -c #(nop) ADD file:1dd78a123212328bd…   123MB
    
    #可以看到镜像小了一点点
    docker images | grep node-
    node-multi-stage   331b81a245b1   678MB
    node-vanilla       075d229d3f48   679MB
    ```

- 使用Distroless移除容器中的所有累赘

  - 目前的镜像不仅含有Node.js，还含有`yarn`、`npm`、`bash`以及大量其他二进制文件。“Distroless”镜像只包含应用程序及其运行时依赖。不包含包管理器、Shell以及其他标准Linux发行版中能找到的其他程序。

    ```dockerfile
    FROM node:8 as build
    
    WORKDIR /app
    COPY package.json index.js ./
    RUN npm install
    
    FROM gcr.io/distroless/nodejs
    
    COPY --from=build /app /
    EXPOSE 3000
    CMD ["index.js"]
    
    docker images | grep node-distroless
    node-distroless   7b4db3b7f1e5   76.7MB
    
    #不过由于Distroless是原始操作系统的精简版本，不包含额外的程序。容器里并没有Shell！
    ```

- 使用apline镜像，如果用到Glibc，可以是slim如node:slim、python:slim

  ```dockerfile
  FROM node:8 as build
  
  WORKDIR /app
  COPY package.json index.js ./
  RUN npm install
  
  FROM node:8-alpine
  
  COPY --from=build /app /
  EXPOSE 3000
  CMD ["npm", "start"]
  
  docker images | grep node-alpine
  node-alpine   aa1f85f8e724   69.7MB
  
  docker exec -ti 9d8e97e307d7 sh
  / #
  ```

  



