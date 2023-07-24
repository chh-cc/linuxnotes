## 备份

### 修改数据存储位置

1. 修改配置文件

   ```shell
   mkdir /data/gitlab-data
   
   vim /etc/gitlab/gitlab.rb
   git_data_dirs({
     “default” => {
       “path” => “/data/gitlab-data”
      }
   })
   ```

2. 将旧数据拷贝到新的数据目录

   ```shell
   rsync -av /var/opt/gitlab/git-data/repositories /data/gitlab-data/
   ```

3. 停止gitlab服务，加载配置

   ```shell
   gitlab-ctl stop
   gitlab-ctl reconfigure
   ```

4. 加载完成后启动gitlab服务会看到/data/gitlab-data/下多了一个 repositories目录

   ```shell
   gitlab-ctl start
   ```

### 修改备份存储位置

1. 修改配置文件

   ```shell
   mkdir /data/gitlab-backup -p
   
   
   vim /etc/gitlab/gitlab.rb
   gitlab_rails['backup_path'] = "/data/gitlab-backups"
   ```

2. 停止gitlab服务，加载配置

   ```shell
   gitlab-ctl stop
   gitlab-ctl reconfigure
   ```

   

### 每日备份

执行备份命令需要sudo权限

12.1版本之后：gitlab-backup create

12.1版本之前：gitlab-rake gitlab:backup:create



`Warning: Your gitlab.rb and gitlab-secrets.json files contain sensitive data 
and are not included in this backup. You will need these files to restore a backup.
Please back them up manually.`表示gitlab.rb 和 gitlab-secrets.json 两个文件包含敏感信息。未被备份到备份文件中。需要手动备份。

### 备份保存30天

修改配置文件

```shell
vim /etc/gitlab/gitlab.rb

gitlab_rails['manage_backup_path'] = true
gitlab_rails['backup_path'] = "/data/gitlab-backups"
 
gitlab_rails['backup_archive_permissions'] = 0644   //生成的备份文件权限
 
gitlab_rails['backup_keep_time'] = 2592000      //默认备份保留天数为7天（604800秒,这里是30天）
```

### 自动备份

结合crontab实施自动定时备份

```shell
vim gitlab-bak.sh

#!/bin/bash
gitlab-backup create
cd /data/gitlab-backups

mkdir gitlab_$(date +%Y%m%d)

find . -name '*.tar' -ctime -1 -exec cp  {}  /data/gitlab-backups/gitlab_$(date +%Y%m%d)/ \;

cp /etc/gitlab/gitlab.rb /data/gitlab-backups/gitlab_$(date +%Y%m%d)
cp /var/opt/gitlab/nginx/conf /data/gitlab-backups/gitlab_$(date +%Y%m%d)
cp /etc/postfix/main.cfpostfix /data/gitlab-backups/gitlab_$(date +%Y%m%d)
cp /etc/gitlab/gitlab-secrets.json /data/gitlab-backups/gitlab_$(date +%Y%m%d)
zip -rP password  gitlab_$(date +%Y%m%d).zip gitlab_$(date +%Y%m%d)/
 
cp  gitlab_$(date +%Y%m%d).zip xxx
```

### 备份文件的忽略

全量备份时考虑到有些内容不需要备份，否则备份文件过大，拷贝起来比较缓慢，筛选信息，首先看一下备份文件说明：

```shell
db        //数据库备份：主要为PostgreSQL数据库数据内容

uploads                //附件数据备份

repositories                //Git仓库数据备份

builds                //CI 作业输入日志等数据备份

artifacts                //CI 作业工件数据备份（artifacts用于指定在job 成功或失败 时应附加                                  到作业的文件和目录的列表。）

lfs                //LFS对象数据备份

registry                //容器镜像备份

pages                //GitLab Pages content，页面内容数据备份
```

使用SKIP选项（指定的内容不作为备份的对象目录的意思），以忽略artifacts为例，

```shell
gitlab-backup create SKIP=artifacts
```



## 恢复

将备份文件拷贝到待恢复测试的服务器上，注意：GitLab恢复数据的时候需要满足版本一致，即进行恢复的GitLab的版本和备份数据时的GitLab的版本一致。

停止与连接 数据库有关的进程

```vbscript
sudo gitlab-ctl stop unicorn
sudo gitlab-ctl stop sidekiq
```

恢复

```shell
gitlab-rake gitlab:backup:restore BACKUP=/data/gitlab-backups/1665366086_2022_10_10_15.4.0_gitlab_backup
```

