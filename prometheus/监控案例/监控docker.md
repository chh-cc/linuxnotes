# 监控docker

cAdvisor （Container Advisor） ：用于收集正在运行的容器资源使用和性能信息。 

项目地址：https://github.com/google/cadvisor



在本地运行CAdvisor也非常简单，直接运行一下命令即可：

```shell
docker run \
  --volume=/:/rootfs:ro \
  --volume=/var/run:/var/run:rw \
  --volume=/sys:/sys:ro \
  --volume=/var/lib/docker/:/var/lib/docker:ro \
  --publish=8080:8080 \
  --detach=true \
  --name=cadvisor \
  google/cadvisor:latest
```

在Prometheus配置文件添加监控端：

```yaml
- job_name: cadvisor
  static_configs:
  - targets:
    - 192.168.31.72:8080
```

通过访问[http://ip:8080](http://localhost:8080/)可以查看，当前主机上容器的运行状态

访问http://ip:8080/metrics即可获取到标准的Prometheus监控样本输出

下面表格中列举了一些CAdvisor中获取到的典型监控指标：

| 指标名称                               | 类型    | 含义                                         |
| -------------------------------------- | ------- | -------------------------------------------- |
| container_cpu_load_average_10s         | gauge   | 过去10秒容器CPU的平均负载                    |
| container_cpu_usage_seconds_total      | counter | 容器在每个CPU内核上的累积占用时间 (单位：秒) |
| container_cpu_system_seconds_total     | counter | System CPU累积占用时间（单位：秒）           |
| container_cpu_user_seconds_total       | counter | User CPU累积占用时间（单位：秒）             |
| container_fs_usage_bytes               | gauge   | 容器中文件系统的使用量(单位：字节)           |
| container_fs_limit_bytes               | gauge   | 容器可以使用的文件系统总量(单位：字节)       |
| container_fs_reads_bytes_total         | counter | 容器累积读取数据的总量(单位：字节)           |
| container_fs_writes_bytes_total        | counter | 容器累积写入数据的总量(单位：字节)           |
| container_memory_max_usage_bytes       | gauge   | 容器的最大内存使用量（单位：字节）           |
| container_memory_usage_bytes           | gauge   | 容器当前的内存使用量（单位：字节             |
| container_spec_memory_limit_bytes      | gauge   | 容器的内存使用量限制                         |
| machine_memory_bytes                   | gauge   | 当前主机的内存总量                           |
| container_network_receive_bytes_total  | counter | 容器网络累积接收数据总量（单位：字节）       |
| container_network_transmit_bytes_total | counter | 容器网络累积传输数据总量（单位：字节）       |

当能够正常采集到cAdvisor的样本数据后，可以通过以下表达式计算容器的CPU使用率：

sum(irate(container_cpu_usage_seconds_total{image!=""}[1m])) without (cpu)

查询容器内存使用量（单位：字节）:

container_memory_usage_bytes{image!=""}

查询容器网络接收量速率（单位：字节/秒）：

sum(rate(container_network_receive_bytes_total{image!=""}[1m])) without (interface)

查询容器网络传输量速率（单位：字节/秒）：

sum(rate(container_network_transmit_bytes_total{image!=""}[1m])) without (interface)

查询容器文件系统读取速率（单位：字节/秒）：

sum(rate(container_fs_reads_bytes_total{image!=""}[1m])) without (device)

查询容器文件系统写入速率（单位：字节/秒）：

sum(rate(container_fs_writes_bytes_total{image!=""}[1m])) without (device)



仪表盘模板ID：193

![image-20210603002019543](https://gitee.com/c_honghui/picture/raw/master/img/20210603002019.png)

添加选择多节点按钮：

![image-20210603002101220](https://gitee.com/c_honghui/picture/raw/master/img/20210603002101.png)

图表增加筛选条件:

![image-20210603002138910](https://gitee.com/c_honghui/picture/raw/master/img/20210603002138.png)



