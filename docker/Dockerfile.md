# Dockerfile

## dockerfile

Dockerfile 是一个文本文件，其内包含了一条条的指令(Instruction)，每一条指令构建一层，因此每一条指令的内容，就是描述该层应当如何构建。

把jar包制作成docker镜像就需要通过dockerfile制作成镜像

### dockerfile语法

Dockerfile 由一行行命令语句组成，并且支持以#开头的注释行。
一般而言，Dockerfile分为四部分：基础镜像信息、维护者信息、镜像操作指令和容器启动时执行指令。

**FROM**

```dockerfile
选择基础镜像，推荐alpine
FROM [--platform=<platform>] <image>[@<digest>] [AS <name>]

例子：
FROM openjdk:8-jre

注意：如果是从私有仓库拉取镜像，如果有配置权限，那么需要先登录到私有仓库。
```

LABEL

```text
没有实际作用。常用于声明镜像作者
LABEL <key>=<value> <key>=<value> <key>=<value> ...
LABEL maintainer="SvenDowideit@home.org.au"
之前直接通过指令MAINTAINER指定，这种写法已经废弃掉了
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

**ENV**

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
执行命令，比如
RUN mkdir -p /home/admin/app/ && \
         wget http://edas-hz.oss-cn-hangzhou.aliyuncs.com/demo/1.0/hello-edas-0.0.1-SNAPSHOT.jar -O /home/admin/app/hello-edas-0.0.1-SNAPSHOT.jar
    
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

如果指定了ENTRYPOINT，则CMD将只是提供参数，传递给ENTRYPOINT
```

官方有一段关于`CMD`和`ENTRYPOINT`组合出现时的结果：

https://docs.docker.com/engine/reference/builder/#understand-how-cmd-and-entrypoint-interact

|                                | 没有 ENTRYPOINT            | ENTRYPOINT exec_entry p1_entry | ENTRYPOINT [“exec_entry”, “p1_entry”]          |
| :----------------------------- | :------------------------- | :----------------------------- | :--------------------------------------------- |
| **没有 CMD**                   | 报错                       | /bin/sh -c exec_entry p1_entry | exec_entry p1_entry                            |
| **CMD [“exec_cmd”, “p1_cmd”]** | exec_cmd p1_cmd            | /bin/sh -c exec_entry p1_entry | exec_entry p1_entry exec_cmd p1_cmd            |
| **CMD [“p1_cmd”, “p2_cmd”]**   | p1_cmd p2_cmd              | /bin/sh -c exec_entry p1_entry | exec_entry p1_entry p1_cmd p2_cmd              |
| **CMD exec_cmd p1_cmd**        | /bin/sh -c exec_cmd p1_cmd | /bin/sh -c exec_entry p1_entry | exec_entry p1_entry /bin/sh -c exec_cmd p1_cmd |

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

例子：
ADD target/*.jar /application.jar

•ADD支持Go风格的通配符，如ADD check* /testdir/
•src如果是文件，则必须包含在编译上下文中，ADD 指令无法添加编译上下文之外的文件
•src如果是URL
•如果dest结尾没有/，那么dest是目标文件名，如果dest结尾有/，那么dest是目标目录名
•如果src是一个目录，则所有文件都会被复制至dest
•如果src是一个本地压缩文件，则在ADD 的同时完整解压操作
•如果dest不存在，则ADD 指令会创建目标目录
•应尽量减少通过ADDURL 添加remote 文件，建议使用curl 或者wget&& untar
```

ADD支持Go风格的通配符，如ADD check* /testdir/•src如果是文件，则必须包含在编译上下文中，ADD 指令无法添加编译上下文之外的文件•src如果是URL•如果dest结尾没有/，那么dest是目标文件名，如果dest结尾有/，那么dest是目标目录名•如果src是一个目录，则所有文件都会被复制至dest•如果src是一个本地压缩文件，则在ADD 的同时完整解压操作•如果dest不存在，则ADD 指令会创建目标目录•应尽量减少通过ADDURL 添加remote 文件，建议使用curl 或者wget&& untar

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



多段构建

有效减少镜像层级的方式

```dockerfile
FROM golang:1.16-alpine AS build
RUN apk add --no-cache git
RUN go get github.com/golang/dep/cmd/dep

COPY Gopkg.lock Gopkg.toml /go/src/project/
WORKDIR /go/src/project/
RUN dep ensure -vendor-only

COPY . /go/src/project/
RUN go build -o /bin/project（只有这个二进制文件是产线需要的，其他都是waste）

FROM scratch
COPY --from=build /bin/project /bin/project
ENTRYPOINT ["/bin/project"]
CMD ["--help"]
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

### 制作jar包镜像

```shell
FROM lizhenliang/java:8-jdk-alpine
LABEL maintainer www.aliangedu.cn
RUN  sed -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/g' /etc/apk/repositories && \
     apk add -U tzdata && \
     ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
COPY ./target/portal-service.jar ./
COPY skywalking /skywalking
EXPOSE 8080
CMD java -jar -javaagent:/skywalking/skywalking-agent.jar=agent.service_name=ms-portal,agent.instance_name=$(echo $HOSTNAME | awk -F- '{print $1"-"$NF}'),collector.backend_service=192.168.31.90:11800 /portal-service.jar
```

## 优化dockerfile

- 从Docker 1.10起，`COPY`、`ADD`和`RUN`语句会在镜像中添加新层。
  
- 可以把几个RUN命令合并成一句
  
- 使用多阶段构建（多个from）

  多阶段构建的应用场景及优势就是为了降低复杂性并减少依赖，避免镜像包含不必要的软件包

  简单来说，多阶段构建就是允许一个`Dockerfile`中出现多条`FROM`指令，只有最后一条`FROM`指令中指定的基础镜像作为本次构建镜像的基础镜像，其它的阶段都可以认为是只为中间步骤

  每一条`FROM`指令都表示着多阶段构建过程中的一个构建阶段，后面的构建阶段可以拷贝利用前面构建阶段的产物

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

  



