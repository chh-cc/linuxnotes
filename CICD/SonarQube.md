# SonarQube

## SonarQube简介

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