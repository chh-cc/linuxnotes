# node_exporter采集主机数据

Prometheus Server并不能直接服务监控特定的目标，其主要任务负责数据的收集，存储并且对外提供数据查询支持。因此为了能够能够监控到某些东西，如主机的CPU使用率，我们需要使用到Exporter。**Prometheus周期性的从Exporter暴露的HTTP服务地址（通常是/metrics）拉取监控样本数据。**

端口：9100

node_exporter：用于监控Linux系统的指标采集器。
常用指标：
• CPU
• 内存
• 硬盘
• 网络流量
• 文件描述符
• 系统负载
• 系统服务  

## 安装node_exporter

在被监控的服务器上下载安装node exporter

```shell
wget https://github.com/prometheus/node_exporter/releases/download/v1.1.1/node_exporter-1.1.1.linux-amd64.tar.gz
tar xzf node_exporter-1.1.1.linux-amd64.tar.gz
mv node_exporter-1.1.1.linux-amd64 /usr/local/node_exporter

nohup ./node_exporter &
ss -tnl	#9100端口
State       Recv-Q Send-Q                                    Local Address:Port                                                   Peer Address:Port              
LISTEN      0      128                                                   *:22                                                                *:*                  
LISTEN      0      128                                                [::]:10851                                                          [::]:*                  
LISTEN      0      128                                                [::]:9100                                                           [::]:*                  
LISTEN      0      128                                                [::]:22                                                             [::]:*  
```

```shell
# cat >/usr/lib/systemd/system/node_exporter.service  <<EOF

[Unit]
Description=node_exporter

[Service]
ExecStart=/usr/local/node_exporter/node_exporter \
--web.listen-address=:9100 \
--collector.systemd \
--collector.systemd.unit-whitelist="(ssh|docker|rsyslog|redis-server).service" \
--collector.textfile.directory=/usr/local/node_exporter/textfile.collected
Restart=on-failure

[Install]
WantedBy=multi-user.target
EOF

# systemctl enable node_exporter
# systemctl start node_exporter.service
```



在prometheus服务器修改配置文件并重启服务

```shell
vim /usr/local/prometheus/prometheus.yml
#配置被监控端
scrape_configs:
  # 采集prometheus监控数据
  - job_name: 'prometheus'
    static_configs:
    - targets: ['localhost:9090']
  # 采集node exporter监控数据
  - job_name: 'linux server'
    static_configs:
    - targets:
      - 10.1.96.3:9100
```

当我们需要采集不同的监控指标(例如：主机、MySQL、Nginx)时，我们只需要运行相应的监控采集程序，并且让Prometheus Server知道这些Exporter实例的访问地址。

在Prometheus中，每一个暴露监控样本数据的HTTP服务称为一个实例。例如在当前主机上运行的node exporter可以被称为一个**实例(Instance)**。
而一组用于相同采集目的的实例，或者同一个采集进程的多个副本则通过一个一个**任务(Job)**进行管理。

常用的node exporter指标

```text
node_cpu_seconds_total

node_memory_MemTotal_bytes

node_filesystem_size_bytes{mount_point=PATH}

node_system_unit_state{name=}

node_vmstat_pswpin：系统每秒从硬盘读到内存的字节数

node_vmstat_pswpout：系统每秒从内存写到硬盘的字节数
```

##　监控系统资源的方法论

系统资源监控用得较多的方法是"USE"方法，分别表示为：Utilization（使用率）、Saturation（饱和度）、Errors（错误数）

### 1、CPU使用率监控

```shell
(1- (avg(irate(node_cpu_seconds_total{nodename=~"monitor01",mode="idle"}[5m])))) * 100

或者

100 - (avg(irate(node_cpu_seconds_total{nodename=~"monitor01",mode="idle"}[5m])) * 100)
```

### 2、内存使用率监控

