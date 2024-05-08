# Dockerfile

## dockerfile

Dockerfile 是一个文本文件，其内包含了一条条的指令(Instruction)，**每一条指令构建一层**，因此每一条指令的内容，就是描述该层应当如何构建。

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
#指定环境变量，构建期（RUN）和运行期（CMD/ENTRYPOINT）都可以生效
#环境变量可用于ADD、COPY、ENV、EXPOSE、FROM、LABEL、USER、VOLUME、WORKDIR、ONBUILD指令中。
ENV <key> <value>
ENV <key>=<value> ...

#假如你安装了JAVA程序，需要设置JAVA_HOME，那么你可以在Dockerfile中这样写：
ENV JAVA_HOME /path/to/java/dirent
ENV JAVA_HOME /usr/java/latest
ENV PATH $JAVA_HOME/bin:$PATH
ENV LANG en_us.UTF-8

#构建期不能修改ENV的值
#运行期通过docker run -e app=456就可以修改
ENV app=123
```

**ARG**

```shell
#定义一个构建变量，定义以后RUN命令使用生效
ARG parm=123456
RUN echo $parm
#不像ENV不能并排写
ARG parm=123456 msg="hello world"
#不能用于运行命令CMD和ENTRYPOINT

#使用--build-arg version=3.13.5改变
ARG version=3.13.4
FROM alpine:$version
```



**RUN**

```dockerfile
#构建时期运行的命令，比如
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

**CMD**（可以被替换）

```dockerfile
#用来指定启动容器时默认执行的命令。它支持三种格式：
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
#把上下文指定的内容复制到镜像中，如果是压缩包则自动解压，如果是远程文件则自动下载
ADD https://xxxx.tar.gz /dest/

•ADD支持Go风格的通配符，如ADD check* /testdir/
•src如果是文件，则必须包含在编译上下文中，ADD 指令无法添加编译上下文之外的文件
•如果dest结尾没有/，那么dest是目标文件名，如果dest结尾有/，那么dest是目标目录名
•如果src是一个目录，则所有文件都会被复制至dest
•如果dest不存在，则ADD 指令会创建目标目录
•应尽量减少通过ADDURL 添加remote 文件，建议使用curl 或者wget&& untar
```

ADD支持Go风格的通配符，如ADD check* /testdir/•src如果是文件，则必须包含在编译上下文中，ADD 指令无法添加编译上下文之外的文件•src如果是URL•如果dest结尾没有/，那么dest是目标文件名，如果dest结尾有/，那么dest是目标目录名•如果src是一个目录，则所有文件都会被复制至dest•如果src是一个本地压缩文件，则在ADD 的同时完整解压操作•如果dest不存在，则ADD 指令会创建目标目录•应尽量减少通过ADDURL 添加remote 文件，建议使用curl 或者wget&& untar

**COPY**

```dockerfile
#把上下文指定的内容复制到镜像中，不会自动解压和远程下载
#尽量使用COPY不使用ADD.
COPY [--chown=<user>:<group>] <src>... <dest>
COPY [--chown=<user>:<group>] ["<src>",... "<dest>"] (this form is required for paths containing whitespace)
```

**WORKDIR**

```shell
#为后续的RUN、CMD、ENTRYPOINT、COPY、ADD指令配置工作目录，类似cd，可多次切换
#用WORKDIR，不要用RUN cd 尽量使用绝对目录！
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
FROM openjdk:8-jre-alpine
LABEL maintainer="xxxxx"

COPY target/*.jar /app.jar
RUN ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && echo 'Asia/Shanghai'

ENV JAVA_OPTS=" "
ENV PARAMS=" "

ENTRYPOINT [ "sh","-c","java -Djava.security.egd=file:/dev/./urandom $JAVA_OPTS -jar /app.jar $PARAMS"]

#docker run -e JAVA_OPTS="-Xmx512m -Xms33 -" -e PARAMS="--spring.profiles=dev --server.port=8080" -jar /app/app.jar
```

## 镜像瘦身

**镜像的本质是镜像层和运行配置文件组成的压缩包**，构建镜像是通过运行 Dockerfile 中的 `RUN` 、`COPY` 和 `ADD` 等指令生成镜像层和配置文件的过程。

-  **`RUN`、`COPY` 和 `ADD` 指令会在已有镜像层的基础上创建一个新的镜像层**，执行指令产生的所有文件系统变更会在指令结束后作为一个镜像层整体提交。
-  镜像层具有 `copy-on-write` 的特性，如果去**更新其他镜像层中已存在的文件**，会先将其复制到新的镜像层中再修改，造**成双倍的文件空间占用**。
-  如果去**删除其他镜像层的一个文件**，只会在当前镜像层生成一个该文件的删除标记，并不**会减少整个镜像的实际体积**。



优化技巧：

避免产生无用的文档或缓存

1. **避免在本地保留安装缓存。**大部分包管理器会在安装时缓存下载的资源以备之后使用，以 pip 为例，会将下载的响应和构建的中间文件保存在 `~/.cache/pip` 目录，应使用 `--no-cache-dir` 选项禁用默认的缓存行为。
2.  **避免安装文档。**部分包管理器提供了选项可以不安装附带的文档，如 dnf 可使用 `--nodocs` 选项。
3.  **避免缓存包索引**。部分包管理器在执行安装之前，会尝试查询所有已启用仓库的包列表、版本等元信息缓存在本地作为索引。个别仓库的索引缓存可达到 150 M 以上。我们应该仅在安装包时查询索引，并在安装完成后清理，不应该在单独的指令中执行 `yum makecache` 这类缓存索引的命令。



- 选择最小的基础镜像，优先选alpine

- 合并RUN环节的所有指令，少一些层

- RUN期间可能安装一些程序会生成临时缓存，要自行删除

  ```shell
  RUN apt-get update && apt-get install -y \
      bzr \
      cvs \
      git \
      && rm -rf /var/lib/apt/lists/*
  ```

- 使用多段构建
