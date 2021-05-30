# K8S集群部署

k8s基础集群环境主要是运行kubernetes管理端服务以及node节点上的服务部署及使用。

## k8s高可用集群环境规划信息

安装实际需求，进行规划与部署相应的单master或者多master的高可用k8s运行环境。

### 单master

见 kubeadm 安装k8s

### 多master

![image-20210531000812384](https://gitee.com/c_honghui/picture/raw/master/img/20210531000812.png)

## 服务器信息

| 类型            | 服务器IP地址          | 备注                                     |
| --------------- | --------------------- | ---------------------------------------- |
| Ansible(2台)    | 192.168.7.101/102     | K8S集群部署服务器                        |
| K8S Master(2台) | 192.168.7.101/102     | K8s控制端，通过一个VIP做主备高可用       |
| Harbor(2台)     | 192.168.7.103/104     | 高可用镜像服务器                         |
| Etcd(最少3台）  | 192.168.7.105/106/107 | 保存k8s集群数据的服务器                  |
| Hproxy(2台)     | 192.168.7.108/109     | 高可用etcd代理服务器                     |
| Node节点(2-N台) | 192.168.7.111/112/xxx | 真正运行容器的服务器，高可用环境至少两台 |

## 软件清单

API端口：

```text
端口：192.168.7.248：6443 #需要配置在负载均衡上实现反向代理，dashboard的端口为8443
操作系统：ubuntu server 1804
k8s版本： 1.13.5
calico：3.4.4
```

## 基础环境准备

keepalived:

```shell
root@k8s-ha1:~# cat /etc/keepalived/keepalived.conf
vrrp_instance VI_1 {
state MASTER
interface eth0
virtual_router_id 1
priority 100
advert_int 3
unicast_src_ip 192.168.7.108
unicast_peer {
192.168.7.109
}
authentication {
auth_type PASS
auth_pass 123abc
}
virtual_ipaddress {
192.168.7.248 dev eth0 label eth0:1
}
}
```

haproxy:

```shell
listen k8s_api_nodes_6443
bind 192.168.7.248:6443
mode tcp
#balance leastconn
server 192.168.7.101 192.168.7.101:6443 check inter 2000 fall 3 rise 5
#server 192.168.100.202 192.168.100.202:6443 check inter 2000 fall 3 rise 5
```

harbor:

```shell
root@k8s-harbor1:/usr/local/src/harbor# pwd
/usr/local/src/harbor
root@k8s-harbor1:/usr/local/src/harbor# mkdir certs/

# openssl genrsa -out /usr/local/src/harbor/certs/harbor-ca.key #生成私有key
# openssl req -x509 -new -nodes -key /usr/local/src/harbor/certs/harbor-ca.key -subj
"/CN=harbor.magedu.net" -days 7120 -out /usr/local/src/harbor/certs/harbor-ca.crt #签证

# vim harbor.cfg
hostname = harbor.magedu.net
ui_url_protocol = https
ssl_cert = /usr/local/src/harbor/certs/harbor-ca.crt
ssl_cert_key = /usr/local/src/harbor/certs/harbor-ca.key
harbor_admin_password = 123456

# ./install.sh

client 同步在crt证书：
master1:~# mkdir /etc/docker/certs.d/harbor.magedu.net -p
harbor1:~# scp /usr/local/src/harbor/certs/harbor-ca.crt
192.168.7.101:/etc/docker/certs.d/harbor.magedu.net
master1:~# vim /etc/hosts #添加host文件解析
192.168.7.103 harbor.magedu.net
master1:~# systemctl restart docker #重启docker

测试登录harbor：
root@k8s-master1:~# docker login harbor.magedu.net
Username: admin
Password:
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store
Login Succeeded

测试push镜像到harbor：
master1:~# docker pull alpine
root@k8s-master1:~# docker tag alpine harbor.magedu.net/library/alpine:linux36
root@k8s-master1:~# docker push harbor.magedu.net/library/alpine:linux36
The push refers to repository [harbor.magedu.net/library/alpine]
256a7af3acb1: Pushed
linux36: digest:
sha256:97a042bf09f1bf78c8cf3dcebef94614f2b95fa2f988a5c07314031bc2570c7a size: 528
```

## 手动二进制部署



## ansible部署

基础环境准备：

```shell
# apt-get install python2.7
# ln -s /usr/bin/python2.7 /usr/bin/python
# apt-get install git ansible -y
# ssh-keygen #生成密钥对
# apt-get install sshpass #ssh同步公钥到各k8s服务器
#分发公钥脚本：
root@k8s-master1:~# cat scp.sh
#!/bin/bash
#目标主机列表
IP="
192.168.7.101
192.168.7.102
192.168.7.103
192.168.7.104
192.168.7.105
192.168.7.106
192.168.7.107
192.168.7.108
192.168.7.109
192.168.7.110
192.168.7.111
"
for node in ${IP};do
sshpass -p 123456 ssh-copy-id ${node} -o StrictHostKeyChecking=no
if [ $? -eq 0 ];then
echo "${node} 秘钥copy完成"
else
echo "${node} 秘钥copy失败"
fi
done
#同步docker证书脚本：
#!/bin/bash
#目标主机列表
IP="
192.168.7.101
192.168.7.102
192.168.7.103
192.168.7.104
192.168.7.105
192.168.7.106
192.168.7.107
192.168.7.108
192.168.7.109
192.168.7.110
192.168.7.111
"
for node in ${IP};do
sshpass -p 123456 ssh-copy-id ${node} -o StrictHostKeyChecking=no
if [ $? -eq 0 ];then
echo "${node} 秘钥copy完成
echo "${node} 秘钥copy完成,准备环境初始化....."
ssh ${node} "mkdir /etc/docker/certs.d/harbor.magedu.net -p"
echo "Harbor 证书目录创建成功!"
scp /etc/docker/certs.d/harbor.magedu.net/harbor-ca.crt
${node}:/etc/docker/certs.d/harbor.magedu.net/harbor-ca.crt
echo "Harbor 证书拷贝成功!"
scp /etc/hosts ${node}:/etc/hosts
echo "host 文件拷贝完成"
scp -r /root/.docker ${node}:/root/
echo "Harbor 认证文件拷贝完成!"
scp -r /etc/resolv.conf ${node}:/etc/
else
echo "${node} 秘钥copy失败"
fi
done

#执行脚本同步：
k8s-master1:~# bash scp.sh

root@s2:~# vim ~/.vimrc #取消vim 自动缩进功能
set paste
```

clone项目：

```shell
# git clone -b 0.6.1 https://github.com/easzlab/kubeasz.git
root@k8s-master1:~# mv /etc/ansible/* /opt/
root@k8s-master1:~# mv kubeasz/* /etc/ansible/
root@k8s-master1:~# cd /etc/ansible/
root@k8s-master1:/etc/ansible# cp example/hosts.m-masters.example ./hosts #复制hosts模板文件
```

准备hosts文件：

```shell
root@k8s-master1:/etc/ansible# pwd
/etc/ansible
root@k8s-master1:/etc/ansible# cp example/hosts.m-masters.example ./hosts
root@k8s-master1:/etc/ansible# cat hosts
# 集群部署节点：一般为运行ansible 脚本的节点
# 变量 NTP_ENABLED (=yes/no) 设置集群是否安装 chrony 时间同步
[deploy]
192.168.7.101 NTP_ENABLED=no
# etcd集群请提供如下NODE_NAME，注意etcd集群必须是1,3,5,7...奇数个节点
[etcd]
192.168.7.105 NODE_NAME=etcd1
192.168.7.106 NODE_NAME=etcd2
192.168.7.107 NODE_NAME=etcd3
[new-etcd] # 预留组，后续添加etcd节点使用
#192.168.7.x NODE_NAME=etcdx
[kube-master]
192.168.7.101
[new-master] # 预留组，后续添加master节点使用
#192.168.7.5
[kube-node]
192.168.7.110
[new-node] # 预留组，后续添加node节点使用
#192.168.7.xx
# 参数 NEW_INSTALL：yes表示新建，no表示使用已有harbor服务器
# 如果不使用域名，可以设置 HARBOR_DOMAIN=""
[harbor]
#192.168.7.8 HARBOR_DOMAIN="harbor.yourdomain.com" NEW_INSTALL=no
# 负载均衡(目前已支持多于2节点，一般2节点就够了) 安装 haproxy+keepalived
[lb]
192.168.7.1 LB_ROLE=backup
192.168.7.2 LB_ROLE=master
#【可选】外部负载均衡，用于自有环境负载转发 NodePort 暴露的服务等
[ex-lb]
#192.168.7.6 LB_ROLE=backup EX_VIP=192.168.7.250
#192.168.7.7 LB_ROLE=master EX_VIP=192.168.7.250
[all:vars]
# ---------集群主要参数---------------
#集群部署模式：allinone, single-master, multi-master
DEPLOY_MODE=multi-master
#集群主版本号，目前支持: v1.8, v1.9, v1.10，v1.11, v1.12, v1.13
K8S_VER="v1.13"
# 集群 MASTER IP即 LB节点VIP地址，为区别与默认apiserver端口，设置VIP监听的服务端口8443
# 公有云上请使用云负载均衡内网地址和监听端口
MASTER_IP="192.168.7.248"
KUBE_APISERVER="https://{{ MASTER_IP }}:6443"
# 集群网络插件，目前支持calico, flannel, kube-router, cilium
CLUSTER_NETWORK="calico"
# 服务网段 (Service CIDR），注意不要与内网已有网段冲突
SERVICE_CIDR="10.20.0.0/16"
# POD 网段 (Cluster CIDR），注意不要与内网已有网段冲突
CLUSTER_CIDR="172.31.0.0/16"
# 服务端口范围 (NodePort Range)
NODE_PORT_RANGE="20000-60000"
# kubernetes 服务 IP (预分配，一般是 SERVICE_CIDR 中第一个IP)
CLUSTER_KUBERNETES_SVC_IP="10.20.0.1"
# 集群 DNS 服务 IP (从 SERVICE_CIDR 中预分配)
CLUSTER_DNS_SVC_IP="10.20.254.254"
# 集群 DNS 域名
CLUSTER_DNS_DOMAIN="linux36.local."
# 集群basic auth 使用的用户名和密码
BASIC_AUTH_USER="admin"
BASIC_AUTH_PASS="123456"
# ---------附加参数--------------------
#默认二进制文件目录
bin_dir="/usr/bin"
#证书目录
ca_dir="/etc/kubernetes/ssl"
#部署目录，即 ansible 工作目录，建议不要修改
base_dir="/etc/ansible"
```

准备二进制文件：

```shell
k8s-master1:/etc/ansible/bin# pwd
/etc/ansible/bin
k8s-master1:/etc/ansible/bin# tar xvf k8s.1-13-5.tar.gz
k8s-master1:/etc/ansible/bin# mv bin/* .
```

部署：

环境初始化

```shell
root@k8s-master1:/etc/ansible# pwd
/etc/ansible
root@k8s-master1:/etc/ansible# ansible-playbook 01.prepare.yml
```

部署etcd集群

```shell
root@k8s-master1:/etc/ansible# ansible-playbook 02.etcd.yml
```

各etcd服务器验证etcd服务

```shell
root@k8s-etcd1:~# export NODE_IPS="192.168.7.105 192.168.7.106 192.168.7.107"
root@k8s-etcd1:~# for ip in ${NODE_IPS}; do ETCDCTL_API=3 /usr/bin/etcdctl --
endpoints=https://${ip}:2379 --cacert=/etc/kubernetes/ssl/ca.pem --
cert=/etc/etcd/ssl/etcd.pem --key=/etc/etcd/ssl/etcd-key.pem endpoint health; done
https://192.168.7.105:2379 is healthy: successfully committed proposal: took =
2.198515ms
https://192.168.7.106:2379 is healthy: successfully committed proposal: took =
2.457971ms
https://192.168.7.107:2379 is healthy: successfully committed proposal: took =
1.859514ms
```

部署docker

```shell
root@k8s-master1:/etc/ansible# ansible-playbook 03.docker.yml
```

部署master

```shell
root@k8s-master1:/etc/ansible# ansible-playbook 04.kube-master.yml
```

部署node

```shell
node节点必须安装docker

root@k8s-master1:/etc/ansible# vim roles/kube-node/defaults/main.yml
# 基础容器镜像
SANDBOX_IMAGE: "harbor.magedu.net/baseimages/pause-amd64:3.1"
root@k8s-master1:/etc/ansible# ansible-playbook 05.kube-node.yml
```

部署网络服务calico

```shell
# docker load -i calico-cni.tar
# docker tag calico/cni:v3.4.4 harbor.magedu.net/baseimages/cni:v3.4.4
# docker push harbor.magedu.net/baseimages/cni:v3.4.4
# docker load -i calico-node.tar
# docker tag calico/node:v3.4.4 harbor.magedu.net/baseimages/node:v3.4.4
# docker push harbor.magedu.net/baseimages/node:v3.4.4
# docker load -i calico-kube-controllers.tar
# docker tag calico/kube-controllers:v3.4.4 harbor.magedu.net/baseimages/kubecontrollers:v3.4.4
# docker push harbor.magedu.net/baseimages/kube-controllers:v3.4.4
执行部署网络：
root@k8s-master1:/etc/ansible# ansible-playbook 06.network.yml
```

验证calico

```shell
root@k8s-master1:/etc/ansible# calicoctl node status
Calico process is running.
IPv4 BGP status
+---------------+-------------------+-------+----------+-------------+
| PEER ADDRESS | PEER TYPE | STATE | SINCE | INFO |
+---------------+-------------------+-------+----------+-------------+
| 192.168.7.110 | node-to-node mesh | up | 14:22:44 | Established |
+---------------+-------------------+-------+----------+-------------+
IPv6 BGP status
No IPv6 peers found.


kubectl run net-test1 --image=alpine --replicas=4 sleep 360000 #创建pod测试夸主机网络通信是否正常
```

添加node节点

```shell
[kube-node]
192.168.7.110
[new-node] # 预留组，后续添加node节点使用
192.168.7.111
root@k8s-master1:/etc/ansible# ansible-playbook 20.addnode.yml
```

添加master节点

```shell
注释掉lb，否则无法下一步

[kube-master]
192.168.7.101
[new-master] # 预留组，后续添加master节点使用
192.168.7.102
root@k8s-master1:/etc/ansible# ansible-playbook 21.addmaster.yml
```

验证当前状态

```shell
root@k8s-master1:/etc/ansible# calicoctl node status
Calico process is running.
IPv4 BGP status
+---------------+-------------------+-------+----------+-------------+
| PEER ADDRESS | PEER TYPE | STATE | SINCE | INFO |
+---------------+-------------------+-------+----------+-------------+
| 192.168.7.110 | node-to-node mesh | up | 14:22:45 | Established |
| 192.168.7.111 | node-to-node mesh | up | 14:33:24 | Established |
| 192.168.7.102 | node-to-node mesh | up | 14:42:21 | Established |
+---------------+-------------------+-------+----------+-------------+
IPv6 BGP status
No IPv6 peers found.
root@k8s-master1:/etc/ansible# kubectl get nodes
NAME STATUS ROLES AGE VERSION
192.168.7.101 Ready,SchedulingDisabled master 37m v1.13.5
192.168.7.102 Ready,SchedulingDisabled master 41s v1.13.5
192.168.7.110 Ready node 33m v1.13.5
192.168.7.111 Ready node 15m v1.13.5
```

