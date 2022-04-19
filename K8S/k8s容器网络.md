## k8s容器网络

## 容器跨主机网络

理解容器“跨主通信”的原理，就要先从 Flannel 这个项目说起。

Flannel 项目本身只是一个框架，真正为我们提供容器网络功能的，是 Flannel 的后端实现，目前，Flannel 支持三种后端实现，分别是：

1. VXLAN（虚拟可扩展局域网）；
2. host-gw；
3. UDP。（淘汰）



VXLAN 的覆盖网络的设计思想是：在现有的三层网络之上，“覆盖”一层虚拟的、由内核 VXLAN 模块负责维护的二层网络，使得连接在这个 VXLAN 二层网络上的“主机”（虚拟机或者容器都可以）之间，可以像在同一个局域网（LAN）里那样自由通信。

而为了能够在二层网络上打通“隧道”，VXLAN 会在宿主机上设置一个特殊的网络设备作为“隧道”的两端，**这个设备就叫作 VTEP**，即：VXLAN Tunnel End Point（虚拟隧道端点）。docker0 与这个设备通过 IP 转发（路由表）进行协作。

基于 VTEP 设备进行“隧道”通信的流程：

<img src="https://gitee.com/c_honghui/picture/raw/master/img/20220325170720.png" alt="img" style="zoom: 50%;" />

flannel.1 的设备，就是 VXLAN 所需的 VTEP 设备，它既有 IP 地址，也有 MAC 地址。

container-1访问container-2，目的地址是 10.1.16.3 的 IP 包，会先出现在 docker0 网桥，然后被路由到本机 flannel.1 设备，也就是”隧道“的入口

当 Node 2 启动并加入 Flannel 网络之后，在 Node 1（以及所有其他节点）上，flanneld 就会添加一条如下所示的路由规则：

```shell
$ route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
...
10.1.16.0       10.1.16.0       255.255.255.0   UG    0      0        0 flannel.1

#这条规则的意思是：凡是发往 10.1.16.0/24 网段的 IP 包，都需要经过 flannel.1 设备发出，并且，它最后被发往的网关地址是：10.1.16.0(Node 2 上的 VTEP 设备)。
```

VTEP设备之间，要想办法建立一个虚拟的二层网络，通过二层数据帧通信。

“源 VTEP 设备”收到“原始 IP 包”后，就要想办法把“原始 IP 包”加上一个目的 MAC 地址，封装成一个二层数据帧，然后发送给“目的 VTEP 设备”（当然，这么做还是因为这个 IP 包的目的地址不是本机）。

接下来，Linux 内核还需要再把“内部数据帧”进一步封装成为宿主机网络里的一个普通的数据帧，好让它“载着”“内部数据帧”，通过宿主机的 eth0 网卡进行传输。



Node 1 上的 flannel.1 设备就可以把这个数据帧从 Node 1 的 eth0 网卡发出去。显然，这个帧会经过宿主机网络来到 Node 2 的 eth0 网卡。

这时候，Node 2 的内核网络栈会发现这个数据帧里有 VXLAN Header，并且 VNI=1。所以 Linux 内核会对它进行拆包，拿到里面的内部数据帧，然后根据 VNI 的值，把它交给 Node 2 上的 flannel.1 设备。

而 flannel.1 设备则会进一步拆包，取出“原始 IP 包”。最终，IP 包就进入到了 container-2 容器的 Network Namespace 里。

## Kubernetes网络模型与CNI网络插件

网络插件，通过某种方法，把不同宿主机上的特殊设备连通，从而达到容器跨主机通信的目的。

Kubernetes 是通过一个叫作 CNI 的接口，维护了一个单独的网桥来代替 docker0。这个网桥的名字就叫作：**CNI 网桥**。它在宿主机上的设备名字默认是：**cni0**。



k8s网络模型：

1. 所有容器都可以直接使用 IP 地址与其他容器通信，而无需使用 NAT。
2. 所有宿主机都可以直接使用 IP 地址与所有容器通信，而无需使用 NAT，反之亦然。
3. 容器自己『看到』的自己的 IP 地址，和别人（宿主机或者容器）看到的地址是完全一样的。