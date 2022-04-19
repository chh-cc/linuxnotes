# kubeadm安装

本次安装采用的是Kubeadm安装工具，安装版本是K8s 1.20+，采用的系统为CentOS 7.9，其中Master节点3台，Node节点2台，高可用工具采用HAProxy + KeepAlived

## 节点规划

| 主机名            | IP                  | 角色       | 配置          |
| ----------------- | ------------------- | ---------- | ------------- |
| k8s-master01 ~ 03 | 192.168.101.145~147 | master节点 | 2C2G 40G      |
| k8s-node01 ~ 02   | 192.168.101.148~149 | node节点   | 2C2G 40G      |
| k8s-master-lb     | 192.168.101.200     | VIP        | VIP不占用机器 |

| 信息        | 备注           |
| ----------- | -------------- |
| 系统版本    | CentOS 7.4     |
| Docker版本  | 19.03.x        |
| K8s版本     | 1.20.0         |
| Pod网段     | 172.168.0.0/16 |
| Service网段 | 10.96.0.0/12   |

## 基本配置

所有节点配置hosts

```shell
[root@k8s-master01 ~]# cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6

192.168.101.145	k8s-master01
192.168.101.146	k8s-master02
192.168.101.147	k8s-master03
192.168.101.148	k8s-node01
192.168.101.149	k8s-node02
192.168.101.200	k8s-master_lb
```

yum源配置

```shell
curl -o /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-7.repo
yum install -y yum-utils device-mapper-persistent-data lvm2
yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
sed -i -e '/mirrors.cloud.aliyuncs.com/d' -e '/mirrors.aliyuncs.com/d' /etc/yum.repos.d/CentOS-Base.repo
```

必备工具安装

```shell
yum install wget jq psmisc vim net-tools telnet yum-utils device-mapper-persistent-data lvm2 git -y
```

所有节点关闭防火墙、selinux、dnsmasq、swap。服务器配置如下：

```shell
systemctl disable --now firewalld 
systemctl disable --now dnsmasq
systemctl disable --now NetworkManager

setenforce 0
sed -i 's#SELINUX=enforcing#SELINUX=disabled#g' /etc/sysconfig/selinux
sed -i 's#SELINUX=enforcing#SELINUX=disabled#g' /etc/selinux/config
```

关闭swap分区

```shell
swapoff -a && sysctl -w vm.swappiness=0
sed -ri '/^[^#]*swap/s@^@#@' /etc/fstab
```

安装ntpdate

```shell
rpm -ivh http://mirrors.wlnmp.com/centos/wlnmp-release-centos.noarch.rpm
yum install ntpdate -y
```

所有节点同步时间。时间同步配置如下：

```shell
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
echo 'Asia/Shanghai' >/etc/timezone
ntpdate time2.aliyun.com
```

加入到crontab

```shell
*/5 * * * * ntpdate time2.aliyun.com
```

所有节点配置limit：

```shell
ulimit -SHn 65535

vim /etc/security/limits.conf
# 末尾添加如下内容
* soft nofile 655360
* hard nofile 131072
* soft nproc 655350
* hard nproc 655350
* soft memlock unlimited
* hard memlock unlimited
```

Master01节点免密钥登录其他节点：

```shell
ssh-keygen -t rsa
for i in k8s-master02 k8s-master03 k8s-node01 k8s-node02;do ssh-copy-id -i .ssh/id_rsa.pub $i;done
```

下载安装所有的源码文件：

```shell
cd /root/ ; git clone https://github.com/dotbalo/k8s-ha-install.git
```

所有节点升级系统并重启：

```shell
yum update -y --exclude=kernel* && reboot
```

centos7升级内核到4.18+

master01节点下载内核然后传到其他节点

