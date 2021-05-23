# docker

## 基于docker的pipeline

准备工作：

```shell
docker run --name jenkins -itd \
-p 8081:8080 \
-p 50000:50000 \
-v ~/jenkins:/var/jenkins_home \
#挂载
-v /var/run/docker.sock:/var/run/docker.sock \
-v /usr/local/bin/docker:/usr/bin/docker \
jenkins/jenkins:lts
```

解决权限问题

```shell
#以root运行jenkins
docker exec -it -u root jenkins bash
#把jenkins用户加入root组
usermod -aG root jenkins

id jenkins
```

测试流水线：

```groovy
pipeline {
    agent {
        docker {
            image 'maven:3.6.3-jdk-8' #流水线运行在该镜像上
            args '-v $HOME/.m2:/root/.m2'
        }
    }
    stages {
        stage('Build'){ #步骤为build，执行命令mvn -v
            steps{
                sh 'mvn -v'
            }
        }
    }
}
```

## 基于docker配置前端流水线

编写jenkinsfile：

```groovy
pipeline {
    agent none
    stages {
        stage('WebBuild'){
            agent {
                docker {
                    image 'node:10.19.0-apline'
                    args '-u 0:0 -v /var/jenkins_home/.npm:/root/.npm' #使用root构建 -u:0:0;挂载缓存卷
                }
            }
            steps{
                sh """
                    id
                    ls /root/.npm
                    
                    #npm config set unsafe-perm=true
                    npm config list
                    npm config set cache /root/.npm
                    #npm config set registy https://registy.npm.taobao.org
                    npm config list
                    ls
                    cd demo && npm install --unsafe-perm=true && npm run build && ls -l dist/ && sleep 15
               """  
            }
        }
    }
}
```

