# Ansible入门

## 简介

Ansible ⽤ Python 编写，尽管市⾯上已经有很多可供选择的 配置管理解决⽅案（例如 Salt、Puppet、Chef等),但它们各有优 劣，⽽Ansible的特点在于它的简洁。让 Ansible 在主流的配置管理 系统中与众不同的⼀点便是，它并不需要你在想要配置的每个节点上安 装⾃⼰的组件。同时提供的另⼀个优点，如果需要的话，你可以在不⽌ ⼀个地⽅控制你的整个基础架构。

总结一句话：ansible 就是⽤什么模块，让谁去⼲什么事情。

## 原理

![image-20210305222545959](https://gitee.com/c_honghui/picture/raw/master/img/20210305222553.png)

1、在ANSIBLE 管理体系中，存在"管理节点" 和 "被管理节点" 两 种⻆⾊。 
2、被管理节点通常被称为"资产" 
3、在管理节点上，Ansible将 AdHoc 或 PlayBook 转换为Python 脚本。 并通过SSH将这些Python 脚本传递到被管理服务器上。 在被管理服务器上依次执⾏，并实时的将结果返回给管理节点。

## 安装

![image-20210305222750080](https://gitee.com/c_honghui/picture/raw/master/img/20210305222750.png)

yum ⽅式

```shell
[root@qfedu.com ~]# yum install epel-release
[root@qfedu.com ~]# yum install ansible
```

pip ⽅式

```shell
[root@qfedu.com ~]# yum install epel-release
[root@qfedu.com ~]# yum install python2-pip
[root@qfedu.com ~]# pip install ansible
#这⾥是使⽤系统⾃带的 python2 的环境
#如果系统中安装的 pip ，可以直接使⽤ pip 安装 ansible
```

查看版本

```shell
[root@qfedu.com ~]# ansible --version
ansible 2.9.6
 config file = /etc/ansible/ansible.cfg
 configured module search path =
[u'/root/.ansible/plugins/modules',
u'/usr/share/ansible/plugins/modules']
 ansible python module location =
/usr/lib/python2.7/site-packages/ansible
 executable location = /usr/bin/ansible
 python version = 2.7.5 (default, Aug 7
2019, 00:51:29) [GCC 4.8.5 20150623 (Red Hat
4.8.5-39)]
```

## 管理节点与被管理节点建⽴SSH 信 任关系

管理节点（ansible）中创建密钥对

```shell
[root@qfedu.com ~]# ssh-keygen -t rsa
```

将本地的公钥传输到被管理节点

```shell
[root@qfedu.com ~]# ssh-copy-id root@172.18.0.3
```

## ansible资产

###  静态资产

默认情况下，Ansible的资产⽂件位于 /ect/ansible/hosts。 pip 安装的可能没有这个⽂件，创建⼀个即可。

#### 自定义资产

```shell
# cat /ect/ansible/hosts
1.1.1.1
2.2.2.2
3.3.3.[1:15]
test01.qfedu.com
test03.qfedu.com
test[05:09].qfedu.com

[web_servers]
192.168.1.2
192.168.1.3
192.168.1.5

[dbdb_servers]
192.168.2.2
192.168.2.3
192.168.1.5

[alldb_servers]
[alldb_servers:children]
dbdb_servers
web_servers
```

1. Ansible 的资产⽂件中，可以以IP地址的形式或者主机名的形式存在。 
2. Ansible 的资产若连续，可以使⽤[stat:end] 的形式去表达。 
3. 可以将服务器按照业务场景定义成组，⽐如dbdb_servers 和 web_servers 
4. 组和组之间可以存在继承关系，⽐如dbdb_servers 和 web_servers 同时继承 alldb_servers 组

#### 使用自定义资产

列举出所有资产

```shell
# ansible all [-i inventory.ini] --list-hosts
 hosts (29):
 1.1.1.1
 2.2.2.2
 3.3.3.1
 ...略...
```

列举出选定资产

```shell
# ansible web_servers -i inventory.ini --listhosts
 hosts (3):
 192.168.2.2
 192.168.2.3
 192.168.1.5
```

## Ad-Hoc 命令

Ansible提供两种⽅式去完成任务,⼀是 ad-hoc 命令,⼀是写 Ansible playbook
前者可以解决⼀些简单的任务, 后者解决较复杂的任务，⽐如做配 置管理或部署。

### 命令格式

```shell
ansible pattern [-i inventory] -m module -a argument

pattern 资产选择器
-i 指定资产清单⽂件的位置
-m 指定本次Ansible ad-hoc 要执⾏的模块。可以类别成
SHELL 中的命令。
-a 模块的参数. 可以类⽐成SHELL 中的命令参数
```

### 模块类型

Ansible 模块分三种类型: 核⼼模块(core module)、附加模块 (extra module)及⽤户⾃定义模块(consume module)。

列举出所有的核⼼模块和附加模块

```shell
# ansible-doc -l
```

查询某个模块的使⽤⽅法

```shell
# ansible-doc modulename
或
# ansible-doc -s modulename
```

### 常⽤模块

**command & shell 模块**

两个模块都是在远程服务器上去执⾏命令。

shell 模块可以执⾏SHELL 的内置命令和 特性（⽐如管道 符）。 
command 模块⽆法执⾏SHELL 的内置命令和特性

```shell
# ansible all -i hosts -m shell -a "echo 'hello'|grep -o 'e'"
172.18.0.3 | CHANGED | rc=0 >>
e
172.18.0.4 | CHANGED | rc=0 >>
e
# ansible all -i hosts -a "echo 'hello'|grep -o 'e'"
172.18.0.4 | CHANGED | rc=0 >>
hello|grep -o e
172.18.0.3 | CHANGED | rc=0 >>
hello|grep -o e
```

**script 模块**

将管理节点上的脚本传递到被管理节点(远程服务器)上进⾏执⾏。

```shell
[root@qfedu.com ~]# ansible webservers -i hosts -m script -a "/root/a.sh"
172.18.0.4 | CHANGED => {
 "changed": true,
 "rc": 0,
 "stderr": "Shared connection to 172.18.0.4
closed.\r\n",
 "stderr_lines": [
 "Shared connection to 172.18.0.4 closed."
 ],
 "stdout": "",
 "stdout_lines": []
}
```

**copy 模块**

要⽤于管理节点和被管理节点之间的⽂件拷⻉

常⽤参数: 

- src 指定拷⻉⽂件的源地址 
- dest 指定拷⻉⽂件的⽬标地址 
- backup 拷⻉⽂件前，若原⽬标⽂件发⽣了变化，则对⽬标⽂件进⾏备份 
- woner 指定新拷⻉⽂件的所有者 
- group 指定新拷⻉⽂件的所有组 
- mode 指定新拷⻉⽂件的权限

```shell
# ansible webservers -i hosts -m copy -a "src=./nginx.repo dest=/etc/yum.repos.d/nginx.repo"
```

copy 前， 在被管理节点上对原⽂件进⾏备份

```shell
# ansible all -i hosts -m copy -a "src=./nginx.repo dest=/etc/yum.repos.d/nginx.repo backup=yes"
```

copy ⽂件的同时对⽂件进⾏⽤户及⽤户组设置

```shell
# ansible all -i hosts -m copy -a "src=./nginx.repo dest=/etc/yum.repos.d/nginx.repo owner=nobody group=nobody"
```

copy ⽂件的同时对⽂件进⾏权限设置

```shell
# ansible all -i hosts -m copy -a "src=./nginx.repo dest=/etc/yum.repos.d/nginx.repo mode=0755"
```

**yum_repository模块**

添加yum仓库

常⽤参数 

- name 仓库名称，就是仓库⽂件中第⼀⾏的中括号中名称，必 须的参数。 
- description 仓库描述信息，添加时必须的参数 
- baseurl yum存储库 “repodata” ⽬录所在⽬录的URL，添加 时必须的参数。 它也可以是多个URL的列表。 
- file 仓库⽂件保存到被管理节点的⽂件名，不包含 .repo。 默认是 name 的值。
- state preset 确认添加仓库⽂件， absent 确认删除仓库⽂ 件。 
- gpgcheck 是否检查 GPG yes|no， 没有默认值，使 ⽤/etc/yum.conf 中的配置。

添加epel源

```shell
[root@qfedu.com ~]# ansible dbservers -i hosts -m yum_repository -a "name=epel
baseurl='https://download.fedoraproject.org/pub/epel/ $releasever/$basearch/' description='EPEL YUM repo'"
```

**yum模块**

等同于 Linux 上的YUM 命令

常⽤参数: 

- name 要安装的软件包名， 多个软件包以英⽂逗号(,) 隔开 
- state 对当前指定的软件安装、移除操作(present installed latest absent removed) ⽀持的参数 - present 确认已经安 装，但不升级 - installed 确认已经安装 - latest 确保安装，且 升级为最新 - absent 和 removed 确认已移除

安装⼀个软件包

```shell
# ansible webservers -i hosts -m yum -a "name=nginx state=present"
```

移除⼀个软件包

```shell
# ansible webservers -i hosts -m yum -a "name=nginx state=absent"
```

**systemd 模块**

Centos6 之前的版本使⽤ service 模块。

管理远程节点上的 systemd 服务

常⽤参数： 

- daemon_reload 重新载⼊ systemd，扫描新的或有变动的单 元 
- enabled 是否开机⾃启动 yes|no 
- name 必选项，服务名称 ，⽐如 httpd vsftpd\
- state 对当前服务执⾏启动，停⽌、重启、重新加载等操作 (started,stopped,restarted,reloaded)

重新加载 systemd

```shell
# ansible webservers -i hosts -m systemd -a "daemon_reload=yes"
```

启动 Nginx 服务

```shell
# ansible webservers -i hosts -m systemd -a "name=nginx state=started"
```

**group 模块**

在被管理节点上，对组进⾏管理。

常⽤参数: 

- name 组名称， 必须的 
- system 是否为系统组, yes/no ， 默认是 no 
- state 删除或这创建，present/absent ，默认是present

创建普通组 db_admin

```shell
# ansible dbservers -i hosts -m group -a "name=db_admin"
```

**user 模块**

⽤于在被管理节点上对⽤户进⾏管理。

常⽤参数： 

- **name** 必须的参数， 指定⽤户名 
- password 设置⽤户的密码，这⾥接受的是⼀个加密的值，因 为会直接存到 shadow, 默认不设置密码 
- update_password 假如设置的密码不同于原密码，则会更新 密码. 在 1.3 中被加⼊ 
- home 指定⽤户的家⽬录 
- shell 设置⽤户的 shell
- comment ⽤户的描述信息 
- create_home 在创建⽤户时，是否创建其家⽬录。默认创建， 假如不创建，设置为 no。2.5版本之前使⽤ createhome 
- group 设置⽤户的主组 
- groups 将⽤户加⼊到多个其他组中，多个⽤逗号隔开。 默认会把⽤户从其他已经加⼊的组中删除。 
- append yes|no 和 groups 配合使⽤，yes 时， 不会把⽤户从其他已经加⼊的组中删除 
- system 设置为 yes 时，将会创建⼀个系统账号 
- expires 设置⽤户的过期时间，值为时间戳,会转为为天数后， 放在 shadow 的第 8 个字段⾥ 
- generate_ssh_key 设置为 yes 将会为⽤户⽣成密钥，这不会 覆盖原来的密钥 
- ssh_key_type 指定⽤户的密钥类型， 默认 rsa, 具体的类型取 决于被管理节点 
- **state** 删除或添加⽤户, present 为添加，absent 为删除； 默认值 present 
- remove 当与 state=absent ⼀起使⽤，删除⼀个⽤户及关联的 ⽬录， ⽐如家⽬录，邮箱⽬录。可选的值为: yes/no

 **file 模块**

⽤于远程主机上的⽂件操作

常⽤参数:

- owner 定义⽂件/⽬录的属主
- group 定义⽂件/⽬录的属组
- mode 定义⽂件/⽬录的权限
- path 必选项，定义⽂件/⽬录的路径
- recurse 递归的设置⽂件的属性，只对⽬录有效
- src 链接(软/硬)⽂件的源⽂件路径，只应⽤于state=link的情况
- dest 链接⽂件的路径，只应⽤于state=link的情况
- state
  - directory 如果⽬录不存在，创建⽬录
  - file ⽂件不存在，则不会被创建，存在则返回⽂件的信 息，常⽤于检查⽂件是否存在。
  - link 创建软链接
  - hard 创建硬链接
  - touch 如果⽂件不存在，则会创建⼀个新的⽂件，如果⽂件或⽬录 已存在，则更新其最后修改时间
  - absent 删除⽬录、⽂件或者取消链接⽂件



```shell
// 创建⼀个⽂件
# ansible all -i hosts -m file -a
"path=/tmp/foo.conf state=touch"
// 改变⽂件所有者及权限
# ansible all -i hosts -m file -a
"path=/tmp/foo.conf owner=nobody group=nobody
mode=0644"
// 创建⼀个软连接
# ansible all -i hosts -m file -a "src=/tmp/foo.conf
dest=/tmp/link.conf state=link"
// 创建⼀个⽬录
# ansible all -i hosts -m file -a "path=/tmp/testdir
state=directory"
// 取消⼀个连接
# ansible all -i hosts -m file -a
"path=/tmp/link.conf state=absent"
// 删除⼀个⽂件
# ansible all -i hosts -m file -a
"path=/tmp/foo.conf state=absent"
```

**cron 模块**

管理远程节点的CRON 服务

注意：使⽤ Ansible 创建的计划任务，是不能使⽤本地 crontab -e去编辑，否则 Ansible ⽆法再次操作此计划任务了。

常⽤参数:

- name 指定⼀个cron job 的名字。⼀定要指定，便于⽇之后删 除。
- minute 指定分钟，可以设置成(0-59, *, */2 等)格式。 默认是 * , 也就是每分钟。
- hour 指定⼩时，可以设置成(0-23, *, */2 等)格式。 默认是 * , 也就是每⼩时。
- day 指定天， 可以设置成(1-31, *, */2 等)格式。 默认是 * , 也 就是每天。
- month 指定⽉份， 可以设置成(1-12, *, */2 等)格式。 默认是 * , 也就是每周。
- weekday 指定星期， 可以设置成(0-6 for Sunday-Saturday, * 等)格式。默认是 *，也就是每星期。
- job 指定要执⾏的内容，通常可以写个脚本，或者⼀段内容。
- state 指定这个job的状态，可以是新增(present)或者是删除 (absent)。 默认为新增(present)

```shell
// 新建⼀个 CRON JOB 任务
# ansible all -i hosts -m cron -a "name='create new
job' minute='0' job='ls -alh > /dev/null'"
// 删除⼀个 CRON JOB 任务，删除时，⼀定要正确指定job 的name
参数，以免误删除。
# ansible all -i hosts -m cron -a "name='create new
job' state=absent"
```

登录任何⼀台管理机验证cron

```shell
# crontab -l
#Ansible: create new job
0 * * * * ls -alh > /dev/null
```

**debug模块**

⽤于调试时使⽤，通常的作⽤是将⼀个变量的值 给打印出来。

常⽤参数: 

- var 直接打印⼀个指定的变量值 
- msg 打印⼀段可以格式化的字符串

**template 模块**

可以进⾏⽂档内 变量的替换

```shell
1. 建⽴⼀个 template ⽂件, 名为 hello_world.j2
# cat hello_world.j2
Hello {{var}} !
2. 执⾏命令，并且设置变量 var 的值为 world
# ansible all -i hosts -m template -a
"src=hello_world.j2 dest=/tmp/hello_world.world" -e
"var=world"
```