```shell
wget http://193.49.22.109/elrepo/kernel/el7/x86_64/RPMS/kernel-ml-devel-4.19.12-1.el7.elrepo.x86_64.rpm
wget http://193.49.22.109/elrepo/kernel/el7/x86_64/RPMS/kernel-ml-4.19.12-1.el7.elrepo.x86_64.rpm
for i in k8s-master02 k8s-master03 k8s-node01 k8s-node02;do scp kernel-ml-devel-4.19.12-1.el7.elrepo.x86_64.rpm kernel-ml-4.19.12-1.el7.elrepo.x86_64.rpm $i:/root;done
```

所有节点安装内核

```shell
cd /root && yum localinstall -y kernel-ml*
```

所有节点更改内核启动顺序

```shell
grub2-set-default 0 && grub2-mkconfig -o /etc/grub2.cfg
grubby --args="user_namespace.enable=1" --update-kernel="$(grubby --default-kernel)"
```

检查默认内核是不是4.19

```shell
grubby --default-kernel
```

所有节点重启，然后检查内核

```shell
uname -a
```

## 内核配置

centos7需要升级内核至4.18+

master01节点下载内核然后传到其他节点

```shell
wget http://193.49.22.109/elrepo/kernel/el7/x86_64/RPMS/kernel-ml-devel-4.19.12-1.el7.elrepo.x86_64.rpm
wget http://193.49.22.109/elrepo/kernel/el7/x86_64/RPMS/kernel-ml-4.19.12-1.el7.elrepo.x86_64.rpm
for i in k8s-master02 k8s-master03 k8s-node01 k8s-node02;do scp kernel-ml-devel-4.19.12-1.el7.elrepo.x86_64.rpm kernel-ml-4.19.12-1.el7.elrepo.x86_64.rpm $i:/root;done
```

所有节点安装内核

```shell
cd /root && yum localinstall -y kernel-ml*
```

所有节点更改内核启动顺序

```shell
grub2-set-default 0 && grub2-mkconfig -o /etc/grub2.cfg
grubby --args="user_namespace.enable=1" --update-kernel="$(grubby --default-kernel)"
```

检查默认内核是不是4.19

```shell
grubby --default-kernel
```

所有节点重启，然后检查内核

```shell
uname -a
```

所有节点安装ipvsadm

```shell
yum install ipvsadm ipset sysstat conntrack libseccomp -y
```

所有节点配置ipvs模块，在内核4.19+版本nf_conntrack_ipv4已经改为nf_conntrack,4.18以下使用nf_conntrack_ipv4

```shell
vim /etc/modules-load.d/ipvs.conf
ip_vs
ip_vs_lc
ip_vs_wlc
ip_vs_rr
ip_vs_wrr
ip_vs_lblc
ip_vs_lblcr
ip_vs_dh
ip_vs_sh
ip_vs_fo
ip_vs_nq
ip_vs_sed
ip_vs_ftp
ip_vs_sh
nf_conntrack
ip_tables
ip_set
xt_set
ipt_set
ipt_rpfilter
ipt_REJECT
ipip

systemctl enable --now systemd-modules-load.service
```

检查是否加载

```shell
lsmod|grep -e ip_vs -e nf_conntrack
```

所有节点开启一些k8s集群必须的内核参数

```shell
cat <<EOF > /etc/sysctl.d/k8s.conf
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
fs.may_detach_mounts = 1
vm.overcommit_memory = 1
vm.panic_on_oom = 0
fs.inotify.max_user_watches=89100
fs.file-max=52706963
fs.nr_open=52706963
net.netfilter.nf_conntrack_max=2310720
net.ipv4.tcp_keepalive_time = 600
net.ipv4.tcp_keepalive_probes = 3
net.ipv4.tcp_keepalive_intvl = 15
net.ipv4.tcp_max_tw_buckets = 36000
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_max_orphans = 327680
net.ipv4.tcp_orphan_retries = 3
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_max_syn_backlog = 16384
net.ipv4.ip_conntrack_max = 65536
net.ipv4.tcp_timestamps = 0
net.core.somaxconn = 16384
EOF

sysctl --system
```

## 基本组件安装

所有节点安装Docker-ce 19.03（经过官方验证的版本）

