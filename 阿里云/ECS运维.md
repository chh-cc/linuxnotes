# ECS运维

## Linux 启动与登录问题

Linux 启动与登录问题是 ECS 的高频问题，而往往处理不及时会直接影响到用户业务的正常可持续运行，因此也变成了我们处理问题优先级的重中之重。
在云环境上影响 ECS 启动与登录的因素非常多，镜像、管控、虚拟化、底层硬件、系统与文件异常等等。

### 系统启动与登陆异常

#### 系统启动异常

1.	 部分 CentOS 系统启动黑屏，无异常报错的场景，可以 fsck 一下系统盘。
2.	 根分区空间满，以及 inode 数量耗尽。
3.	 升级内核或者从老的共享实例迁移到独享规格导致的启动异常。
3.1	 手动注入驱动 (mkinitrd virtio 相关驱动 )。
3.2	 修改 grub 的启动顺序，优先尝试使用老内核启动。
3.3	 /boot 目录下面内核的关联文件是否全（下面仅为 demo，不同系统内核版
本文件不一致，部分内核版本 boot 下的 i386 目录也是有用的）。  

```text
config-4.9.0-7-amd64
initrd.img-4.9.0-7-amd64
System.map-4.9.0-7-amd64
vmlinuz-4.9.0-7-amd64
```

​	    3.4　/boot/grub/device.map 里面的 hda 改成 vda。  

4. fstab/grub 中的 uuid 不对，可以直接修改为 /dev/vda1 这种形式尝试。数据盘分区异常加载起不来的场景，可以去注释 fstab 所有的行，添加类似下面的启动项尝试，也适用于系统盘快照创建云盘挂载后，uuid 一致导致的启动异常，改成非 UUID 的挂载即可。  

```shell
/dev/vda1 / ext4 defaults 1 1
```

5.	 根目录权限 777（部分目录 777）也会导致启动异常，或者 ssh 登陆异常。
	 可参考下面的文章仅限修复尝试。
https://yq.aliyun.com/articles/761371
	
6. 常见的关键目录缺失，有的是软链，也可以看看对应目录下面的文件数量（文件
   数量要跟同内核版本或者相差不大的版本对比），简单判断。

   ```shell
   /bin /sbin /lib /lib32 /lib64 /etc /boot /usr/bin /usr/sbin /usr/lib /
   usr/lib64 等目录或文件缺失
   for i in /bin /sbin /lib /lib32 /lib64 /etc /boot /usr/bin /usr/sbin /
   usr/lib /usr/lib64 ;do ls -l $i |wc -l ;done
   ```

7. 影响启动的参数。
   如果参数设置不当，是会导致启动异常的，如 /etc/sysctl.conf 以及检查 rc.local
   的配置，profile 的检查。

   ```text
   vm.nr_hugepages
   vm.min_free_kbytes 
   ```

8. CentOS 的 selinux 需要关闭。  

#### root登陆异常

1. /etc/passwd /etc/shadow ( 用户名 root polikt dbus 等关键用户存在与否，文
   件为空，格式乱（dos2unix)。

2. /etc/pam.d 目录下是否有为空的文件及参数设置是否正常，如常见的 system-auth passwd。

3. /etc/pam.d 下面所有文件里面涉及的 so 文件，看看文件是否存在，是否为空 /
   usr/lib64/security。

4. 查 /etc /lib64 /bin /sbin /usr/bin /usr/sbin 等目录有没有 size 为 0 的文件。

5. /etc/profile /etc/profile.d( 打 印 列 表 ) /etc/bashrc /root/.bash_profile /root/.
   bashrc 等涉及登陆环境设置的文件是否异常。

6. 注意内核版本，是否存在新老内核，多更换几个内核试下。

7. 系统日志也是一个比较重要的检查项（后面会介绍无法登陆怎么检查）。

8. Ubuntu 12.04 登陆异常 在 /etc/login.defs 里面配置了错误的 ERASECHAR
   导致，恢复默认 0177 即可。  

   ```text
   configuration error - cannot parse erasechar value  
   ```

