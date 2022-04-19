# Linux

内核是操作系统最重要的部分——它执行重要的功能，如进程管理、内存管理、文件系统管理等。

Linux 是一个内核，而不是一个完整的操作系统。Linux内核与GNU系统结合构成一个完整的操作系统。因此，基于 linux 的操作系统也被称为 GNU/Linux 系统。GNU 是一个广泛的自由软件集合，如编译器、调试器、C 库等。

Linux 内核由 Linus Torvalds 独立开发和发布。Linux 内核是免费和开源的 - https://github.com/torvalds/linux

## 流行的 Linux 发行版列表

- Fedora
- Ubuntu
- Debian
- Centos
- Red Hat Enterprise Linux
- Suse
- Arch Linux

| Packaging systems    | Distributions                            | Package manager |
| :------------------- | :--------------------------------------- | :-------------- |
| Debian style (.deb)  | Debian, Ubuntu                           | APT             |
| Red Hat style (.rpm) | Fedora, CentOS, Red Hat Enterprise Linux | YUM             |



Shell 是一个程序，它从用户那里获取命令并将它们交给操作系统进行处理。Shell 是 CLI（命令行界面）的一个示例。Bash 是 Linux 服务器上最流行的 shell 程序之一。其他流行的 shell 程序是 zsh、ksh 和 tcsh。

终端是一个程序，可以打开一个窗口并让您与 shell 进行交互。一些流行的终端示例是 gnome-terminal、xterm、konsole 等。

## 开机流程

centos7开机流程：

1. 开机启动，BIOS自检，检查CPU、硬盘等硬件信息
2. MBR引导，主分区引导【读取0磁道0柱面1扇区的前446字节】
3. GRUB2引导，确定加载哪个系统【GRUB是GUN项目的多操作系统启动程序】
4. 由GRUB2加载内核（/boot目录下面的kernel）【cat /proc/version或uname -a即可查看内核版本信息】
5. 内核启动系统第一个进程systemd【Linux启动的第一个进程】
6. systemd开始调用默认单元组，并按照默认单元组开始运行子单元组，如初始化系统、启动字符界面直到登录界面启动

## CPU

CPU主要和内存进行交互，从内存提取指令并执行它。由于访问内存获取执行或数据要比执行指令花费的时间长，因此所有的 CPU 内部都会包含一些`寄存器`来保存关键变量和临时结果。

## 内存

存储器系统采用了分层次结构

<img src="https://gitee.com/c_honghui/picture/raw/master/img/20220228112349.png" alt="os1-8.png" style="zoom:67%;" />



只有内核才可以直接访问**物理内存**。

Linux 内核给每个进程都提供了一个独立的**虚拟地址空间**，并且这个地址空间是连续的。这样，进程就可以很方便地访问内存，更确切地说是访问虚拟内存。

进程在用户态时，只能访问用户空间内存；只有进入内核态后，才可以访问内核空间内存。内核空间，其实关联的都是相同的物理内存。



## 网络

**linux处理数据包过程：**