```shell
yum install docker-ce-19.03.* -y
```

由于新版kubelet建议使用systemd，所以可以把docker的CgroupDriver改成systemd

```shell
mkdir /etc/docker
cat > /etc/docker/daemon.json<<EOF
{
"exec-opts":["native.cgroupdriver=systemd"]
}
EOF
```

所有节点设置开机自启动Docker

```shell
systemctl daemon-reload && systemctl enable --now docker
```

安装k8s组件

```shell
yum list kubeadm.x86_64 --showduplicates | sort -r
```

所有节点安装最新版本kubeadm（用于生产环境小版本大于5，如1.19.5）

```shell
yum install kubeadm -y
```

默认配置的pause镜像使用gcr.io仓库，国内可能无法访问，所以这里配置Kubelet使用阿里云的pause镜像：

```shell
cat >/etc/sysconfig/kubelet<<EOF
KUBELET_EXTRA_ARGS="--cgroup-driver=systemd --pod-infra-container-image=registry.cn-hangzhou.aliyuncs.com/google_containers/pause-amd64:3.2"
EOF
```

设置Kubelet开机自启动

```shell
systemctl daemon-reload
systemctl enable --now kubelet
```

## 高可用组件安装

所有Master节点通过yum安装HAProxy和KeepAlived：

```shell
yum install keepalived haproxy -y
```

所有Master节点配置HAProxy

```shell
[root@k8s-master01 etc]# vim /etc/haproxy/haproxy.cfg 
global
  maxconn  2000
  ulimit-n  16384
  log  127.0.0.1 local0 err
  stats timeout 30s

defaults
  log global
  mode  http
  option  httplog
  timeout connect 5000
  timeout client  50000
  timeout server  50000
  timeout http-request 15s
  timeout http-keep-alive 15s

frontend monitor-in
  bind *:33305
  mode http
  option httplog
  monitor-uri /monitor

frontend k8s-master
  bind 0.0.0.0:16443
  bind 127.0.0.1:16443
  mode tcp
  option tcplog
  tcp-request inspect-delay 5s
  default_backend k8s-master

backend k8s-master
  mode tcp
  option tcplog
  option tcp-check
  balance roundrobin
  default-server inter 10s downinter 5s rise 2 fall 2 slowstart 60s maxconn 250 maxqueue 256 weight 100
  server k8s-master01	192.168.101.145:6443  check
  server k8s-master02	192.168.101.146:6443  check
  server k8s-master03	192.168.101.147:6443  check
```

所有Master节点配置KeepAlived，配置不一样，注意区分每个节点的IP和网卡

Master01节点的配置：

```shell
[root@k8s-master01 ~]# vim /etc/keepalived/keepalived.conf 
! Configuration File for keepalived
global_defs {
    router_id LVS_DEVEL
    script_user root
    enable_script_security
}
vrrp_script chk_apiserver {
    script "/etc/keepalived/check_apiserver.sh"
    interval 5
    weight -5
    fall 2  
    rise 1
}
vrrp_instance VI_1 {
    state MASTER
    interface ens33
    mcast_src_ip 192.168.101.145
    virtual_router_id 51
    priority 101
    advert_int 2
    authentication {
        auth_type PASS
        auth_pass K8SHA_KA_AUTH
    }
    virtual_ipaddress {
        192.168.101.200
    }
    track_script {
       chk_apiserver
    }
}
```

Master02节点的配置：

```shell
! Configuration File for keepalived
global_defs {
    router_id LVS_DEVEL
	script_user root
    enable_script_security
}
vrrp_script chk_apiserver {
    script "/etc/keepalived/check_apiserver.sh"
   interval 5
    weight -5
    fall 2  
	rise 1
}
vrrp_instance VI_1 {
    state BACKUP
    interface ens33
    mcast_src_ip 192.168.101.146
    virtual_router_id 51
    priority 100
    advert_int 2
    authentication {
        auth_type PASS
        auth_pass K8SHA_KA_AUTH
    }
    virtual_ipaddress {
        192.168.101.200
    }
    track_script {
       chk_apiserver
    }
}
```

