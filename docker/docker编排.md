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

services 下，首先需**要定义服务名称**，例如你这个服务是 nginx 服务

```yaml
version: "3.8"
services:
  nginx: 
```

常用的 16 种 service 配置如下

build： 用于构建 Docker 镜像，类似于`docker build`命令，build 可以指定 Dockerfile 文件路径，然后根据 Dockerfile 命令来构建文件。

```yaml
build:
  ## 构建执行的上下文目录
  context: .
  ## Dockerfile 名称
  dockerfile: Dockerfile-name
```

cap_add、cap_drop： 指定容器可以使用到哪些内核能力（capabilities）。

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

**depends_on：** 用于指定服务间的依赖关系，这样可以**先启动被依赖的服务**。例如，我们的服务依赖数据库服务 db，可以指定 depends_on 为 db。

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

devices： 挂载主机的设备到容器中。

```yaml
devices:
  - "/dev/sba:/dev/sda"
```

dns： 自定义容器中的 dns 配置。

```yaml
dns:
  - 8.8.8.8
  - 114.114.114.114
```

dns_search： 配置 dns 的搜索域。

```yaml
dns_search:
  - svc.cluster.com
  - svc1.cluster.com
```

entrypoint： 覆盖容器的 entrypoint 命令。

```yaml
entrypoint: sleep 3000
或
entrypoint: ["sleep", "3000"]
```

env_file： 指定容器的环境变量文件，启动时会把该文件中的环境变量值注入容器中。

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

pid： 共享主机的进程命名空间，像在主机上直接启动进程一样，可以看到主机的进程信息。

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

networks： 这是服务要使用的网络名称，对应顶级的 networks 中的配置。

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

docker-compose up -d [service1] [service2] ...

如果拉取镜像出现以下错误，可能需要登录dockerhub
ERROR: error pulling image configuration: Get https://d2iks1dkcwqcbx.cloudfront.net/docker/registry/v2/blobs/sha256/8a/8a5a7863221ad86b3dc8d6f7068f0b2c2f28597766e9285ecca5921eee7166d1/data?Expires=1617175007&Signature=B7bEQef-2X0U6OYtuazMOpg0lLqt8IiVIQ4XEwM8q0VpPQiSSSzZbtP4X355Yt6RltKRQMRY~qUcdV932HBeDolssKtjzwf3~eF-m-AQjx0-UfEzKUx5Ul3P9zDZjPgfdV7MmC7SdufyvlLaWEl7UDuThkeQZKEo8rVZ7MMS2WuCkzFSBOoKXtFiBmy2xHGmRDAlD3poWYV88RfO3cA9YkQdfSxLjwqF8PoHW-KJkE~wZ84mFP~g7fxCXwymgUEhlCmlIyZipX3fC9Lt0-LBah2cmgWuzlLvePQrGqNJvpXyoRVcxl2uf0FmkGS~HrBluM3ELxJcwxeQ6-EufFZu5A__&Key-Pair-Id=APKAIVAVKHB6SNHJAJQQ: read tcp 192.168.71.131:36856->13.225.100.166:443: read: connection reset by peer
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

## Docker Swarm

Docker Compose，它能够在单一节点上管理和编排多个容器，当我们的服务和容器数量较小时可以使用 Docker Compose 来管理容器。

Docker Swarm ，我们可以用它来管理规模更大的容器集群。

### Swarm 的架构

Swarm 的架构整体分为管理节点（Manager Nodes）和工作节点（Worker Nodes）

