# playbook

## playbook简介

AD-HOC 每次只能在被管理节 点上执行简单的命令。Ansible引进了 PLAYBOOK 来帮忙我们解决这样复 杂问题。

Playbook 也通常被翻译成剧本。

PlayBook遵循YAML 的语法格式。

### yaml

#### yaml特点

- 以 # 为注释符 
- 以 .yml 或者.yaml 结尾 
- 以 --- 开始 , 以 ... 结束, 但开始和结束标志都是可选的
- 大小写敏感 
- 使⽤缩进表示层级关系 
- 缩进时是使用Tab键还是使用空格一定要达到统一，建议使用 空格。 
- 相同层级的元素必须左侧对齐即可

#### 数据结构

**字符串**

```yaml
---
# YAML 中的字符串可以不使用引号，即使中间存在空格的时候，当然了使用单引号和双引号也没有错。
this is a string
'this is a string'
"this is a string"
# YAML 中若一行写不完你要表述的内容的时候，可以进⾏折⾏。写法
如下:
long_line: |
 Example 1
 Example 2
 Example 3
# 或者
long_line: >
 Example 1
 Example 2
 Example 3
...
```

**列表**

```yaml
---
# 若熟悉 Python 的话， 可以认为它就是Python中的List ,若熟
悉 C 语⾔的话， 可以认为它是 C 中的数组。
# 如何定义: 以短横线开头 + 空格 + 具体的值
- red
- green
- blue
# 以上的值假如转换成 python 的 List 会是这样：
# ['red', 'green', 'blue']
...
```

**字典**

```yaml
---
# 若熟悉 Python 的话， 可以认为它就是 Python 中的 Dict
# 如何定义: key + 冒号(:) + 空格 + 值(value), 即 key:
value
name: Using Ansible
code: D1234
# 转换为 python 的 Dict
# {'name': 'Using Ansibel', 'code': 'D1234'}
...
```

## Playbook 的编写

### Play 的定义

1、每个Play 都是以短横杠开始的 

2、每 个Play 都是一个YAML 字典格式

```yaml
---
- key1: value1
 key2: value2
 key3: value3
...
```

多个play

```yaml
# 含有3个Play 的伪PlayBook构成
- key1: value1
 key2: value2
 key3: value3
- key4: value1
 key5: value2
 key6: value3
- key1: value1
 key2: value2
 key3: value3
...
```

### Play 属性

Play中的每个key，如 key1, key2 等；这些key在PlayBook中被定义为Play的属性。

常用属性:

- name 属性， 每个play的名字 
- hosts 属性, 每个play 涉及的被管理服务器，同ad-hoc 中的资 产选择器 
- tasks 属性, 每个play 中具体要完成的任务，以列表的形式表达 
- become 属性，如果需要提权，则加上become 相关属性 
- become_user 属性, 若提权的话，提权到哪个用户上 
- remote_user属性，指定连接到远程节点的用户，就是在**远程服务器上执行具体操作的用户**。若不指定，则默认使用当前执行 ansible Playbook 的用户

### 一个完整剧本

```yaml
---
- name: the first play example
  hosts: webservers
  remote_user: root
  tasks:
  - name: install nginx package
    yum: name=nginx state=present
  - name: copy nginx.conf to remote server
    copy: src=nginx.conf
dest=/etc/nginx/nginx.conf
  - name: start nginx server
    service:
      name: nginx
      enabled: true
      state: started
```

### Playbook校验

只能校验PlayBook是否正确，不能校验YAML 文件是否语法正确。

```shell
# ansible-playbook -i hosts myplaybook.yml --syntaxcheck
```

检查YAML的 语法格式的⽅法进⾏检查PlayBook的语法正确性。

```shell
# python -c 'import yaml,sys; print yaml.safe_load(sys.stdin)' < myplaybook.yml
```

### 运行PlayBook

```shell
# ansible-playbook -i hosts myplaybook.yml
```

### 单步跟从调试PlayBook

```shell
// 执行Task中的任务，需要⼿动确认是否往下执⾏。
# ansible-playbook -i hosts myplaybook.yml --step
```

### 测试运行PlayBook

会执行完整个PlayBook ,但是所有Task中的行为都不 会在远程服务器上执行，所有执行都是模拟行为。

```shell
# ansible-playbook -i hosts myplaybook.yml -C
// -C 为大写的字母 C
```

## Ansible-变量

### 变量规则

保留关键字不能作为变量名称

```text
add, append, as_integer_ratio, bit_length,
capitalize, center, clear,
conjugate, copy, count, decode, denominator,
difference,
difference_update, discard, encode, endswith,
expandtabs,
extend, find, format, fromhex, fromkeys, get,
has_key,
hex, imag, index, insert, isalnum, intersection,
intersection_update, isalpha, isdecimal, isdigit,
isdisjoint, is_integer, islower,
isnumeric, isspace, issubset, issuperset, istitle,
isupper,
items, iteritems, iterkeys, itervalues, join, keys,
ljust, lower,
 lstrip, numerator, partition, pop, popitem, real,
remove,
 replace, reverse, rfind, rindex, rjust, rpartition,
rsplit, rstrip,
 setdefault, sort, split, splitlines, startswith,
strip, swapcase,
symmetric_difference, symmetric_difference_update,
title,
translate, union, update, upper, values, viewitems,
viewkeys,
viewvalues, zfill
```

