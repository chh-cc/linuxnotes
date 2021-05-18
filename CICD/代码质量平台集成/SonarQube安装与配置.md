# SonarQube安装与配置

## SonarQube简介

7.9版本开始不再支持mysql，mysql-->PG

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

gitlabsso集成：

![image-20210518231521683](https://gitee.com/c_honghui/picture/raw/master/img/20210518231521.png)

<img src="https://gitee.com/c_honghui/picture/raw/master/img/20210518231442.png" alt="image-20210518231442110" style="zoom:67%;" />