Master03节点的配置：

```shell
! Configuration File for keepalived
global_defs {
    router_id LVS_DEVEL
	script_user root
    enable_script_security
}
vrrp_script chk_apiserver {
    script "/etc/keepalived/check_apiserver.sh"
    interval 5
    weight -5
    fall 2  
	rise 1
}
vrrp_instance VI_1 {
    state BACKUP
    interface ens192
    mcast_src_ip 192.168.101.147
    virtual_router_id 51
    priority 100
    advert_int 2
    authentication {
        auth_type PASS
        auth_pass K8SHA_KA_AUTH
    }
    virtual_ipaddress {
        192.168.101.200
    }
    track_script {
       chk_apiserver
    }
}
```

所有master节点配置KeepAlived健康检查文件：

```shell
[root@k8s-master01 keepalived]# vim /etc/keepalived/check_apiserver.sh 
#!/bin/bash

err=0
for k in $(seq 1 3)
do
    check_code=$(pgrep haproxy)
    if [[ $check_code == "" ]]; then
        err=$(expr $err + 1)
        sleep 1
        continue
    else
        err=0
        break
    fi
done

if [[ $err != "0" ]]; then
    echo "systemctl stop keepalived"
    /usr/bin/systemctl stop keepalived
    exit 1
else
    exit 0
fi


chmod +x /etc/keepalived/check_apiserver.sh
启动haproxy和keepalived
[root@k8s-master01 keepalived]# systemctl daemon-reload
[root@k8s-master01 keepalived]# systemctl enable --now haproxy
[root@k8s-master01 keepalived]# systemctl enable --now keepalived
```

测试VIP

```shell
[root@k8s-master01 ~]# ping 192.168.101.200
PING 192.168.101.200 (192.168.101.200) 56(84) bytes of data.
64 bytes from 192.168.101.200: icmp_seq=1 ttl=64 time=0.027 ms
64 bytes from 192.168.101.200: icmp_seq=2 ttl=64 time=0.060 ms
64 bytes from 192.168.101.200: icmp_seq=3 ttl=64 time=0.044 ms
```

## 集群初始化

在master01节点创建kubeadm-config.yaml配置文件如下：

```yaml
vim kubeadm-config.yaml

apiVersion: kubeadm.k8s.io/v1beta2
bootstrapTokens:
- groups:
  - system:bootstrappers:kubeadm:default-node-token
  token: 7t2weq.bjbawausm0jaxury
  ttl: 24h0m0s
  usages:
  - signing
  - authentication
kind: InitConfiguration
localAPIEndpoint:
  advertiseAddress: 192.168.101.145
  bindPort: 6443
nodeRegistration:
  criSocket: /var/run/dockershim.sock
  name: k8s-master01
  taints:
  - effect: NoSchedule
    key: node-role.kubernetes.io/master
---
apiServer:
  certSANs:
  - 192.168.101.200
  timeoutForControlPlane: 4m0s
apiVersion: kubeadm.k8s.io/v1beta2
certificatesDir: /etc/kubernetes/pki
clusterName: kubernetes
controlPlaneEndpoint: 192.168.101.200:16443
controllerManager: {}
dns:
  type: CoreDNS
etcd:
  local:
    dataDir: /var/lib/etcd
imageRepository: registry.cn-hangzhou.aliyuncs.com/google_containers
kind: ClusterConfiguration
kubernetesVersion: v1.20.0
networking:
  dnsDomain: cluster.local
  podSubnet: 172.168.0.0/12
  serviceSubnet: 10.96.0.0/12
scheduler: {}
```

转换格式

```shell
kubeadm config migrate --old-config kubeadm-config.yaml --new-config new.yaml
```

将new.yaml文件复制到其他master节点，之后所有Master节点提前下载镜像，可以节省初始化时间：

```shell
kubeadm config images pull --config /root/new.yaml 
```

