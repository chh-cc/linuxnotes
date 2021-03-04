# docker编排

当我们的业务越来越复杂时，需要多个容器相互配合，甚至需要多个主机组成容器集群才能满足我们的业务需求，这个时候就需要用到容器的编排工具了。

三种常用的编排工具：Docker Compose、Docker Swarm 和 Kubernetes。

## Docker Compose

### 安装 Docker Compose

（1）使用 curl 命令（一种发送 http 请求的命令行工具）下载 Docker Compose 的安装包：

```shell
$ sudo curl -L "https://github.com/docker/compose/releases/download/1.27.3/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

（2）修改 Docker Compose 执行权限：

```shell
$ sudo chmod +x /usr/local/bin/docker-compose
```

（3）检查 Docker Compose 是否安装成功：

```shell
$ docker-compose --version
docker-compose version 1.27.3, build 1110ad01
```

### 编写 Docker Compose 模板文件

在使用 Docker Compose 启动容器时， Docker Compose 会默认使用 docker-compose.yml 文件，有三个版本：v1，v2，v3

Docker Compose 文件主要分为三部分： services（服务）、networks（网络） 和 volumes（数据卷）。

- services（服务）：服务定义了容器启动的各项配置，就像我们执行`docker run`命令时传递的容器启动的参数一样，指定了容器应该如何启动，例如容器的启动参数，容器的镜像和环境变量等。
- networks（网络）：网络定义了容器的网络配置，就像我们执行`docker network create`命令创建网络配置一样。
- volumes（数据卷）：数据卷定义了容器的卷配置，就像我们执行`docker volume create`命令创建数据卷一样。

#### 编写 Service 配置

services 下，首先需要定义服务名称，例如你这个服务是 nginx 服务

```yaml
version: "3.8"
services:
  nginx: 
```

常用的 16 种 service 配置如下

**build：** 用于构建 Docker 镜像，类似于`docker build`命令，build 可以指定 Dockerfile 文件路径，然后根据 Dockerfile 命令来构建文件。

```yaml
build:
  ## 构建执行的上下文目录
  context: .
  ## Dockerfile 名称
  dockerfile: Dockerfile-name
```

**cap_add、cap_drop：** 指定容器可以使用到哪些内核能力（capabilities）。

```yaml
cap_add:
  - NET_ADMIN
cap_drop:
  - SYS_ADMIN
```

**command：** 用于覆盖容器默认的启动命令，它和 Dockerfile 中的 CMD 用法类似，也有两种使用方式：

```yaml
command: sleep 3000
或
command: ["sleep", "3000"]
```

**container_name：** 用于指定容器启动时容器的名称。使用格式如下：

```yaml
container_name: nginx
```

**depends_on：** 用于指定服务间的依赖关系，这样可以先启动被依赖的服务。例如，我们的服务依赖数据库服务 db，可以指定 depends_on 为 db。

```yaml
version: "3.8"
services:
  my-web:
    build: .
    depends_on:
      - db
  db:
    image: mysql
```

**devices：** 挂载主机的设备到容器中。

```yaml
devices:
  - "/dev/sba:/dev/sda"
```

**dns：** 自定义容器中的 dns 配置。

```yaml
dns:
  - 8.8.8.8
  - 114.114.114.114
```

**dns_search：** 配置 dns 的搜索域。

```yaml
dns_search:
  - svc.cluster.com
  - svc1.cluster.com
```

**entrypoint：** 覆盖容器的 entrypoint 命令。

```yaml
entrypoint: sleep 3000
或
entrypoint: ["sleep", "3000"]
```

**env_file：** 指定容器的环境变量文件，启动时会把该文件中的环境变量值注入容器中。

```yaml
env_file:
  - ./dbs.env
```

env 文件的内容格式如下：

```yaml
KEY_ENV=values
```

**environment：** 指定容器启动时的环境变量。

```yaml
environment:
  - KEY_ENV=values
