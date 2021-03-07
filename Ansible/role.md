# role

## role简介

role，它允许管理员将复杂的playbook拆分成多个小的逻辑单元，便于维护和管理。

ansible一个平台：https://galaxy.ansible.com/ 运维人员可以在上面分享和下载role

### 结构

示例：

```shell
webservers/
├── Defaults
│   └── main.yml
├── files
├── handlers
│   └── main.yml
├── meta
│   └── main.yml
├── tasks
│   └── main.yml
├── templates
├── vars
    └── main.yml
```

示例中，roles的名字叫webservers，使用时，每个目录必须包含一个main.yml文件，这个文件应该包含如下对应目录名称对应的内容：

- task -包含角色要执行的任务的主要列表
- handlers -包含处理程序
- defaults -角色的默认变量值
- vars -角色的其他变量
- files -包含可以通过此角色部署的文件
- templates -包含可以通过此角色部署的模板
- meta -为此角色定义一些元数据

## 制作一个role

playbook

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

分解上面这个playbook，role的名字为nginx

```shell
nginx/
├── files
├── handlers
│   └── main.yml
├── tasks
│   └── main.yml
├── templates
├── vars
    └── main.yml
```

files目录存放文件

```text
存放www.qfedu.com.conf文件
```

handlers目录的main.yml文件

```yaml
---
- name: reload nginx server
  service: name=nginx state=reloaded
  when:
    - nginxsyntax.rc == 0
    - nginxrunning.stat.exists == true
```

task目录中的main.yml文件

```yaml
---
把task任务全部复制过来
```

vars目录中的main.yml文件

```yaml
---
createuser:
  - tomcat
  - www
  - mysql
```

## 在playbook中使用role

role本身不能直接执行，需要借助playbook调用

```yaml
- name: a playbook used role
  hosts：webserver
  roles：
    - nginx
```

