# 发布maven项目

1. 安装maven integration插件

2. 配置git账号密码

   ![image-20210421112500457](https://gitee.com/c_honghui/picture/raw/master/img/20210421112506.png)

3. 把jar包打包到maven仓库

![image-20210421112827581](https://gitee.com/c_honghui/picture/raw/master/img/20210421112827.png)

4. 配置post steps

```shell
#!/bin/bash
#服务名称
SERVER_NAME=mayikt_springboot
#源jar路径，mvn打包后，target目录下的jar包名称
JAR_NAME=mayikt_springboot-0.0.1-SNAPSHOT
#源jar路径
#/usr/local/jenkins_home/workspace---->jenkins工作目录
#demo项目目录
#target打包生成jar包的目录
JAR_PATH=/var/jenkins_home/workspace/mayikt_springboot/target
#打包完后把jar包移动到运行jar包的目录---->work_daemon,该目录需要自己创建
JAR_WORK_PATH=/var/jenkins_home/workspace/mayikt_springboot/target

echo "查询进程id---->$SERVER_NAME"
PID=`ps -ef|grep $SERVER_NAME|awk '{print $2}'`
echo "得到进程id：$PID"
echo "结束进程"
for id in $PID
do
    kill -9 $id
    echo "killed $id"
done
echo "结束进程完成"

#复制jar包到执行目录
cp ${JAR_PATH}/${JAR_NAME}.jar $JAR_WORK_PATH
echo "复制jar包完成"
cd $JAR_WORK_PATH
#修改文件权限
chmod 755 $JAR_NAME.jar

BUILD_ID=dontKillMe nohup java -jar $JAR_NAME.jar &
```