![img](https://gitee.com/c_honghui/picture/raw/master/img/20220227210652.png)

如果Linux主机有多块网卡，如果不开启数据包转发功能，则这些网卡之间是无法互通的。例如eth0是172.16.10.0/24网段，而eth1是192.168.100.0/24网段，到达该Linux主机的数据包无法从eth0交给eth1或者从eth1交给eth0，除非Linux主机开启了数据包转发功能。

在Linux上开启转发功能有多种方法：

```
shell> echo 1 > /proc/sys/net/ipv4/ip_forward
shell> sysctl -w net.ipv4.ip_forward=1
```

查看是否开启了转发功能

```shell
sysctl net.ipv4.ip_forward
net.ipv4.ip_forward = 0
```

**网卡配置文件ifcfg-***

```shell
DEVICE="eth0"      # 显示的名称，必须/sys/class/net/目录下的某个网卡名相同
IPV6INIT="no" 
BOOTPROTO="dhcp"
ONBOOT=yes 
TYPE="Ethernet"
DEFROUTE="yes"
PEERDNS="yes"      # 设置为yes时，此文件设置的DNS将覆盖/etc/resolv.conf，
                   # 若开启了DHCP，则默认为yes，所以dhcp的dns也会覆盖/etc/resolv.conf
PEERROUTES="yes"
IPV4_FAILURE_FATAL="no"
NAME="System eth0"
DNS1=114.114.114.114
DNS2=8.8.8.8
DNS3=114.114.115.115
```

ifconfig所有的配置都是应用于内核的，所以只会临时生效，重启网络服务后会立即失效

**DNS配置文件/etc/resolv.conf**

```shell
domain  domain_name       # 声明本地域名，即解析时自动隐式补齐的域名
search  domain_name_list  # 指定域名搜索顺序(最多6个)，和domain不能共存，若共存了，则后面的行生效
nameserver  IP1           # 设置DNS指向，最多3个
nameserver  IP2
nameserver  IP3        
options timeout:n attempts:n  # 指定解析超时时间(默认5秒)和解析次数(默认2次)
```

**端口和服务的对应关系/etc/services**

Linux上分为3种路由：
主机路由：直接指明到某台具体的主机怎么走，主机路由也就是所谓的静态路由
网络路由：指明某类网络怎么走
默认路由：不走主机路由的和网络路由的就走默认路由。操作系统上设置的默认路由一般也称为网关。

例如下面的路由表中，若ping 192.168.5.20，则先比对192.168.100.78发现无法匹配，然后比对192.168.100.0，发现也无法匹配，接着再匹配192.168.0.0这条网络路由条目，发现能匹配，所以选择该路由条目。

```shell
# route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         192.168.100.2   0.0.0.0         UG    100    0        0 eth0
172.16.10.0     0.0.0.0         255.255.255.0   U     100    0        0 eth1
192.168.0.0     192.168.100.70  255.255.0.0     UG    0      0        0 eth0
192.168.100.0   0.0.0.0         255.255.255.0   U     100    0        0 eth0
192.168.100.78  0.0.0.0         255.255.255.255 UH    0      0        0 eth0

#U (route is up)
#H (target is a host)
#G (use gateway，也即是设置了下一跳的路由条目)
```

## 进程与线程

### 进程

严格说，在某一瞬间，CPU只能运行一个进程，如果把时间定位为1秒内的话，它可能运行多个进程，就会产生`并行`的错觉。

创建进程的方式:

**系统初始化**

启动操作系统时，通常会创建若干个进程。其中有些是**前台进程(numerous processes)**，也就是同用户进行交互并替他们完成工作的进程。一些运行在后台，并不与特定的用户进行交互，例如，设计一个进程来接收发来的电子邮件，这个进程大部分的时间都在休眠，但是只要邮件到来后这个进程就会被唤醒。进程运行在后台用来处理一些活动像是 e-mail，web 网页，新闻，打印等等被称为 **守护进程(daemons)**。

**系统调用创建**

通常，一个正在运行的进程会发出`系统调用`用来创建一个或多个新进程来帮助其完成工作。例如，如果有大量的数据需要经过网络调取并进行顺序处理，那么创建一个进程读数据，并把数据放到共享缓冲区中，而让第二个进程取走并正确处理会比较容易些。在多处理器中，让每个进程运行在不同的 CPU 上也可以使工作做的更快。

**用户请求创建**

在许多交互式系统中，输入一个命令或者双击图标就可以启动程序

### 线程

## 权限

### 文件基本权限

文件基本权限是最常用也是最有效的linux安全防护手段

遵循最小化权限原则

#### 文件身份

身份类别：所属主u，所属组g，其他o

身份修改：`chown [-R] 所属主:所属组 filename`  `chgrp [-R] 所属组 filename`

#### 文件权限的含义

r：查看文件内容

w：编辑文件内容，但不能删除文件本身

x：文件的执行权限

#### 目录权限的含义

r：查看目录的文件列表

w：目录内文件的增删、复制、剪切

x：进入目录

#### 权限修改

`chmod  [-R] [augo] [+-=] [rwx] filename`

`chmod [-R] 644 filename`

### 特殊权限

SUID：作用于二进制文件，默认情况下用户发起一个进程，该进程的属主是用户，如果给二进制程序文件添加SUID权限，则该进程的属主是程序文件的属主

SGID：作用于目录上时，此文件夹下所有用户新建文件都自动继承此目录的用户组

SBIT：当某个目录拥有其Sticky权限 ，则其目录下的文件和目录只有root和其拥有者删除，其他用户不能删除

### sudo权限

配置文件：/etc/sudoers

命令：visudo，尽量用这个命令，有纠错功能

格式：被授权用户 可执行主机= (以哪个身份运行) 标签:可执行命令绝对路径

执行：sudo 授权命令；五分钟内再次执行不需要输入用户密码