![image-20210305102719573](https://gitee.com/c_honghui/picture/raw/master/img/20210305102725.png)

管理节点： 管理节点负责接受用户的请求，用户的请求中包含用户定义的容器运行状态描述，然后 Swarm 负责调度和管理容器，并且努力达到用户所期望的状态。

工作节点： 工作节点运行执行器（Executor）负责执行具体的容器管理任务（Task），例如容器的启动、停止、删除等操作。

### Swarm 核心概念

**Swarm 集群**
Swarm 集群是一组被 Swarm 统一管理和调度的节点，被 Swarm纳管的节点可以是物理机或者虚拟机。其中一部分节点作为管理节点，负责集群状态的管理和协调，另一部分作为工作节点，负责执行具体的任务来管理容器，实现用户服务的启停等功能。

**节点**
Swarm 集群中的每一台物理机或者虚拟机称为节点。节点按照工作职责分为管理节点和工作节点，管理节点由于需要使用 Raft 协议来协商节点状态，生产环境中通常建议将管理节点的数量设置为奇数个，一般为 3 个、5 个或 7 个。

**服务**
服务是为了支持容器编排所提出的概念，它是一系列复杂容器环境互相协作的统称。一个服务的声明通常包含容器的启动方式、启动的副本数、环境变量、存储、配置、网络等一系列配置，用户通过声明一个服务，将它交给 Swarm，Swarm 负责将用户声明的服务实现。

服务分为全局服务（global services）和副本服务（replicated services）。

全局服务：每个工作节点上都会运行一个任务，类似于 Kubernetes 中的 Daemonset。

副本服务：按照指定的副本数在整个集群中调度运行。

**任务**
任务是集群中的最小调度单位，它包含一个真正运行中的 Docker 容器。当管理节点根据服务中声明的副本数将任务调度到节点时，任务则开始在该节点启动和运行，当节点出现异常时，任务会运行失败。此时调度器会把失败的任务重新调度到其他正常的节点上正常运行，以确保运行中的容器副本数满足用户所期望的副本数。

**服务外部访问**
由于容器的 IP 只能在集群内部访问到，而且容器又是用后马上销毁，这样容器的 IP 也会动态变化，因此容器集群内部的服务想要被集群外部的用户访问到，服务必须要映射到主机上的固定端口。Swarm 使用入口负载均衡（ingress load balancing）的模式将服务暴露在主机上，该模式下，每一个服务会被分配一个公开端口（PublishedPort），你可以指定使用某个未被占用的公开端口，也可以让 Swarm 自动分配一个。

Swarm 集群的公开端口可以从集群内的任意节点上访问到，当请求达到集群中的一个节点时，如果该节点没有要请求的服务，则会将请求转发到实际运行该服务的节点上，从而响应用户的请求。公有云的云负载均衡器（cloud load balancers）可以利用这一特性将流量导入到集群中的一个或多个节点，从而实现利用公有云的云负载均衡器将流量导入到集群中的服务。

### 搭建 Swarm 集群

要求：

Docker 版本大于 1.12，推荐使用最新稳定版 Docker；

主机需要开放一些端口（TCP：2377 UDP:4789 TCP 和 UDP:7946）。

环境：

| 节点名        | 节点IP         | 角色    |
| ------------- | -------------- | ------- |
| swarm-manager | 192.168.31.100 | manager |
| swarm-node1   | 192.168.31.101 | node    |
| swarm-node2   | 192.168.31.102 | node    |
| swarm-node3   | 192.168.31.103 | node    |

生产环境中推荐使用至少三个 manager 作为管理节点。

1. 初始化集群

Docker 1.12 版本后， Swarm 已经默认集成到了 Docker 中，因此我们可以直接使用 Docker 命令来初始化 Swarm

```shell
docker swarm init --advertise-addr <YOUR-IP>
#advertise-addr 一般用于主机有多块网卡的情况，如果你的主机只有一块网卡，可以忽略此参数。
```

在管理节点上，通过以下命令初始化集群：

```shell
$ docker swarm init
Swarm initialized: current node (1ehtnlcf3emncktgjzpoux5ga) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-1kal5b1iozbfmnnhx3kjfd3y6yqcjjjpcftrlg69pm2g8hw5vx-8j4l0t2is9ok9jwwc3tovtxbp 192.168.31.100:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
```

集群初始化后， Swarm 会提示我们当前节点已经作为一个管理节点了，并且提示了如何把一台主机加入集群成为工作节点。

2. 加入工作节点

按照第一步集群初始化后输出的提示，只需要复制其中的命令即可，然后在剩余的三台工作节点上分别执行如下命令：

```shell
$ docker swarm join --token SWMTKN-1-1kal5b1iozbfmnnhx3kjfd3y6yqcjjjpcftrlg69pm2g8hw5vx-8j4l0t2is9ok9jwwc3tovtxbp 192.168.31.100:2377
This node joined a swarm as a worker.
```

默认加入的节点为工作节点，如果是生产环境，我们可以使用`docker swarm join-token manager`命令来查看如何加入管理节点：

```shell
$ docker swarm join-token manager
To add a manager to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-1kal5b1iozbfmnnhx3kjfd3y6yqcjjjpcftrlg69pm2g8hw5vx-8fq89jxo2axwggryvom5a337t 192.168.31.100:2377
```

3. 节点查看

节点添加完成后，我们使用以下命令可以查看当前节点的状态：

```shell
$ ]# docker node ls
ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION

1ehtnlcf3emncktgjzpoux5ga *   swarm-manager       Ready               Active              Leader              19.03.12

pn7gdm847sfzydqhcv3vma97y *   swarm-node1         Ready               Active                                        19.03.12

4dtc9pw5quyjs5yf25ccgr8uh *   swarm-node2         Ready               Active                                        19.03.12

est7ww3gngna4u7td22g9m2k5 *   swarm-node3         Ready               Active                                        19.03.12
```

到此，一个包含 1 个管理节点，3 个工作节点的 Swarm 集群已经搭建完成。

### 使用 Swarm

Swarm 集群中常用的服务部署方式有以下两种。

#### （1）通过 docker service 命令创建服务

创建服务:

```shell
$ docker service create --replicas 1 --name hello-world nginx

24f9ng83m9sq4ml3e92k4g5by
overall progress: 1 out of 1 tasks
1/1: running   [==================================================>]
verify: Service converged
```

查看已经启动的服务:

```shell
$ docker service ls

ID                  NAME                  MODE                REPLICAS            IMAGE               PORTS
24f9ng83m9sq        hello-world           replicated          1/1                 nginx:latest
```

删除服务：

```shell
$ docker service rm hello-world

hello-world
```

生产环境中，我们推荐使用 docker-compose 模板文件来部署服务，这样服务的管理会更加方便并且可追踪，而且可以同时创建和管理多个服务，更加适合生产环境中依赖关系较复杂的部署模式。

#### （2）通过 docker stack 命令创建服务



```yaml
version: '3'

services:
   mysql:
     image: mysql:5.7
     volumes:
       - mysql_data:/var/lib/mysql
     restart: always
     environment:
       MYSQL_ROOT_PASSWORD: root
       MYSQL_DATABASE: mywordpress
       MYSQL_USER: mywordpress
       MYSQL_PASSWORD: mywordpress

   wordpress:
     depends_on:
       - mysql
     image: wordpress:php7.4
     deploy:
       mode: replicated
       replicas: 2
     ports:
       - "8080:80"
     restart: always
     environment:
       WORDPRESS_DB_HOST: mysql:3306
       WORDPRESS_DB_USER: mywordpress
       WORDPRESS_DB_PASSWORD: mywordpress
       WORDPRESS_DB_NAME: mywordpress

volumes:
    mysql_data: {}
```

我在服务模板文件中添加了 deploy 指令，并且指定使用副本服务（replicated）的方式启动两个 WordPress 实例。

启动服务：

```shell
$ docker stack deploy -c docker-compose.yml wordpress

Ignoring unsupported options: restart

Creating network wordpress_default
Creating service wordpress_mysql
Creating service wordpress_wordpress
```

执行完以上命令后，我们成功启动了两个服务：

1. MySQL 服务，默认启动了一个副本。
2. WordPress 服务，根据我们 docker-compose 模板的定义启动了两个副本。

```shell
$ docker service ls

ID                  NAME                  MODE                REPLICAS            IMAGE               PORTS
v8i0pzb4e3tc        wordpress_mysql       replicated          1/1                 mysql:5.7
96m8xfyeqzr5        wordpress_wordpress   replicated          2/2                 wordpress:php7.4    *:8080->80/tcp
```

Swarm 已经为我们成功启动了一个 MySQL 服务，并且启动了两个 WordPress 实例。WordPress 实例通过 8080 端口暴露在了主机上，我们通过访问集群中的任意节点的 IP 加 8080 端口即可访问到 WordPress 服务。例如，我们访问[http://192.168.31.101:8080](http://192.168.31.101:8080/)即可成功访问到我们搭建的 WordPress 服务。