# kafka单机安装和消息测试

## 单机安装kafka和zookeeper

先安装并启动zookeeper

zookeeper两种安装方式：

1. 使用kafka自带的zookeeper（一般不推荐，zookeeper和kafka后面都是集群形式的，不推荐使用内置的）

   ```shell
   #进入kafka的解压目录
   cd /../kafka_2.122.3.0/bin
   #启动
   ./zookeeperserverstart.sh ../config/zookeeper.properties
   ```

2. 单独搭建zookeeper（推荐）

   ```shell
   yum install java-1.8.0-openjdk -y
   #解压安装包
   tar -zxf zookeeper-3.4.13.tar.gz
   cd zookeeper-3.4.13
   #在解压目录创建一个存储zookeeper数据的data目录
   mkdir data
   cp conf/zoo_sample.cfg conf/zoo.cfg
   vim conf/zoo.cfg
   ..
   dataDir=../data #修改zk的数据存储目录
   clientPort=2181 #zk的端口号
   
   #启动
   cd bin/
   ./zkServer.sh start
   ./zkServer.sh status
   
   netstat -tunlp|grep 9092
   ```

## 生产和消费消息的测试







