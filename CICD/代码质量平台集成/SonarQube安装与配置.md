# SonarQube安装与配置

## SonarQube简介

组成：

- sonarqube server ：  他有三个程序分别是 webserver（配置和管理sonar） searchserver（搜索结果返回给sonarUI）  ComplateEngineserver（计算服务 将分析结果入库）。
- sonarqube db : 数据库 存放配置。
- sonarqube plugins： 插件增加功能。
- sonar-scanner ： 代码扫描工具 可以有多个。

7.9版本开始不再支持mysql，mysql-->PG



工作流程：

开发人员在IDE中编写代码，并使用SONARLILT来运行本地分析。

开发人员将他们的代码推到他们最喜欢的SCM：Git，Svn，TFVC，…

连续集成服务器触发自动构建，执行SONARQUE扫描器需要运行SONARQUE分析。

分析报告被发送到SONARQUE服务器进行处理。

SONARQUE服务器在SONARQUE数据库中处理和存储分析报告结果，并将结果显示在UI中。

开发人员审查、评论、挑战他们的问题，通过SONARQUE UI管理和减少他们的技术债务。

管理者从分析中得到报告。

OPS使用API来自动配置并从SONARQUE中提取数据。

OPS使用JMX监控SONARQUBE服务器。

## SonarQube安装

条件：安装java（jre 11或jdk 11）

设置参数：

```shell
sysctl -w vm.max_map_count=262144
sysctl -w fs.file-max=65536
ulimit -n 65536
ulimit -u 4096
```

用docker安装：

```shell
LOCALDIR=/data/devops
docker run --rm -d --name sonarqube \
-p 9000:9000
-v ${LOCALDIR}/sonar/sonarqube_conf:/opt/sonarqube/conf \
-v ${LOCALDIR}/sonar/sonarqube_extensions:/opt/sonarqube/extensions \
-v ${LOCALDIR}/sonar/sonarqube_logs:/opt/sonarqube/logs \
-v ${LOCALDIR}/sonar/sonarqube_data:/opt/sonarqube/data \
sonarqube:7.9.2-community
```

docker安装会提示：

![image-20210517233720377](https://gitee.com/c_honghui/picture/raw/master/img/20210517233727.png)

## SonarQube配置

安装中文插件：

<img src="https://gitee.com/c_honghui/picture/raw/master/img/20210518230519.png" alt="image-20210518230512360" style="zoom:67%;" />

强制认证：

![image-20210518230605213](https://gitee.com/c_honghui/picture/raw/master/img/20210518230605.png)

LDAP集成：

<img src="https://gitee.com/c_honghui/picture/raw/master/img/20210518230633.png" alt="image-20210518230633465" style="zoom:67%;" />

```shell
#LDAP settings
#admin
sonar.security.realm=LDAP
ldap.url=ldap://ldao.com.com:389
ldap.bindDn=uid=jenkins,cn=xxxx,DC=xxxxx
ldap.bindPassword=xxxxxxx
#users

ldap.user.baseDn=cn=users, DC=xxxxxx
ldap.user.request=(&(objectClass=inetOrgPerson)(uid={login}))
ldap.user.realNameAttribute=cn
ldap.user.emailAttribute=mai

systemctl restart sonarqube
```



gitlabsso集成：

![image-20210518231521683](https://gitee.com/c_honghui/picture/raw/master/img/20210518231521.png)

<img src="https://gitee.com/c_honghui/picture/raw/master/img/20210518231442.png" alt="image-20210518231442110" style="zoom:67%;" />