Master01节点初始化，初始化以后会在/etc/kubernetes目录下生成对应的证书和配置文件，之后其他Master节点加入Master01即可：

```shell
kubeadm init --config /root/new.yaml  --upload-certs

...
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

You can now join any number of the control-plane node running the following command on each as root:

  kubeadm join 192.168.101.200:16443 --token 7t2weq.bjbawausm0jaxury \
    --discovery-token-ca-cert-hash sha256:3ed23f68bbf38cbb4850cc778f840c647c238d21af6cc433ab3a97b1af3940f3 \
    --control-plane --certificate-key 3843c20e75ef54198c28d5051ef8a5a158bc5953329d47041bf9d29881a2bcb9

Please note that the certificate-key gives access to cluster sensitive data, keep it secret!
As a safeguard, uploaded-certs will be deleted in two hours; If necessary, you can use
"kubeadm init phase upload-certs --upload-certs" to reload certs afterward.

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.101.200:16443 --token 7t2weq.bjbawausm0jaxury \
    --discovery-token-ca-cert-hash sha256:3ed23f68bbf38cbb4850cc778f840c647c238d21af6cc433ab3a97b1af3940f3 
```

Master01节点配置环境变量，用于访问Kubernetes集群：

```shell
cat <<EOF >> /root/.bashrc
export KUBECONFIG=/etc/kubernetes/admin.conf
EOF
source /root/.bashrc
```

查看节点状态：

```shell
[root@k8s-master01 ~]# kubectl get node
NAME           STATUS     ROLES                  AGE     VERSION
k8s-master01   NotReady   control-plane,master   6m12s   v1.20.0
```

初始化其他master加入集群

```shell
kubeadm join 192.168.101.200:16443 --token 7t2weq.bjbawausm0jaxury \
    --discovery-token-ca-cert-hash sha256:3ed23f68bbf38cbb4850cc778f840c647c238d21af6cc433ab3a97b1af3940f3 \
    --control-plane --certificate-key 3843c20e75ef54198c28d5051ef8a5a158bc5953329d47041bf9d29881a2bcb9
```

添加node节点

```shell
kubeadm join 192.168.101.200:16443 --token 7t2weq.bjbawausm0jaxury \
    --discovery-token-ca-cert-hash sha256:3ed23f68bbf38cbb4850cc778f840c647c238d21af6cc433ab3a97b1af3940f3 
```

注：如果token过期，可以重新生成新的token

```shell
kubeadm token create --print-join-command

#master需要生成新的certificate-key
kubeadm init phase upload-certs --upload-certs
```

查看节点状态

```shell
[root@k8s-master01 ~]# kubectl get node
NAME           STATUS     ROLES                  AGE     VERSION
k8s-master01   NotReady   control-plane,master   16m     v1.20.0
k8s-master02   NotReady   control-plane,master   7m33s   v1.20.0
k8s-master03   NotReady   control-plane,master   2m1s    v1.20.0
k8s-node01     NotReady   <none>                 26s     v1.20.0
k8s-node02     NotReady   <none>                 13s     v1.20.0
```

## calico安装

以下步骤只在master01执行

```shell
cd /root/k8s-ha-install && git checkout manual-installation-v1.20.x && cd calico/
```

修改calico-etcd.yaml的以下位置

