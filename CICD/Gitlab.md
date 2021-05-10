# Gitlab

## 安装

服务器要求:建议2c4g起步

rpm安装：

```shell
yum install https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el7/gitlab-ce-13.2.3-
ce.0.el7.x86_64.rpm  
```

docker安装：

```shell
mkdir /opt/gitlab
GITLAB_HOME=/opt/gitlab # 数据持久化目录
docker run --detach \
--hostname gitlab.ctnrs.com \
--publish 443:443 --publish 80:80 --publish 2222:22 \
--name gitlab \
--restart always \
--volume $GITLAB_HOME/config:/etc/gitlab \
--volume $GITLAB_HOME/logs:/var/log/gitlab \
--volume $GITLAB_HOME/data:/var/opt/gitlab \
gitlab/gitlab-ce:latest  
```

gitlab-ce docker-compose 完整配置

```shell
version: '2'
services:
  gitlab:
    image: 'gitlab/gitlab-ce:12.4.1-ce.0'
    restart: always
    container_name: gitlab
    hostname: 'gitlab'
    environment:
      GITLAB_OMNIBUS_CONFIG: |
        external_url 'https://code.example.com'
        nginx['enable'] = true
        nginx['redirect_http_to_https'] = true
        nginx['redirect_http_to_https_port'] = 80
        # 开启 pages 功能
        pages_external_url 'http://pages.example.com'
        gitlab_pages['inplace_chroot'] = true
        gitlab_rails['lfs_enabled'] = true
        # 设置时区为北京时间
        gitlab_rails['time_zone'] = 'PRC'
        gitlab_rails['gitlab_email_enabled'] = true
        gitlab_rails['gitlab_email_from'] = 'code@example.com'
        gitlab_rails['gitlab_email_display_name'] = 'code'
        gitlab_rails['gitlab_email_reply_to'] = 'code@example.com'
        gitlab_rails['smtp_enable'] = true
        gitlab_rails['smtp_address'] = 'smtp.exmail.qq.com'
        gitlab_rails['smtp_port'] = 465
        gitlab_rails['smtp_user_name'] = 'code@example.com'
        gitlab_rails['smtp_password'] = '******'
        gitlab_rails['smtp_domain'] = 'exmail.qq.com'
        gitlab_rails['smtp_authentication'] = 'login'
        gitlab_rails['smtp_enable_starttls_auto'] = true
        gitlab_rails['smtp_tls'] = true
        unicorn['worker_processes'] = 2
        unicorn['worker_timeout'] = 60
        sidekiq['concurrency'] = 4
        # 解决 GitLab 响应 Forbidden
        gitlab_rails['rack_attack_git_basic_auth'] = {'enabled' => true, 'ip_whitelist' => ["0.0.0.0"], 'maxretry' => 300, 'findtime' => 5, 'bantime' => 60}
    # 内存和CPU限制，worker_processes 配置声明使用2核CPU
    mem_limit: 5500m
    cpu_shares: 200 #2核
    ports:
      - '443:443'
      - '80:80'
      - '22:22'
    volumes:
      # 挂载宿主机目录可以根据实际情况挂载
      - '/mnt/gitlab-docker/config:/etc/gitlab'
      - '/mnt/gitlab-docker/logs:/var/log/gitlab'
      - '/mnt/gitlab-docker/data:/var/opt/gitlab'
      - '/etc/localtime:/etc/localtime'
```

## 配置

开启https

```shell
external_url 'https://gitlab.example.com'
nginx['enable'] = true
nginx['redirect_http_to_https'] = true    # http重定向到https
nginx['redirect_http_to_https_port'] = 80
```

申请Let's Encrypt证书并手动添加证书

```shell
# /mnt/gitlab-docker/config 是挂载宿主机目录

$ mkdir -p /mnt/gitlab-docker/config/ssl
$ chmod 700 /mnt/gitlab-docker/config/ssl
$ cp gitlab.example.com.key gitlab.example.com.crt /mnt/gitlab-docker/config/ssl
```

重新加载配置：
gitlab-ctl reconfigure
如果使用docker部署，直接docker restart gitlab即可。  

## 权限

### 概念

```text
1.项目由项目组来创建，而不是由用户创建
2.用户通过加入到不同的组，来实现对项目的访问或者提交
3.项目可以设置为只有项目组可以查看，所有登陆用户可以查看和谁都可以看三种
```

Gitlab用户在组中有五种权限：Guest、Reporter、Developer、Master、Owner

- Guest：可以创建issue、发表评论，不能读写版本库
- Reporter：可以克隆代码，不能提交，QA、PM可以赋予这个权限
- Developer：可以克隆代码、开发、提交、push，RD可以赋予这个权限
- Master：可以创建项目、添加tag、保护分支、添加项目成员、编辑项目，核心RD负责人可以赋予这个权限
- Owner：可以设置项目访问权限 - Visibility Level、删除项目、迁移项目、管理组成员，开发组leader可以赋予这个权限

