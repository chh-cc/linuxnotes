# pipeline

## 声明式pipeline

### 基础结构

1.pipeline :声明其内容为一个声明式的pipeline脚本

2.agent:指定流水线的执行节点（job运行的slave或者master节点）

3.stage:阶段，每个阶段都必须要有名称

4.stages:流水线中多个stage的容器

5.steps:阶段中的一个或多个具体步骤（step）的容器

6.post（可选）:执行构建后的操作，根据构建结果来执行对应的操作，post分成多种条件块

- always：无论当前完成状态时什么，都执行
- changed：当前完成状态于上一次不同就执行
- fixed：上一次完成状态为失败或不稳定，当前状态为成功时执行
- regression：上一次完成状态为成功，当前完成状态为失败、不稳定或中止时执行
- aborted：当前执行结果是中止状态时执行
- failure：当前完成状态为失败时执行
- success：当前完成状态为成功时执行
- unstable：当前完成状态为不稳定时执行
- cleanup：清理条件块

```yaml
pipeline {
    agent any
    stages {
        stage('build') {
            steps {
                echo "hello world"
            }
        }
    }
}
```



### 指令

基础结构满足不了多变的需求，所以还通过指令来丰富。

#### 1.environment

设置环境变量，可定义在pipeline或stage部分

```shell
environment {
        P1="parameters 1"
    }
```

#### 2.tools

可定义在pipeline后stage部分，自动下载并安装指定的工具，并将其加入PATH变量中

#### 3.input

定义在stage部分，会暂停pipeline，提示你输入内容

#### 4.options

options指令能够提供给脚本更多的选项

- buildDiscarder:保存最近历史构建记录的数量
  用法：options { buildDiscarder(logRotator(numToKeepStr: ‘10’)) }
- disableConcurrentBuilds：设置job不能够同时运行
  用法：options { disableConcurrentBuilds() }
- skipDefaultCheckout：跳过默认设置的代码check out
  用法：options { skipDefaultCheckout() }
- skipStagesAfterUnstable:一旦构建状态变得UNSTABLE，跳过该阶段
  用法：options { skipStagesAfterUnstable() }
- checkoutToSubdirectory:在工作空间的子目录进行check out
  用法：options { checkoutToSubdirectory(‘children_path’) }
- timeout:设置jenkins执行的超时时间，超过超时时间，job会自动被终止
  用法：options { timeout(time: 10, unit: ‘MINUTES’) }
- retry :当发生失败时进行重试
  用法：options { retry(3) }
- timestamps:为控制台输出增加时间戳
  用法：options { timestamps() }

备注：当options作用在stage内部的时候，可选的只能是跟stage相关的选项（skipDefaultCheckout、timeout、retry、timestamps)

#### 3.parameters

执行pipeline前传入的一些参数

- 作用域：被最外层pipeline所包裹，并且只能出现一次，参数可被全局使用
- 好处：使用parameters好处是能够使参数也变成code,达到pipeline as code，pipeline中设置的参数会自动在job构建的时候生成，形成参数化构建

```shel
parameters {
        string(name: 'P1', defaultValue: 'it is p1', description: 'it is p1')
        booleanParam(name: 'P2', defaultValue: true, description: 'it is p2')
    }
```

#### 4.when

满足when定义的条件时，阶段才执行

可选条件

- branch ：判断分支名称是否符合预期
  用法：when { branch ‘master’ }
- environment ： 判断环境变量是否符合预期
  用法：when { environment name: ‘DEPLOY_TO’, value: ‘production’ }
- expression：判断表达式是否符合预期
  用法：when { expression { return params.DEBUG_BUILD } }
- not : 判断条件是否为假
  用法：when { not { branch ‘master’ } }
- allOf：判断所有条件是不是都为真
  用法：when { allOf { branch ‘master’; environment name: ‘DEPLOY_TO’, value: ‘production’ } }
- anyOf：判断是否有一个条件为真
  用法：when { anyOf { branch ‘master’; branch ‘staging’ } }

### 脚本

在声明式pipeline中不能直接在steps块中写Groovy代码

可以在script步骤中写Groovy代码



## 脚本式pipeline

对语法的要求比较宽松，顶层可以是node，也可以是stage。node可以嵌套stage，stage反过来也可以嵌套node。典型的脚本式Pipeline语法如下：

```shell
node {
    dir('/home/share/node/falcon') {
        stage("git") {
            sh "git fetch origin"
            sh "git checkout -f origin/master"
        }
}
```

dir(‘路径’)
更换执行目录，jenkins默认的执行目录在环境设置中设置，默认是/

stage(“git”)
方法是阶段的名称，这个是完全自定义的，相当于给构建流程中的某些步骤称为一个阶段，比如git操作阶段、安装依赖阶段、编译阶段、发布阶段