```shell
sed -i 's#etcd_endpoints: "http://<ETCD_IP>:<ETCD_PORT>"#etcd_endpoints: "https://192.168.101.145:2379,https://192.168.101.146:2379,https://192.168.101.147:2379"#g' calico-etcd.yaml


ETCD_CA=`cat /etc/kubernetes/pki/etcd/ca.crt | base64 | tr -d '\n'`
ETCD_CERT=`cat /etc/kubernetes/pki/etcd/server.crt | base64 | tr -d '\n'`
ETCD_KEY=`cat /etc/kubernetes/pki/etcd/server.key | base64 | tr -d '\n'`
sed -i "s@# etcd-key: null@etcd-key: ${ETCD_KEY}@g; s@# etcd-cert: null@etcd-cert: ${ETCD_CERT}@g; s@# etcd-ca: null@etcd-ca: ${ETCD_CA}@g" calico-etcd.yaml


sed -i 's#etcd_ca: ""#etcd_ca: "/calico-secrets/etcd-ca"#g; s#etcd_cert: ""#etcd_cert: "/calico-secrets/etcd-cert"#g; s#etcd_key: "" #etcd_key: "/calico-secrets/etcd-key" #g' calico-etcd.yaml

POD_SUBNET=`cat /etc/kubernetes/manifests/kube-controller-manager.yaml | grep cluster-cidr= | awk -F= '{print $NF}'`

sed -i 's@# - name: CALICO_IPV4POOL_CIDR@- name: CALICO_IPV4POOL_CIDR@g; s@#   value: "192.168.0.0/16"@  value: '"${POD_SUBNET}"'@g' calico-etcd.yaml
```

创建calico

```shell
kubectl apply -f calico-etcd.yaml
```

查看容器状态

```shell
kubectl get po -n kube-system

NAME                                       READY   STATUS    RESTARTS   AGE
calico-kube-controllers-5f6d4b864b-hwrzb   1/1     Running   0          5m45s
calico-node-4fqgc                          1/1     Running   0          5m45s
calico-node-68bvf                          1/1     Running   0          5m45s
calico-node-72nzj                          1/1     Running   0          5m45s
calico-node-nj2hf                          1/1     Running   0          5m45s
calico-node-svh89                          1/1     Running   0          5m45s
coredns-54d67798b7-6l74h                   1/1     Running   0          33m
coredns-54d67798b7-vj556                   1/1     Running   0          33m
etcd-k8s-master01                          1/1     Running   0          33m
etcd-k8s-master02                          1/1     Running   0          23m
etcd-k8s-master03                          1/1     Running   0          18m
kube-apiserver-k8s-master01                1/1     Running   0          33m
kube-apiserver-k8s-master02                1/1     Running   0          23m
kube-apiserver-k8s-master03                1/1     Running   0          18m
kube-controller-manager-k8s-master01       1/1     Running   1          33m
kube-controller-manager-k8s-master02       1/1     Running   0          23m
kube-controller-manager-k8s-master03       1/1     Running   0          18m
kube-proxy-64hw8                           1/1     Running   0          16m
kube-proxy-h4kh5                           1/1     Running   0          33m
kube-proxy-jcxmt                           1/1     Running   0          18m
kube-proxy-pcsl4                           1/1     Running   0          23m
kube-proxy-vz46h                           1/1     Running   0          16m
kube-scheduler-k8s-master01                1/1     Running   1          33m
kube-scheduler-k8s-master02                1/1     Running   0          23m
kube-scheduler-k8s-master03                1/1     Running   0          18m
```

## Mertics部署

通过Metrics采集节点和Pod的内存、磁盘、CPU和网络的使用率。

将Master01节点的front-proxy-ca.crt复制到所有Node节点

```shell
scp /etc/kubernetes/pki/front-proxy-ca.crt k8s-node01:/etc/kubernetes/pki/front-proxy-ca.crt
scp /etc/kubernetes/pki/front-proxy-ca.crt k8s-node02:/etc/kubernetes/pki/front-proxy-ca.crt
```

安装metrics server

```shell
cd /root/k8s-ha-install/metrics-server-0.4.x-kubeadm/

[root@k8s-master01 metrics-server-0.4.x-kubeadm]# kubectl  create -f comp.yaml 
serviceaccount/metrics-server created
clusterrole.rbac.authorization.k8s.io/system:aggregated-metrics-reader created
clusterrole.rbac.authorization.k8s.io/system:metrics-server created
rolebinding.rbac.authorization.k8s.io/metrics-server-auth-reader created
clusterrolebinding.rbac.authorization.k8s.io/metrics-server:system:auth-delegator created
clusterrolebinding.rbac.authorization.k8s.io/system:metrics-server created
service/metrics-server created
deployment.apps/metrics-server created
apiservice.apiregistration.k8s.io/v1beta1.metrics.k8s.io created
```

