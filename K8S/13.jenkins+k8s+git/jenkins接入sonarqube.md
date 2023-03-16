## 安装sonarqube

[SonarScanner | SonarQube Docs](https://docs.sonarqube.org/latest/analysis/scan/sonarscanner/)

官网上已经声明 sonarQube 7.9 版本以上不再支持 mysql 了，我们为了以后升级新版本不做数据库迁移，**尽量使用 postgresql** 



安装postgresql数据库:

```shell
docker pull postgres:11

mkdir -p /home/apps/postgres/{postgresql,data}

docker run -d --name postgres -p 5432:5432 \
-v /home/apps/postgres/postgresql:/var/lib/postgresql \
-v /home/apps/postgres/data:/var/lib/postgresql/data \
-v /etc/localtime:/etc/localtime:ro \
-e POSTGRES_USER=sonar \
-e POSTGRES_PASSWORD=sonar \
-e POSTGRES_DB=sonar \
-e TZ=Asia/Shanghai \
--restart always \
--privileged=true \
postgres:11
```

#数据库操作：

```shell
# 进入docker容器
docker exec -it postgres /bin/bash
 
# 用户登录(sonar)
psql -U sonar
 
# 创建新用户
create user admin with password '123456';
 
# 创建数据库，指定用户
create database testDB with owner admin;
 
# 退出
\q
 
# 查看用户
\du
 
# 列出数据库
\l
 
# 删除用户
drop user admin;
 
# 删除数据库
drop database dbtest;
```

安装sonarqube：

```shell
docker pull sonarqube:8.9.2-community

mkdir -p /home/apps/sonarqube/{extensions,logs,data}

---
vim /etc/sysctl.conf
 
# 增加以下配置
vm.max_map_count=262144
fs.file-max=65536
 
# 使配置生效
sysctl -p
---

docker run -d --name sonarqube -p 9000:9000 \
--link postgres \
-v /home/apps/sonarqube/extensions:/opt/sonarqube/extensions \
-v /home/apps/sonarqube/logs:/opt/sonarqube/logs \
-v /home/apps/sonarqube/data:/opt/sonarqube/data \
-e SONARQUBE_JDBC_URL=jdbc:postgresql://postgres:5432/sonar \
-e SONARQUBE_JDBC_USERNAME=sonar \
-e SONARQUBE_JDBC_PASSWORD=sonar \
--restart always \
--privileged=true \
sonarqube:8.9.2-community
```

## jenkins接入sonarqube

1.**jenkins安装sonarqube插件**

系统管理-->插件管理-->可选插件：搜索sonar，找到Sonarqube Scanner

安装之后重启jenkins即可



访问sonarqube，http:ip:9000，账号密码admin/admin



2.**创建token**

Administration-->Security-->Users

![image-20230316144514272](assets/image-20230316144514272.png)

<img src="assets/image-20230316144646821.png" alt="image-20230316144646821" style="zoom:50%;" />

3.**在jenkins配置sonarqube**

![image-20220619231004557](../../CICD/Jenkins/assets/image-20220619231004557.png)

<img src="../../CICD/Jenkins/assets/image-20220619231032539.png" alt="image-20220619231032539" style="zoom:67%;" />



## sonarqube的使用

**sonarqube安装中文插件：**

Administration->Marketplace->搜索chinese，安装完会提示restart server需要重启sonarqube

![img](assets/6dd84ab5b9f03d5224a7c619b6fdb87d.png)



创建项目

<img src="assets/image-20230316151103136.png" alt="image-20230316151103136" style="zoom:50%;" />

设置token

<img src="assets/image-20230316151212072.png" alt="image-20230316151212072" style="zoom:50%;" />

使用对应项目（maven or gradle？）的命令可以将代码上传到sonarqueb

<img src="assets/image-20230316151503584.png" alt="image-20230316151503584" style="zoom:50%;" />

上传后如图

![image-20230316171814510](assets/image-20230316171814510.png)

