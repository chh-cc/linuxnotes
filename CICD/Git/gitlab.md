## gitlab安装

rpm安装：

 yum install https://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el7/gitlab-ce-13.2.3- ce.0.el7.x86_64.rpm



docker安装：

```shell
mkdir /opt/gitlab 
GITLAB_HOME=/opt/gitlab # 数据持久化目录 
docker run --detach \ 
--hostname gitlab.ctnrs.com \ 
--publish 443:443 --publish 80:80 
--publish 2222:22 \ 
--name gitlab \ 
--restart always \ 
--volume $GITLAB_HOME/config:/etc/gitlab \ 
--volume $GITLAB_HOME/logs:/var/log/gitlab \ 
--volume $GITLAB_HOME/data:/var/opt/gitlab \ 
gitlab/gitlab-ce:latest
```



gitlab涉及组件

```shell
# gitlab-ctl service-list
alertmanager* 
gitaly* 
gitlab-exporter*
gitlab-workhorse*
grafana*
logrotate*
nginx*
postgres-exporter*
postgresql*
prometheus*
puma*
redis*
redis-exporter*
sidekiq*
sshd*
```

## 基本配置

配置SMTP（发邮件）：

```shell
# vim /etc/gitlab/gitlab.rb
gitlab_rails['time_zone'] = 'Asia/Shanghai' # 时区
gitlab_rails['gitlab_email_from'] = 'baojingtongzhi@163.com' 
gitlab_rails['smtp_enable'] = true
gitlab_rails['smtp_address'] = "smtp.163.com"
gitlab_rails['smtp_port'] = 25
gitlab_rails['smtp_user_name'] = "baojingtongzhi@163.com"
gitlab_rails['smtp_password'] = "TKOLBLXRONCPHANJ" # 授权码或者密码
gitlab_rails['smtp_domain'] = "163.com"
gitlab_rails['smtp_authentication'] = "login"
gitlab_rails['smtp_enable_starttls_auto'] = true
gitlab_rails['smtp_tls'] = false

重新加载配置：
gitlab-ctl reconfigure
如果使用docker部署，直接docker restart gitlab即可。
测试是否配置成功：
gitlab-rails console
irb(main):001:0> Notify.test_email('邮箱地址', '标题', '内容').deliver_now
irb(main):002:0> exit
```

配置HTTPS：

```shell
# vim /etc/gitlab/gitlab.rb
external_url 'https://gitlab.ctnrs.com' # 访问使用的域名或者IP
nginx['enable'] = true
nginx['redirect_http_to_https'] = true #设置开启自动将HTTP跳转到HTTPS
nginx['ssl_certificate'] = "/etc/gitlab/ssl/gitlab.ctnrs.com.pem"
nginx['ssl_certificate_key'] = "/etc/gitlab/ssl/gitlab.ctnrs.com-key.pem“
```

## 常用管理命令

gitlab-ctl Gitlab管理工具

gitlab-rails

gitlab-redis-cli 访问Redis数据库

gitlab-psql 访问PGSQL数据库

gitlab-rake 备份与恢复

gitlab-backup 12.1版本以后增加的备份与恢复工具



gitlab-ctl常用指令

start 启动所有服务 

restart 重启所有服务 

status 查看所有服务状态 

tail 查看日志 

service-list 列出所有启动的服务 

graceful-kill 平滑停止一个服务 

reconfigure 重新加载配置 

show-config 查看所有服务的配置文件 

uninstall 卸载这个软件 

cleanse 删除gitlab数据，重新白手起家

## gitlab使用

使用流程：

1. 创建gitlab用户

2. 创建群组

   群组有三种访问权限： 

   • Private：只有组成员才能访问，企业内部一般都用这个。

   • Internal：只要登录的用户就能看到。一般用于对IT部门公开的项目 

   • Public：不用登录也能看到。一般用于开源项目

3. 在群组创建项目

3. 群组邀请成员
   Gitlab用户在群组中有五种权限：

   Guest：可以创建issue、发表评论，不能读写版本库 

   Reporter：可以克隆代码，不能提交，QA、PM可以赋予这个权限 

   Developer：可以**克隆代码、开发、提交**、push，RD开发者可以赋予 这个权限 

   Maintainer：可以**创建项目、添加tag、保护分支、添加项目成员、编 辑项目**，核心RD负责人可以赋予这个权限 

   Owner：可以**设置项目访问权限 - 删除项目、迁移项目、管理组成员**， 开发组leader可以赋予这个权限