### 变量类型

- 全局变量 
- 剧本变量 
- 资产变量

#### 全局变量

使用ansible 或使用ansible-playbook 时，**动通过 -e 参数**传递给Ansible 的变量。

传递普通的key=value 的形式：

```shell
# ansible all -i localhost, -m debug -a "msg='my key is {{ key }}'" -e "key=value"
```

传递⼀个YAML/JSON 的形式:

```shell
# cat a.yml
---
name: qfedu
type: school
...
# ansible all -i localhost, -m debug -a "msg='name is {{ name }}, type is {{ type }}'" -e @a.yml
```

#### 剧本变量

定义在PlayBook中的

通过PLAY 属性 vars 定义：

```yaml
---
- name: test play vars
  hosts: all
  vars:
    user: lilei
    home: /home/lilei
```

通过PLAY 属性 vars_files 定义:

```yaml
# 当通过vars属性定义的变量很多时，这个Play就会感觉特别臃肿。此时我们可以将变量单独从Play中抽离出来，形成单独的YAML 文件。
---
- name: test play vars
 hosts: all
 vars_files:
   - vars/users.yml
# cat vars/users.yml
---
user: lilei
home: /home/lilei
```

使用变量：

在playbook中使用 {{ 变量名 }} 来使⽤变量

```yaml
---
- name: test play vars
 hosts: all
 vars:
   user: lilei
   home: /home/lilei
 tasks:
   - name: create the user {{ user }}
     user:
       name: "{{ user }}"
       home: "{{ home }}"
#注意加双引号
```

#### 资产变量

主机变量

此变量只针对 172.18.0.3 这台服务器有效

```shell
# cat hostsandhostvars
[webservers]
172.18.0.3 user=lilei port=3309
172.18.0.4
```

 主机组变量

针对 webservers 这个主机组中的所有服务器有效

```shell
[webservers:vars]
home="/home/lilei"
```

主机变量 user 的优先级更高。

Inventory 内置变量

内置变量几乎都是以 ansible_ 为前缀

```text
ansible_ssh_host
 将要连接的远程主机名与你想要设定的主机的别名不同的话,可
通过此变量设置.
ansible_ssh_port
 ssh端⼝号.如果不是默认的端⼝号,通过此变量设置.
ansible_ssh_user
 默认的 ssh ⽤户名
ansible_ssh_pass
ssh 密码(这种⽅式并不安全,官⽅强烈建议使⽤ --askpass 或 SSH 密钥)
ansible_sudo_pass
 sudo 密码(这种⽅式并不安全,官⽅强烈建议使⽤ --asksudo-pass)
ansible_sudo_exe (new in version 1.8)
 sudo 命令路径(适⽤于1.8及以上版本)
ansible_ssh_private_key_file
 ssh 使⽤的私钥⽂件.适⽤于有多个密钥,⽽你不想使⽤ SSH
代理的情况.
ansible_python_interpreter
 ⽬标主机的 python 路径.适⽤于的情况: 系统中有多个
Python, 或者命令路径不是"/usr/bin/python",⽐如
/usr/local/bin/python3
```

#### Facts变量

它的声明和赋值完全有Ansible 中的 setup 模块帮我们完成。

它收集了有关被管理服务器的操作系统版本、服务器IP地址、主机 名，磁盘的使⽤情况、CPU个数、内存⼤⼩等等有关被管理服务器的私 有信息。

```shell
# ansible all -i localhost, -c local -m setup
```

#### 注册变量

往往⽤于保存⼀个task任务的执⾏结果, 以便于debug时使⽤。 或者将此次task任务的结果作为条件，去判断是否去执⾏其他task 任务。 

注册变量在PlayBook中通过register关键字去实现。

```yaml
---
- name: install a package and print the result
  hosts: webservers
  remote_user: root
  tasks:
    - name: install nginx package
      yum: name=nginx state=present
      register: install_result
    - name: print result
      debug: var=install_result
```

## 任务控制

### 条件判断

例子：

Nginx 语法校验

```yaml
- name: check nginx syntax
  shell: /usr/sbin/nginx -t
```

如何将Nginx语法检查的TASK同Nginx启动的TASK关联起来:

获取Task任务结果

```yaml
- name: check nginx syntax
  shell: /usr/sbin/nginx -t
  register: nginxsyntax
```

通过debug模块去确认返回结果的数据结构

```yaml
- name: print nginx syntax result
  debug: var=nginxsyntax
```

通过debug 模块，打印出来的返回结果。 当nginxsyntax.rc 为 0 时语法校验正确。