```shell
(1- (node_memory_MemAvailable_bytes{nodename="monitor01"})/node_memory_MemTotal_bytes{nodename="monitor01"} ) * 100

或

(node_memory_MemTotal_bytes{nodename="monitor01"} - node_memory_MemAvailable_bytes{nodename="monitor01"})/node_memory_MemTotal_bytes{nodename="monitor01"} * 100
```

如果将Buffers和Cached也作为可用内存，则内存使用率计算公式如下：

```shell
(node_memory_MemTotal_bytes{nodename="monitor01"} - (node_memory_MemFree_bytes{nodename="monitor01"} + node_memory_Buffers_bytes{nodename="monitor01"} + node_memory_Cached_bytes{nodename="monitor01"}))/node_memory_MemTotal_bytes{nodename="monitor01"} * 100
```

上述公式的标签名用nodename来替换了instance，因为在prometheus中配置时，instance和job都按缺省取默认值。

### 3、磁盘分区使用率监控

```shell
(1- (node_filesystem_avail_bytes{nodename="monitor01",mountpoint="/"} / node_filesystem_size_bytes{nodename="monitor01",mountpoint="/"})) * 100
```

磁盘使用预测

predict_linear()函数，根据前一个时间段的值来预测未来某个时间点数据的走势。

```shell
predict_linear(node_filesystem_free_bytes{device="rootfs",nodename=~"monitor01",mountpoint="/"}[1d],24*3600) /(1024*1024*1024)
```

上面这个表达式含义是根据近1天的磁盘空闲情况，预测在明天的这个时间磁盘还空闲多少。

测试：dd if=/dev/zero of=/disktest bs=1024 count=2097152

### 4、CPU饱和度监控

CPU饱和度通常是按系统的平均负载来衡量，如观察主机CPU数量(通常按逻辑CPU来算)在一段时间内平均运行的队列长度，当平均负载小于vCPU数量则认为是正常的，若负载长时间超出CPU的数量则认为CPU饱和。

我们先来看看如何查看主机的物理CPU个数、每个物理CPU的核数、每个物理CPU有多少个逻辑CPU（购买云主机时的vcpu数或叫线程数）。

通过过滤"physical id"查看物理CPU数

`cat /proc/cpuinfo |grep "physical id" | uniq | wc -l`

通过过滤"cpu cores"查看每个物理CPU有多个核数

`cat /proc/cpuinfo | grep "cpu cores" | uniq`

通过过滤"siblings"查看每个物理CPU下有多少个逻辑CPU数

`cat /proc/cpuinfo | grep "siblings" |uniq`

或者通过过滤"processor"查看所有物理cpu下的逻辑cpu个数

`cat /proc/cpuinfo | grep "processor" | wc -l`



CPU饱和度监控

```shell
avg(node_load1{nodename="monitor01"}) by (nodename)/count(node_cpu_seconds_total{nodename="monitor01",mode="system"}) by (nodename)

或者

avg(node_load1{nodename="monitor01"})/count(node_cpu_seconds_total{nodename="monitor01",mode="system"})
```

### 5、内存饱和度

可以用下面这两个指标来评估内存饱和度。

node_vmstat_pswpin：系统每秒从swap读到内存的字节数，读取的是/proc/vmstat下的pswpin(si)，单位是KB/s 。

node_vmstat_pswpout：系统每秒从内存写到swap字节数，读取的是/proc/vmstat下的pswpout(so)，单位是KB/s。

内存饱和度计算，个人理解是上面2个指标之和大于0时表示已使用到交换分区，内存达到饱和？但如果交换分区未启用该如何计算饱和度？

### 6、磁盘IO使用率

```shell
avg(irate(node_disk_io_time_seconds_total{nodename="monitor01"}[1m])) by(nodename) * 100
```

### 7、网卡接收/发送流量监控

```shell
irate(node_network_receive_bytes_total{nodename=~'monitor01',device=~"ens5"}[5m])*8

irate(node_network_transmit_bytes_total{nodename=~'monitor01',device=~"ens5"}[5m])*8
```