sh
后接的就是命令行操作了，如果只有一行，那么用’’或者用””包裹起来，如果有多行的话，用’’’包裹

### 判断

```shell
node {
    dir('/home/share/www') {
        stage('Git') {
            if(fileExists('openapi')) {
                dir('/home/share/www/openapi') {
                    sh 'git fetch origin'
                    sh 'git checkout master'
                    sh 'git pull'
                }
            } else {
                sh 'git clone git@git.coding.net:flashtd1/DPOpenAPI.git openapi'
            }
        }
    }
}

#if(fileExists(‘文件名’))
#判断文件是否存在
```

### 异常处理

当任何一个步骤因各种原因而出现异常时，都必须在Groovy中使用try/catch/finally语句块进行处理，举例如下：

```shell
Jenkinsfile (Scripted Pipeline)
node {
    stage('Example') {
        try {
            sh 'exit 1'
        }
        catch (exc) {
            echo 'Something failed, I should sound the klaxons!'
            throw
        }
    }
}
```

### Steps

pipeline最核心和基本的部分就是“step”，从根本上来说，steps作为Declarative pipeline和Scripted pipeline语法的最基本的语句构建块来告诉jenkins应该执行什么操作。

可参考jenkins官网对该部分的介绍Pipeline Steps reference

### node

对应于Declarative Pipeline的agent，用于指定构建步骤应该在哪个构建服务器执行。

```shell
node('master'){
    stage('Build'){
        echo 'Building...'
    }
}
```

### withEnv

withEnv方法和environment语句块对应，用于定义环境变量。

## pipeline配置java项目

```shell
pipeline {
    agent { label 'slave' }
    options {
        timestamps()
        disableConcurrentBuilds()
        buildDiscarder(
            logRotator(
                numToKeepStr: '20',
                daysToKeepStr: '30',
            )
        )
    }
    parameters {
        choice(
           name: "DEPLOY_FLAG",
           choices: ['deploy', 'rollback'],
           description: "发布/回滚"
        )
    }
    /*=======================================常修改变量-start=======================================*/
    environment {
        gitUrl = "git地址"
        branchName = "分支名称"
        gitlabCredentialsId = "认证凭证"
        projectRunDir = "项目运行目录"
        jobName = "${env.JOB_NAME}"
        serviceName = "服务名称"
        serviceType = "jar"
        runHosts = "192.168.167.xx,192.168.167.xx"
        rollbackVersion = ""
    }
    stages {
        stage('Deploy'){	#发布阶段
            when {
                expression { return params.DEPLOY_FLAG == 'deploy' }	#当表达式为发布时
            }
            stages {
                stage('Pre Env') {
                    steps {
                        echo "======================================项目名称 = ${env.JOB_NAME}"
                        echo "======================================项目 URL = ${gitUrl}"
                        echo "======================================项目分支 = ${branchName}"
                        echo "======================================当前编译版本号 = ${env.BUILD_NUMBER}"
                    }
                }
                stage('Git Clone') {
                    steps {
                        git branch: "${branchName}",
                        credentialsId: "${gitlabCredentialsId}",
                        url: "${gitUrl}"
                    }
                }
                stage('Mvn Build') {
                    steps {
                        withMaven(jdk: 'jdk1.8', maven: 'maven') {
                            sh "mvn clean package -Dmaven.test.skip=true -U -f ${serviceName}/pom.xml"
                        }
                    }
                }
                stage('Ansible Deploy') {
                    steps{
                        script {
                            sleep 5
                            ansiColor('xterm') {
                                ansiblePlaybook colorized: true, extras: '-e "directory=${projectRunDir}" -e "job=${jobName}" -e "service=${serviceName}" -e "type=${serviceType}"', installation: 'ansible', inventory: '/etc/ansible/hosts.yml', limit: "${runHosts}", playbook: '/etc/ansible/playbook/deploy-jenkins.yml'                            
                            }
                        }
                    }
                }
            }   
        }
        stage('Rollback') {	#回滚
            when {
                expression { return params.DEPLOY_FLAG == 'rollback' }	#当表达式为回滚
            }
            steps{
                script {
                    rollbackVersion = input(
                        message: "请填写要回滚的版本",
                        parameters: [
                            string(name:'last_number')
                        ]
                    )
                    withEnv(["rollbackVersion=${rollbackVersion}"]){
                        sh '''
                            echo "正在回滚至就近第${rollbackVersion}个版本"
                            ansible ${runHosts} -m shell -a "sh ${projectRunDir}/rollback.sh ${rollbackVersion} ${serviceName}"
                        '''
                    }
                }
            }
        }
    }
    post {
        always {
            deleteDir()
        }
        success {
            echo 'This task is successful!'
        }
    }
}
```