Gitlab中的组和项目有三种访问权限：Private、Internal、Public

- Private：只有组成员才能看到
- Internal：只要登录的用户就能看到
- Public：所有人都能看到

开源项目和组设置的是Internal

### 操作流程

```text
1.创建组
2.基于组创建项目
3.创建用户，分配组，分配权限
```

![image-20210312105318823](https://gitee.com/c_honghui/picture/raw/master/img/20210312105318.png)

### 实操

#### 创建组

点击new group

![img](https://gitee.com/c_honghui/picture/raw/master/img/20210312105621.png)

填组名:

<img src="https://gitee.com/c_honghui/picture/raw/master/img/20210312105643.png" alt="img" style="zoom: 67%;" />

群组三种权限：

- private：只有组成员才能访问，企业内部一般用这个
- internal：只要登陆的用户就能看到
- public：不用登陆也能看到

检查:

![img](https://gitee.com/c_honghui/picture/raw/master/img/20210312105829.png)

#### 创建项目

点击new project

<img src="https://gitee.com/c_honghui/picture/raw/master/img/20210312110054.png" alt="img" style="zoom:67%;" />

选择dev组，创建game项目

<img src="https://gitee.com/c_honghui/picture/raw/master/img/20210312110119.png" alt="img" style="zoom:50%;" />

#### 创建用户

点击创建用户

<img src="https://gitee.com/c_honghui/picture/raw/master/img/20210312110343.png" alt="img" style="zoom:67%;" />

创建cto用户

![img](https://gitee.com/c_honghui/picture/raw/master/img/20210312110425.png)

<img src="https://gitee.com/c_honghui/picture/raw/master/img/20210312110438.png" alt="img" style="zoom: 50%;" />

创建密码

![img](https://gitee.com/c_honghui/picture/raw/master/img/20210312110829.png)

<img src="https://gitee.com/c_honghui/picture/raw/master/img/20210312110859.png" alt="img" style="zoom: 50%;" />

#### 授权

dev组添加用户:

![img](https://gitee.com/c_honghui/picture/raw/master/img/20210312111838.png)

添加cto用户:

<img src="https://gitee.com/c_honghui/picture/raw/master/img/20210312112054.png" alt="img" style="zoom: 50%;" />

用户在群组中有五种权限：

- guest：可创建issue、发表评论、不能读写版本库
- reporter：可克隆代码、不能提交，QA、PM可赋予该权限
- developer：可克隆代码、开发、提交、push，开发可赋予该权限
- maintainer：可创建项目、添加tag、保护分支、添加项目成员、编辑项目，核心负责人可赋予该权限
- owner：可设置项目访问权限，开发组leader可赋予该权限

检查

![image-20210312112139706](https://gitee.com/c_honghui/picture/raw/master/img/20210312112139.png)

取消用户注册

![img](https://gitee.com/c_honghui/picture/raw/master/img/20210312112216.png)

## 备份和恢复

• 手动备份
备份数据： gitlab-rake gitlab:backup:create
备份配置文件： gitlab-ctl backup-etc
• 自动备份
\# crontab -l
\* * * * * docker exec -it gitlab gitlab-rake gitlab:backup:create
\* * * * * docker exec -it gitlab gitlab-ctl backup-etc
策略建议：本地保留7天，异地备份永久保存  

• 自动清理
\# vim /etc/gitlab/gitlab.rb
gitlab_rails['manage_backup_path'] = true
gitlab_rails['backup_path'] = "/var/opt/gitlab/backups"
gitlab_rails['backup_keep_time'] = 604800  

第1步：准备新环境
docker rm -f gitlab
mv /opt/gitlab /opt/gitlab.bak
第2步：拷贝备份文件
\# 宿主机
cd /opt cp
gitlab.bak/data/backups/1602317042_2020_10_10_13.3.5_gitlab_backup.tar
gitlab/data/backups/
第3步：停写库服务
\# 容器
chown git.git /etc/gitlab/config_backup
gitlab-ctl stop unicorn
gitlab-ctl stop puma
gitlab-ctl stop sidekiq
第4步：恢复数据
gitlab-rake gitlab:backup:restore BACKUP=1602317042_2020_10_10_13.3.5
第5步：恢复配置文件
\# 宿主机
cd /opt
cp gitlab.bak/config/config_backup gitlab/config/ -rf
\# 容器
cd /etc/gitlab/config_backup/
tar xvf gitlab_config_1602317163_2020_10_10.tar
cp -rf etc/gitlab/* /etc/gitlab/
第6步：访问验证  