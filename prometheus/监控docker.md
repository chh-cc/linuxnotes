监控端安装cadvisor

```shell
docker run -d \
--volume=/:/rootfs:ro \
--volume=/var/run:/var/run:rw \
--volume=/sys:/sys:ro \
--volume=/var/lib/docker:/var/lib/docker:ro \
--publish=8080:8080 \
--name=cadvisor \
google/cadvisor:latest
```

这里启动报`Failed to create summary reader for`错误了，要执行以下命令再启动容器

```shell
mount -o remount,rw '/sys/fs/cgroup'
ln -s /sys/fs/cgroup/cpu,cpuacct /sys/fs/cgroup/cpuacct,cpu
```

浏览器访问http://192.168.101.129:8080/containers/



修改配置文件

```yaml
vim /usr/local/prometheus/prometheus.yml
scrape_configs:
  - job_name: 'cadvisor'
    static_configs:
    - targets: ['192.168.101.129:8080']
    
systemctl restart prometheus
```



常用监控指标

container_cpu_load_average_10s   过去10秒容器CPU的平均负载
container_cpu_usage_seconds_total  容器在每个CPU内核上的累积占用时间(单位: 秒)
container_cpu_system_seconds_total  System CPU累积占用时间 (单位: 秒)
container_cpu_user_seconds_total  User CPU累积占用时间 (单位: 秒)
container_fs_usage_bytes  容器中文件系统的使用量(单位:字节)
container_fs_limit_bytes 容器可以使用的文件系统总量(单位:字节)
container_fs_reads_bytes_total  容器累积读取数据的总量(单位:字节)
container_fs_writes_bytes_total  容器累积写入数据的总量(单位: 字节)
container_memory_max_usage_bytes  容器的最大内存使用量(单位:字节)
container_memory_usage_bytes  容器当前的内存使用量(单位:字节)
container_spec_memory_limit_bytes  容器的内存使用量限制
machine_memory_bytes  当前主机的内存总量
container_network_receive_bytes_total   容器网络累积接收数据总量 (单位:字节)
container_network_transmit_bytes_total      容器网络累积传输数据总量 (单位: 字节)



触发器配置

cat docker_rules.yml

```yaml
groups:
- name: DockerContainers
  rules:
  - alert: ContainerKilled
    expr: time() - container_last_seen > 60
    for: 0m
    labels:
      severity: warning
    annotations:
      summary: "Docker容器被杀死，容器：$labels.instance"
      description: "{{ $value }}个容器消失了"
  - alert: ContainerAbsent
    expr: absent(container_last_seen)
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "没有容器 容器：$labels.instance"
      description: "5分钟检查容器不存在，值为：{{ $value }}"
  - alert: ContainerCpuUsage
    expr: (sum(rate(container_cpu_usage_seconds_total{name!=""}[3m])) by (instance, name) * 100) > 300
    for: 2m
    labels:
      severity: warning
    annotations:
      summary: "容器cpu使用率告警 容器：$labels.instance"
      description: "容器cpu使用率超过300%，当前值为：{{ $value }}"
  - alert: ContainerMemoryUsage
    expr: (sum(container_memory_working_set_bytes{name!=""}) by (instance, name) / sum(container_spec_memory_limit_bytes > 0) by (instance, name) * 100) > 80
    for: 2m
    labels:
      severity: warning
    annotations:
      summary: "容器内存使用率告警 容器：$labels.instance"
      description: "容器内存使用率超过80%，当前值为：{{ $value }}"
```



登录grafana，导入11600模板