等待kube-system命令空间下的Pod全部启动后，查看状态

```shell
[root@k8s-master01 metrics-server-0.4.x-kubeadm]# kubectl  top node
NAME           CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%   
k8s-master01   117m         5%     1269Mi          67%       
k8s-master02   110m         5%     1268Mi          67%       
k8s-master03   92m          4%     1249Mi          66%       
k8s-node01     55m          2%     939Mi           50%       
k8s-node02     54m          2%     947Mi           50%       
```

## 安装dashboard(可以换成kuboard)

```shell
cd /root/k8s-ha-install/dashboard/

[root@k8s-master01 dashboard]# kubectl  create -f .
```

更改dashboard的svc为NodePort：

```shell
kubectl edit svc kubernetes-dashboard -n kubernetes-dashboard

...
type: NodePort
...
```

查看端口号

```shell
[root@k8s-master01 dashboard]# kubectl get svc kubernetes-dashboard -n kubernetes-dashboard
NAME                   TYPE       CLUSTER-IP    EXTERNAL-IP   PORT(S)         AGE
kubernetes-dashboard   NodePort   10.111.4.67   <none>        443:30543/TCP   3m51s
```

访问Dashboard：[https://192.168.101.200:30543](https://192.168.0.236:18282/)（请更改18282为自己的端口），选择登录方式为令牌（即token方式）

![image-20211113165825242](https://gitee.com/c_honghui/picture/raw/master/img/20211113165832.png)

查看token值：

```shell
[root@k8s-master01 dashboard]# kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep admin-user | awk '{print $1}')
Name:         admin-user-token-g67vp
Namespace:    kube-system
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: admin-user
              kubernetes.io/service-account.uid: ed264087-b89e-44cd-8d42-944d34a72085

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1066 bytes
namespace:  11 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6InJNQ3NDN1dqbU9KTXV0OFBHa1pCSVBPS1RieGE0WUgyRW1LdHRKczZyU3MifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJhZG1pbi11c2VyLXRva2VuLWc2N3ZwIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6ImFkbWluLXVzZXIiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiJlZDI2NDA4Ny1iODllLTQ0Y2QtOGQ0Mi05NDRkMzRhNzIwODUiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6a3ViZS1zeXN0ZW06YWRtaW4tdXNlciJ9.wH1a59vVNkGRaaGWccVleZO3YR9MAvSBxFZ7F64TY_adoA9pWGLae02Cz9KESgSIbi6Cex8wjUQ6QwfYBgViwzjVAuY2N9qLQfAxy6xqm_xEWe1pfQaHi_6vAfT5RmgRqtSakO8ewy22GESo6FfFHq2YnKq4F9ijeSZNmUABksMtARpQBDF9juqU3ZX4dyQCD-qQ5cqSK5TDBd-OGEeQSuQE7VjiyQLk7WONLSc15EX35wGqbkAYUGEP7eLev2OL0C9Rjz3w800IAZnRp8f33bfeOGEBavM6RSyEELT7O6aSWUSfOIWQ7ZQg5DKbzDigNfwv0TcrYRcd7WlgDrX3Bg
```

把token输入然后登录

-----

安装kuboard

```shell
kubectl apply -f https://kuboard.cn/install-script/kuboard-beta.yaml
```

查看端口号

```shell
kubectl get svc -n kube-system
```

获取token

```shell
echo $(kubectl -n kube-system get secret $(kubectl -n kube-system get secret | grep ^kuboard-user | awk '{print $1}') -o go-template='{{.data.token}}' | base64 -d)
```

## 更改一些配置

把kube-proxy改成ipvs模式

```shell
kubectl edit cm kube-proxy -n kube-system
mode:"ipvs"
```

更新kube-proxy的pod

```shell
kubectl patch daemonset kube-proxy -p "{\"spec\":{\"template\":{\"metadata\":{\"annotations\":{\"date\":\"`date +'%s'`\"}}}}}" -n kube-system
```

验证kube-proxy模式

```shell
curl 127.0.0.1:10249/proxyMode
ipvs
```

## 注意事项

kubeadm安装的集群，证书有效期默认是一年。master 节点的 kubenapiserver、kube-scheduler、kube-controller-manager-etcd 都是以容器运行的。可以通过 kubectl get po -n kube-system 查看。

启动和二进制不同的是，kubelet 的配置文件在/etc/sysconfig/kubelet 和/var/lib/kubelet/config.yaml
其他组件的配置文件在/etc/Kubernetes/manifests 目录下，比如 kube-apiserver.yaml，该 yaml 文件更改后，kubelet 会自动刷新配置，也就是会重启 pod。不能再次创建该文件

## 集群验证

查看所有namespace的容器

kubectl get po --all-namespaces [-owide]

查看监控数据

kubectl top po --all-namespaces

查看svc

kubectl get svc

kubectl get svc -n kube-system

## k8s证书

建K8S集群 kubeadm 会生成的很多证书

```shell
[root@k8s-master01 ~]# cd /etc/kubernetes/pki/
[root@k8s-master01 pki]# tree 
.
├── apiserver.crt
├── apiserver-etcd-client.crt
├── apiserver-etcd-client.key
├── apiserver.key
├── apiserver-kubelet-client.crt
├── apiserver-kubelet-client.key
├── ca.crt
├── ca.key
├── etcd
│   ├── ca.crt
│   ├── ca.key
│   ├── healthcheck-client.crt
│   ├── healthcheck-client.key
│   ├── peer.crt
│   ├── peer.key
│   ├── server.crt
│   └── server.key
├── front-proxy-ca.crt
├── front-proxy-ca.key
├── front-proxy-client.crt
├── front-proxy-client.key
├── sa.key
└── sa.pub
```

k8s集群一共有多少证书：

```shell
先从Etcd算起：
1、Etcd对外提供服务，要有一套etcd server证书
2、Etcd各节点之间进行通信，要有一套etcd peer证书
3、Kube-APIserver访问Etcd，要有一套etcd client证书

再算kubernetes：
4、Kube-APIserver对外提供服务，要有一套kube-apiserver server证书
5、kube-scheduler、kube-controller-manager、kube-proxy、kubelet和其他可能用到的组件，
   需要访问kube-APIserver，要有一套kube-APIserver client证书
6、kube-controller-manager要生成服务的service account，要有一对用来签署service account的证书(CA证书)
7、kubelet对外提供服务，要有一套kubelet server证书
8、kube-APIserver需要访问kubelet，要有一套kubelet client证书
```

公钥私钥：

```shell
公钥和私钥是成对的，它们互相解密。
公钥加密，私钥解密。
```

数字签名：

![img](https://gitee.com/c_honghui/picture/raw/master/img/20220305174925.jpeg)

1.通过接收方公钥来进行加密得到密文。

为什么要接收方的公钥来加密? 因为只有接收方的私钥可以解开接受方公钥加过的密，保证只有接受方可以解密。

2.对hello kitty哈希得到摘要，接着再经过发送方私匙进行签名，签名后得出的数字签名和密文一起发给接受方。

为什么要用发送方的私钥签名？因为这种方式，才能让接收方确认这条信息是发送方发出来的。只有发送方的公钥才能解开发送方的签名。

接收方同样对接收到的信息（密文及数字签名）进行以下步骤处理：

1.用自己的私匙解开密文，得到hello kitty.

2.对hello kitty哈希得到摘要。

3.通过发送方的公钥解开发送方签名，得到摘要‚。

4.对解密密文的摘要和解密数字签名的摘要‚进行对比，若摘要一致，则可确认信息为发送方所发，及信息的完整性。

根证书和证书：

```shell
ca.pem是CA证书,ca-key.pem是ca的私钥
```



