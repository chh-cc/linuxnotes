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
| K8s版本     | 1.22.x         |
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
for i in k8s-master01 k8s-master02 k8s-master03 k8s-node01 k8s-node02;do ssh-copy-id -i .ssh/id_rsa.pub $i;done
```

下载安装所有的源码文件：

```shell
cd /root/ ; git clone https://github.com/dotbalo/k8s-ha-install.git
```

所有节点升级系统并重启：

```shell
yum update -y --exclude=kernel* && reboot
```

## 内核配置

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
[root@k8s-master01 keepalived]# cat /etc/keepalived/check_apiserver.sh 
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

Master01节点创建new.yaml配置文件如下：

```yaml
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
kubernetesVersion: v1.22.3
networking:
  dnsDomain: cluster.local
  podSubnet: 172.168.0.0/16
  serviceSubnet: 10.96.0.0/12
scheduler: {}
```

将new.yaml文件复制到其他master节点，之后所有Master节点提前下载镜像，可以节省初始化时间：

```shell
kubeadm config images pull --config /root/new.yaml 
```

所有节点设置开机自启动kubelet

```shell
systemctl enable --now kubelet（如果启动失败无需管理，初始化成功以后即可启动）
```

Master01节点初始化，初始化以后会在/etc/kubernetes目录下生成对应的证书和配置文件，之后其他Master节点加入Master01即可：

```shell
kubeadm init --config /root/new.yaml  --upload-certs
```

