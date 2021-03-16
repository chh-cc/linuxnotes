# pipeline

## 声明式pipeline

### 核心概念

1.pipeline :声明其内容为一个声明式的pipeline脚本

2.agent:执行节点（job运行的slave或者master节点）

3.stages:阶段集合，包裹所有的阶段（例如：打包，部署等各个阶段）

4.stage:阶段，被stages包裹，一个stages可以有多个stage

5.steps:步骤,为每个阶段的最小执行单元,被stage包裹

6.post:执行构建后的操作，根据构建结果来执行对应的操作

#### 1.pipeline

> 作用域：应用于全局最外层，表明该脚本为声明式pipeline
> 是否必须：必须
> 参数：无

#### 2.agent

> 作用域：可用在全局与stage内
> 是否必须：是，
> 参数：any,none, label, node,docker,dockerfile

```yaml
//运行在任意的可用节点上
agent any
//全局不指定运行节点，由各自stage来决定
agent none
//运行在指定标签的机器上,具体标签名称由agent配置决定
agent { label 'master' }
//node参数可以扩展节点信息
agent { 
     node {
         label 'master'
         customWorkspace 'xxx'
    } 
}
//使用指定运行的容器
agent { docker 'python'  }
```

#### 3.stages

> 作用域：全局或者stage阶段内，每个作用域内只能使用一次
>
> 是否必须：全局必须
>
> 参数：无

#### 4.stage

> 作用域：被stages包裹，作用在自己的stage包裹范围内
>
> 是否必须：必须
>
> 参数：需要一个string参数，表示此阶段的工作内容
>
> 备注：stage内部可以嵌套stages，内部可单独制定运行的agent

#### 5.steps

作用域：被stage包裹，作用在stage内部
是否必须：必须
参数：无

```yaml
stages{
    stage("Pull Code"){
        steps{
            echo "开始拉取代码"
        }
    }
    stage("Build"){
        steps{
            echo "开始构建代码"
        }
    }
}
```

#### 6.post

作用域：作用在pipeline结束后者stage结束后
条件：always、changed、failure、success、unstable、aborted

### 指令

#### 1.environment：声明一个全局变量或者步骤内部的局部变量

```shell
environment {
        P1="parameters 1"
    }
```

#### 2.options:options指令能够提供给脚本更多的选项

- buildDiscarder:指定build history与console的保存数量
  用法：options { buildDiscarder(logRotator(numToKeepStr: ‘1’)) }
- disableConcurrentBuilds：设置job不能够同时运行
  用法：options { disableConcurrentBuilds() }
- skipDefaultCheckout：跳过默认设置的代码check out
  用法：options { skipDefaultCheckout() }
- skipStagesAfterUnstable:一旦构建状态变得UNSTABLE，跳过该阶段
  用法：options { skipStagesAfterUnstable() }
- checkoutToSubdirectory:在工作空间的子目录进行check out
  用法：options { checkoutToSubdirectory(‘children_path’) }
- timeout:设置jenkins运行的超时时间，超过超时时间，job会自动被终止
  用法：options { timeout(time: 1, unit: ‘MINUTES’) }
- retry :设置retry作用域范围的重试次数
  用法：options { retry(3) }
- timestamps:为控制台输出增加时间戳
  用法：options { timestamps() }

备注：当options作用在stage内部的时候，可选的只能是跟stage相关的选项（skipDefaultCheckout、timeout、retry、timestamps)

#### 3.parameters：提供pipeline运行的参数

- 作用域：被最外层pipeline所包裹，并且只能出现一次，参数可被全局使用
- 好处：使用parameters好处是能够使参数也变成code,达到pipeline as code，pipeline中设置的参数会自动在job构建的时候生成，形成参数化构建

```shel
parameters {
        string(name: 'P1', defaultValue: 'it is p1', description: 'it is p1')
        booleanParam(name: 'P2', defaultValue: true, description: 'it is p2')
    }
```

#### 4.when：根据when指令的判断结果来决定是否执行后面的阶段

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

#### 5. 脚本

> 在声明式的pipeline中默认无法使用脚本语法，但是pipeline提供了一个脚本环境入口：script{},通过使用script来包裹脚本语句，即可使用脚本语法



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