9. 输入 root 后直接 login 失败三连，日志如下。  

   ![image-20210303103154421](https://gitee.com/c_honghui/picture/raw/master/img/20210303103200.png)

​     找个同内核版本的机器对比发现没有 /etc/pam.d/login。
​	 rpm 包校验一下，确认 login 文件没了，手动创建一个，内容拷贝过来，好了。  

```shell
[root@iZbp1cabe6lyx26ikmjie2Z pam.d]# rpm -V util-linux
missing c /etc/pam.d/login
[root@iZbp1cabe6lyx26ikmjie2Z pam.d]# rpm -ql util-linux|egrep -vi "gz|mo|share"
/etc/mtab
/etc/pam.d/chfn
/etc/pam.d/chsh
/etc/pam.d/login
/etc/pam.d/runuser
/etc/pam.d/runuser-l
/etc/pam.d/su
/etc/pam.d/su-l
```

10. /etc/ssh/sshd_config 相 关 参 数 如 LoginGraceTime/Allowusers/PermitRootLogin。  

11.	 问题不好确认的时候，可以将 shadow 密码字段清空，看看登陆是否正常，可以判断是否到密码验证阶段了。  

​       之前有过一篇关于 ssh 问题排查的文档，可参考：https://yq.aliyun.com/articles/540769  

系统登陆不进去了，不挂盘的情况下怎么操作？
上面的检查点很多是需要切换到另外的系统环境下去做检查，比如挂载 LiveCD 或者chroot 切换；但对于使用 ECS 的用户来说，阿里云暂还未提供实例挂载 ISO 镜像的功能，那么如何进行上面的操作呢？可以借助阿里云新推出的卸载系统盘功能，可以把系统盘卸载掉，作为数据盘挂载到一个新的机器，这样就可以执行上面的检查了。
详见： https://help.aliyun.com/document_detail/146752.html    

Linux 系统常见问题诊断覆盖以下场景：  

![image-20210303104549455](https://gitee.com/c_honghui/picture/raw/master/img/20210303104549.png)

Linux 系统常见启动问题修复覆盖以下场景：  

![image-20210303104615919](https://gitee.com/c_honghui/picture/raw/master/img/20210303104616.png)

OS 参数收集脚本： https://public-beijing.oss-cn-beijing.aliyuncs.com/oos/caiji.sh
OS 优化脚本 github 链接： https://github.com/huigher/os-performance-optimization
OS 优化脚本 OSS 链接： https://xiaoling-public.oss-cn-hangzhou.aliyuncs.com/os_performance_optimization.py
OS 离 线 修 复 脚 本： https://public-beijing.oss-cn-beijing.aliyuncs.com/oos/oos_noak.sh  

### PAM不让你登陆

问题描述： ssh 可以登陆，管理终端无法登陆 root， 提示 login in...  

先通过 ssh 方式登录系统，查看登录日志是否有异常。  

```shell
cat /var/log/secure
Jun 2 09:26:48 iZbp1begsz1x269nxhtip4Z login: FAILED LOGIN 1 FROM tty1 FOR root，Authentication failure
```

似乎是 login 验证模块的问题进一步查看对应的配置文件 /etc/pam.d/login。  

```shell
# cat /etc/pam.d/login
#%PAM-1.0
auth required pam_succeed_if.so user != root quiet
auth [user_unknown=ignore success=ok ignore=ignore default=bad] pam_
securetty.so
auth substack system-auth
auth include postlogin
account required pam_nologin.so
account include system-auth
password include system-auth
# pam_selinux.so close should be the first session rule
session required pam_selinux.so close
session required pam_loginuid.so
session optional pam_console.so
# pam_selinux.so open should only be followed by sessions to be executed in
the user context
session required pam_selinux.so open
session required pam_namespace.so
session optional pam_keyinit.so force revoke
session include system-auth
session include postlogin
-session optional pam_ck_connector.so
```

其中一行的作用为禁止本地登录，可以将其注释掉即可  

```shell
auth required pam_succeed_if.so user != root quiet
```

### CentOS登陆卡住

问题描述：系统登陆卡住，需要 ctrl +c 才能进去，如图

![image-20210303110730675](https://gitee.com/c_honghui/picture/raw/master/img/20210303110730.png)

如果一直等的话，会提示如下截图：  

![image-20210303111152869](https://gitee.com/c_honghui/picture/raw/master/img/20210303111153.png)

原因：

/etc/profile 里面有 source /etc/profile 引起死循环，注释即可。  

## Linux性能问题

### 如何找出系统中 load 高时处于运行队列的进程  

有可能系统有很高的负载但是 CPU 使用率却很低，或者负载很低而 CPU 利用率很高  

每隔 1s 统计一次：  

```shell
#!/bin/bash
LANG=C
PATH=/sbin:/usr/sbin:/bin:/usr/bin
interval=1
length=86400
for i in $(seq 1 $(expr ${length} / ${interval}));do
date
LANG=C ps -eTo stat,pid,tid,ppid,comm --no-header | sed -e 's/^ \*//' |
perl -nE 'chomp;say if (m!^\S*[RD]+\S*!)'
date
cat /proc/loadavg
echo -e "\n"
sleep ${interval}
done
```

从统计出来的结果可以看到：  

```shell
at Jan 20 15:54:12 CST 2018
D 958 958 957 nginx
D 959 959 957 nginx
D 960 960 957 nginx
D 961 961 957 nginx
R 962 962 957 nginx
D 963 963 957 nginx
D 964 964 957 nginx
D 965 965 957 nginx
D 966 966 957 nginx
D 967 967 957 nginx
D 968 968 957 nginx
D 969 969 957 nginx
D 970 970 957 nginx
D 971 971 957 nginx
D 972 972 957 nginx
D 973 973 957 nginx
D 974 974 957 nginx
R 975 975 957 nginx
D 976 976 957 nginx
D 977 977 957 nginx
D 978 978 957 nginx
D 979 979 957 nginx
R 980 980 957 nginx
D 983 983 957 nginx
D 984 984 957 nginx
D 985 985 957 nginx
D 986 986 957 nginx
D 987 987 957 nginx
D 988 988 957 nginx
D 989 989 957 nginx
R 11908 11908 18870 ps
Sat Jan 20 15:54:12 CST 2018
25.76 20.60 19.00 12/404 11912
注： R 代表运行中的队列， D 是不可中断的睡眠进程
```

在 load 比较高的时候，有大量的 nginx 处于 R 或者 D 状态，他们才是造成 load 上升的元凶，和我们底层的负载确实是没有关系的。  

查 CPU 使用率比较高的线程小脚本：  

```shell
#!/bin/bash
LANG=C
PATH=/sbin:/usr/sbin:/bin:/usr/bin
interval=1
length=86400
for i in $(seq 1 $(expr ${length} / ${interval}));do
date
LANG=C ps -eT -o%cpu,pid,tid,ppid,comm | grep -v CPU | sort -n -r | head -20
date
LANG=C cat /proc/loadavg
{ LANG=C ps -eT -o%cpu,pid,tid,ppid,comm | sed -e 's/^ *//' | tr -s ' ' |
grep -v CPU | sort -n -r | cut -d ' ' -f 1 | xargs -I{} echo -n "{} + " &&
echo '0'; } | bc -l
sleep ${interval}
done
fuser -k $0
```

### 内存去哪了

背景： 收到报警，系统的内存使用率触发阈值  

![image-20210303113031265](https://gitee.com/c_honghui/picture/raw/master/img/20210303113031.png)

1. 登陆系统，使用命令查看内存分配  

   top 按M

   ![image-20210303113351708](https://gitee.com/c_honghui/picture/raw/master/img/20210303113351.png)

   free -m

   ![image-20210303113442724](https://gitee.com/c_honghui/picture/raw/master/img/20210303113442.png)

   atop看下内存分配（cat /proc/meminfo 也可以看到一些细化的内存使用信息）。  

   ![image-20210303114359952](https://gitee.com/c_honghui/picture/raw/master/img/20210303114400.png)

   

2. 发现 cache 才 1.7G，slab 非常高，4.4G，slab 内存简单理解为是系统占用的。使用 slabtop 继续分析。  

![image-20210303114730778](https://gitee.com/c_honghui/picture/raw/master/img/20210303114730.png)

3.	 看到 proc_inode_cache 使用的最多，这个代表是 proc 文件系统的 inode 的占用的。  

4. 查进程，但是进程不多，再查线程，可以通过如下命令进行检查。

   ps -eLf  

   ![image-20210303115214482](https://gitee.com/c_honghui/picture/raw/master/img/20210303115214.png)

​        计算 socket。  

```shell
ll /proc/22360/task/*/fd/ |grep socket |wc -l
140
```

​        计算有多少fd

```shell
ll /proc/22360/task/*/fd/ |wc -l
335
```

5. 每个 socket 的 inode 也不一样。  

   当时看到的现场有几万个 fd，基本全是 socket，每个 inode 都是占用空间的，且 proc 文件系统是全内存的。 所以我们才会看到 slab 中 proc_inode_cache内存占用高。  

### IO异常

简介： 遇到一个 IO 异常飙升的问题，IO 起飞后系统响应异常缓慢，看不到现场一直无法定位问题，检查对应时间点应用日志也没有发现异常的访问  

1. 确认IO发生在系统盘还是数据盘

   ```shell
   # iostat -d 3 -k -x -t 30
   06/12/2018 09:52:33 AM
   Device: rrqm/s wrqm/s r/s w/s rkB/s wkB/s avgrq-sz
   avgqu-sz await svctm %util
   xvda 0.00 0.39 0.08 0.70 1.97 5.41 18.81
   0.03 44.14 1.08 0.08
   xvdb 0.00 0.00 0.00 0.00 0.00 0.00 8.59
   0.00 1.14 1.09 0.00
   06/12/2018 09:52:36 AM
   
   Device: rrqm/s wrqm/s r/s w/s rkB/s wkB/s avgrq-sz
   avgqu-sz await svctm %util
   xvda 0.00 0.00 0.00 0.67 0.00 2.68 8.00
   0.00 1.00 0.50 0.03
   xvdb 0.00 0.00 0.00 0.00 0.00 0.00 0.00
   0.00 0.00 0.00 0.00
   ```

   每隔 3 秒采集一次磁盘 io，输出时间，一共采集 30 次，想一直抓的话把 30 去掉即可，注意磁盘空余量。  

   通过这个命令我们可以确认如下信息：
   \- 问题发生的时间
   \- 哪块盘发生的 io
   \- 磁盘的 IOPS（ r/s w/s）以及吞吐量（ rkB/s wkB/s ）  

2. 确认哪块盘发生了 IO 还不够，再抓一下发生 IO 的进程，需要安装 iotop 进行捕获  

   ```shell
   # iotop -b -o -d 3 -t -qqq -n 30
   10:18:41 7024 be/4 root 0.00 B/s 2.64 M/s 0.00 % 93.18 % fio
   -direct=1 -iodepth=128 -rw=randwrite -ioengine=libaio -bs=1k -size=1G
   -numjobs=1 -runtime=1000 -group_reporting -filename=iotest -name=Rand_
   Write_Testing
   ```

   每隔 3 秒采集一次，一共采集 30 次，静默模式，只显示有 io 发生的进程
   通过这个命令我们可以确认如下信息：
   \- 问题发生的时间
   \- 产生 IO 的进程 id 以及进程参数（ command）
   \- 进程产生的吞吐量（如果有多个可以把 qqq 去掉可显示总量）
   \- 进程占用当前 IO 的百分比  

3. 使用 fio 进行 IO 压测，具体参数可以参考块存储性能

   ```shell
   fio -direct=1 -iodepth=128 -rw=randwrite -ioengine=libaio -bs=1k -size=1G
   -numjobs=1 -runtime=1000 -group_reporting -filename=iotest -name=Rand_
   Write_Testing
   Rand_Write_Testing: (g=0): rw=randwrite， bs=1K-1K/1K-1K/1K-1K，
   ioengine=libaio， iodepth=128
   fio-2.0.13
   Starting 1 process
   ^Cbs: 1 (f=1): [w] [8.5% done] [0K/2722K/0K /s] [0 /2722 /0 iops] [eta
   05m:53s]
   fio: terminating on signal 2
   Rand_Write_Testing: (groupid=0， jobs=1): err= 0: pid=11974: Tue Jun 12
   10:36:30 2018
   write: io=88797KB， bw=2722.8KB/s， iops=2722 ， runt= 32613msec
   ......
   ```

   通过输出结果的时间点进行对比分析，相对于 fio 的 IOPS，吞吐量跟 iotop 以及iostat 监控到的数据是一致。

## Linux网络问题