通过条件判断(when) 指令去使⽤语法校验的结果

```yaml
- name: check nginx syntax
  shell: /usr/sbin/nginx -t
  register: nginxsyntax
- name: print nginx syntax
  debug: var=nginxsyntax
- name: start nginx server
  service: name=nginx state=started
  when: nginxsyntax.rc == 0
```

另外 when ⽀持如下运算符:

```text
==
!=
> >=
< <=
is defined
is not defined
true
false
⽀持逻辑运算符: and or
```

### 循环控制

在PlayBook中使⽤with_items 去实现循环控制，且循环时的中间变量只能是关键字 item ，⽽不能随意⾃定义。

在这⾥使⽤定义了剧本变量 createuser(⼀个列表) ，然后通过 with_items 循环遍历变量这个变量来达到创建⽤户的⽬的。

```yaml
- name: variable playbook example
  hosts: webservers
  gather_facts: no
  vars:
    createuser:
    - tomcat
    - www
    - mysql
  tasks:
    - name: create user
      user: name={{ item }} state=present
      with_items: "{{ createuser }}"
```

### tags属性

通过Play中的tags 属性，去解决⽬前PlayBook变更⽽导 致的扩⼤变更范围和变更⻛险的问题。

改进后的playbook

```yaml
- name: tags playbook example
  hosts: webservers
  gather_facts: no
  vars:
    createuser:
    - tomcat
    - www
    - mysql
  tasks:
  - name: create user
    user: name={{ item }} state=present
    with_items: "{{ createuser }}"
    
  - name: yum nginx webserver
    yum: name=nginx state=present
    
  - name: update nginx main config
    copy: src=nginx.conf dest=/etc/nginx/
    tags: updateconfig
    
  - name: add virtualhost config
    copy: src=www.qfedu.com.conf dest=/etc/nginx/conf.d/
    tags: updateconfig
    
  - name: check nginx syntax
    shell: /usr/sbin/nginx -t
    register: nginxsyntax
    tags: updateconfig
    
  - name: check nginx running
    stat: path=/var/run/nginx.pid
    register: nginxrunning
    tags: updateconfig
    
  - name: print nginx syntax
    debug: var=nginxsyntax
    
  - name: print nginx syntax
    debug: var=nginxrunning
    
  - name: reload nginx server
    service: name=nginx state=started
    when: nginxsyntax.rc == 0 and nginxrunning.stat.exists == true
    tags: updateconfig
    
  - name: start nginx server
    service: name=nginx state=started
    when:
    - nginxsyntax.rc == 0
    - nginxrunning.stat.exists == false
    tags: updateconfig
```

指定tags 去执⾏PlayBook：

执⾏的过程中只会执⾏task 任务 上打上tag 标记为 updateconfig 的任务：

```shell
# ansible-playbook -i hosts site.yml -t updateconfig
```

### Handlers 属性

上面的playbook中，当我的配置⽂件没有发⽣变化 时，每次依然都会去触发TASK "reload nginx server"。

如何只有配置⽂件发⽣变化的时候才去触发TASK "reload nginx server"，此时可以使⽤ handlers 属性。

改进PlayBook：

```yaml
- name: tags playbook example
  hosts: webservers
  gather_facts: no
  vars:
    createuser:
    - tomcat
    - www
    - mysql
  tasks:
  - name: create user
    user: name={{ item }} state=present
    with_items: "{{ createuser }}"
    
  - name: yum nginx webserver
    yum: name=nginx state=present
    
  - name: update nginx main config
    copy: src=nginx.conf dest=/etc/nginx/
    tags: updateconfig
    notify: reload nginx server
    
  - name: add virtualhost config
    copy: src=www.qfedu.com.conf dest=/etc/nginx/conf.d/
    tags: updateconfig
    notify: reload nginx server
    
  - name: check nginx syntax
    shell: /usr/sbin/nginx -t
    register: nginxsyntax
    tags: updateconfig
    
  - name: check nginx running
    stat: path=/var/run/nginx.pid
    register: nginxrunning
    tags: updateconfig
    
  - name: print nginx syntax
    debug: var=nginxsyntax
    
  - name: print nginx syntax
    debug: var=nginxrunning
    
  - name: reload nginx server
    service: name=nginx state=started
    when: nginxsyntax.rc == 0 and nginxrunning.stat.exists == true
    tags: updateconfig
    
  - name: start nginx server
    service: name=nginx state=started
    when:
    - nginxsyntax.rc == 0
    - nginxrunning.stat.exists == false
  handlers:
    - name: reload nginx server
      service: name=nginx state=reloaded
      when:
        - nginxsyntax.rc == 0
        - nginxrunning.stat.exists == true
```

当Ansible 认为⽂件的内容发⽣了变化(⽂件MD5发⽣变化了)，它就会 发送⼀个通知信号，通知 handlers 中的某⼀个任务。通 知发出后handlers 会根据发送的通知，在handlers中相关的任务中寻 找名称为"reload nginx server" 的任务。