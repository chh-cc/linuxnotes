## 安装node-exporter

node-exporter可以采集机器（物理机、虚拟机、云主机等）的监控指标数据，能够采集到的指标包括CPU,内存，磁盘，网络，文件数等信息。



```yaml
[root@xianchaomaster1 ~]# kubectl create ns monitor-sa
手动解压：
[root@xianchaomaster1 ~]# docker load -i node-exporter.tar.gz

cat node-export.yaml
apiVersion: apps/v1
kind: DaemonSet #可以保证k8s集群的每个节点都运行完全一样的pod
metadata:
  name: node-exporter
  namespace: monitor-sa
labels:
  name: node-exporter
spec:
  selector:
    matchLabels:
      name: node-exporter
  template:
    metadata:
      labels:
        name: node-exporter
    spec:
      hostPID: true
      hostIPC: true
      hostNetwork: true
# hostNetwork、hostIPC、hostPID都为True时，表示这个Pod里的所有容器，会直接使用宿主机的网络，直接与宿主机进行IPC（进程间通信）通信，可以看到宿主机里正在运行的所有进程。加入了hostNetwork:true会直接将我们的宿主机的9100端口映射出来，从而不需要创建service在我们的宿主机上就会有一个9100的端口
      containers:
      - name: node-exporter
        image: prom/node-exporter:v0.16.0
        ports:
        - containerPort: 9100
        resources:
          requests:
            cpu: 0.15 #这个容器运行至少需要0.15核cpu
 	      securityContext:
          privileged: true #开启特权模式
        args:
        - --path.procfs #配置挂载宿主机（node节点）的路径
        - /host/proc
        - --path.sysfs #配置挂载宿主机（node节点）的路径
        - /host/sys
        - --collector.filesystem.ignored-mount-points
        - '"^/(sys|proc|dev|host|etc)($|/)"' #通过正则表达式忽略某些文件系统挂载点的信息收集、
        volumeMounts:
        - name: dev
          mountPath: /host/dev
        - name: proc
          mountPath: /host/proc
        - name: sys
          mountPath: /host/sys
        - name: rootfs
          mountPath: /rootf
#将主机/dev、/proc、/sys这些目录挂在到容器中，这是因为我们采集的很多节点数据都是通过这些文件来获取系统信息的。
      tolerations:
      - key: "node-role.kubernetes.io/master"
        operator: "Exists"
        effect: "NoSchedule"
      volumes:
      - name: proc
        hostPath:
          path: /proc
      - name: dev
        hostPath:
          path: /dev
      - name: sys
        hostPath:
          path: /sys
      - name: rootfs
        hostPath:
          path: /
          
          
 kubectl apply -f node-export.yaml
```

通过node-exporter采集数据,默认的监听端口是9100，可以看到当前主机获取到的所有监控数据

```shell
curl http://主机ip:9100/metrics #node-export

# HELP node_cpu_seconds_total Seconds the cpus spent in each mode.
# TYPE node_cpu_seconds_total counter
node_cpu_seconds_total{cpu="0",mode="idle"} 72963.37
node_cpu_seconds_total{cpu="0",mode="iowait"} 9.35
node_cpu_seconds_total{cpu="0",mode="irq"} 0
node_cpu_seconds_total{cpu="0",mode="nice"} 0
node_cpu_seconds_total{cpu="0",mode="softirq"} 151.4
node_cpu_seconds_total{cpu="0",mode="steal"} 0
node_cpu_seconds_total{cpu="0",mode="system"} 656.12
node_cpu_seconds_total{cpu="0",mode="user"} 267.1
```

\#HELP：解释当前指标的含义，上面表示在每种模式下node节点的cpu花费的时间，以s为单位

\#TYPE：说明当前指标的数据类型，上面是counter类型

node_cpu_seconds_total{cpu="0",mode="idle"}：cpu0上idle进程占用CPU的总时间