```

**image：** 指定容器镜像的地址。

```yaml
image: busybox:latest
```

**pid：** 共享主机的进程命名空间，像在主机上直接启动进程一样，可以看到主机的进程信息。

```yaml
pid: "host"
```

**ports：** 暴露端口信息，使用格式为 HOST:CONTAINER，前面填写要映射到主机上的端口，后面填写对应的容器内的端口。

```yaml
ports:
  - "1000"
  - "1000-1005"
  - "8080:8080"
  - "8888-8890:8888-8890"
  - "2222:22"
  - "127.0.0.1:9999:9999"
  - "127.0.0.1:3000-3005:3000-3005"
  - "6789:6789/udp"
```

**networks：** 这是服务要使用的网络名称，对应顶级的 networks 中的配置。

```yaml
services:
  my-service:
    networks:
     - hello-network
     - hello1-network
```

**volumes：** 不仅可以挂载主机数据卷到容器中，也可以直接挂载主机的目录到容器中

```yaml
version: "3"
services:
  db:
    image: mysql:5.6
    volumes:
      - type: volume
        source: /var/lib/mysql
        target: /var/lib/mysql
或
version: "3"
services:
  db:
    image: mysql:5.6
    volumes:
      - /var/lib/mysql:/var/lib/mysql
```

#### 编写 Volume 配置

如果你想在多个容器间共享数据卷，则需要在外部声明数据卷，然后在容器里声明使用数据卷。例如我想在两个服务间共享日志目录，则使用以下配置：

```yaml
version: "3"
services:
  my-service1:
    image: service:v1
    volumes:
      - type: volume
        source: logdata
        target: /var/log/mylog
  my-service2:
    image: service:v2
    volumes:
      - type: volume
        source: logdata
        target: /var/log/mylog
volumes:
  logdata:
```

#### 编写 Network 配置

想声明一个自定义 bridge 网络配置，并且在服务中使用它，使用格式如下：

```yaml
version: "3"
services:
  web:
    networks:
      mybridge: 
        ipv4_address: 172.16.1.11
networks:
  mybridge:
    driver: bridge
    ipam: 
      driver: default
      config:
        subnet: 172.16.1.0/24
```

### Docker Compose 操作命令

可以使用`docker-compose -h`命令来查看 docker-compose 的用法

```shell
docker-compose [-f <arg>...] [options] [--] [COMMAND] [ARGS...]
```

参数：

```shell
  -f, --file FILE             指定 docker-compose 文件，默认为 docker-compose.yml
  -p, --project-name NAME     指定项目名称，默认使用当前目录名称作为项目名称
  --verbose                   输出调试信息
  --log-level LEVEL           日志级别 (DEBUG, INFO, WARNING, ERROR, CRITICAL)
  -v, --version               输出当前版本并退出
  -H, --host HOST             指定要连接的 Docker 地址
  --tls                       启用 TLS 认证
  --tlscacert CA_PATH         TLS CA 证书路径
  --tlscert CLIENT_CERT_PATH  TLS 公钥证书问价
  --tlskey TLS_KEY_PATH       TLS 私钥证书文件
  --tlsverify                 使用 TLS 校验对端
  --skip-hostname-check       不校验主机名
  --project-directory PATH    指定工作目录，默认是 Compose 文件所在路径。
```

COMMAND 为 docker-compose 支持的命令。支持的命令如下：

```shell
  build              构建服务

  config             校验和查看 Compose 文件

  create             创建服务

  down               停止服务，并且删除相关资源

  events             实时监控容器的时间信息

  exec               在一个运行的容器中运行指定命令

  help               获取帮助

  images             列出镜像

  kill               杀死容器

  logs               查看容器输出

  pause              暂停容器

  port               打印容器端口所映射出的公共端口

  ps                 列出项目中的容器列表

  pull               拉取服务中的所有镜像

  push               推送服务中的所有镜像

  restart            重启服务

  rm                 删除项目中已经停止的容器

  run                在指定服务上运行一个命令

  scale              设置服务运行的容器个数

  start              启动服务

  stop               停止服务

  top                限制服务中正在运行中的进程信息

  unpause            恢复暂停的容器

  up                 创建并且启动服务

  version            打印版本信息并退出
```

