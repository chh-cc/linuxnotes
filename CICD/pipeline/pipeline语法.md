# pipeline语法

一条流水线通过jenkinsfile描述

安装声明式插件Pipeline:Declarative

jenkinsfile组成:

- 指定node节点/workspace
- 指定运行选项
- 指定stages阶段
- 指定构建后的操作

Jenkinsfile使用两种语法进行编写，分别是声明式和脚本式。

## Declarative Pipeline（声明式pipeline）

Declarative Pipeline遵循与Groovy相同的语法规则，但有以下几点例外：

- Pipeline的顶层必须是块，具体来说就是：pipeline { }。
- 不用分号作为语句分隔符，每个声明必须独立一行。
- 块里只能包含Sections（章节）、Directives（指令）、 Steps（步骤）或赋值语句。
- 属性引用以无参方法的方式调用。例如，输入被视为input（）。

### Sections（章节）

Declarative Pipeline里的Sections通常包含一个或多个Directives或Steps。

#### agent

agent指定**整个Pipeline或特定stage在Jenkins环境中执行的位置**。在pipeline代码块的顶层agent必须进行定义，但在stage级使用是可选的。

| **需要** | 是                                        |
| -------- | ----------------------------------------- |
| **参数** | 见参数说明                                |
| **允许** | 在pipeline顶层代码块或每个stage级代码块中 |

```
agent {
    node {
        label "master" //指定运行的节点标签或名称
        customWorkspace "${workspace}" //指定运行工作目录（可选）
    }
}
options {
    timestamps() //日志会有时间
    skipDefaultCheckout() //删除隐式checkout scm语句
    disableCoucurrentBuilds() //禁止并行
    timeout(time:1, unit:'HOURS') //流水线超时设置1小时
}
```

##### 参数列表

为实现Pipeline可能拥有的各种用例，agent支持几种不同类型的参数。这些参数可以应用于pipeline块的顶层，也可以应用在每个stage指令内。

**any**

在任何可用的agent 上执行Pipeline或stage。例如：agent any
**none**

当在pipeline块的顶层使用none时，将不会为整个Pipeline运行分配全局agent ，每个stage部分将需要定义其自己的agent。

**label**

提供label标签名称，在Jenkins环境中可用的agent上执行Pipeline或stage。

例如：agent { label 'my-defined-label' }

**node**

agent { node { label 'labelName' } }，等同于 agent { label 'labelName' }，但node也允许其他选项（如customWorkspace）。

**docker**

定义此参数时，执行Pipeline或stage时会动态提供一个docker节点去运行基于Docker的Pipelines。docker还可以接受一个args参数，直接传递给docker run指令调用。

例如：agent { docker 'maven:3-alpine' }或

```
agent {
    docker {
        image 'maven:3-alpine'
        label 'my-defined-label'
        args  '-v /tmp:/tmp'
    }
}
```

**dockerfile**

使用从Dockerfile仓库中包含的dockerfile创建镜像文件来构建执行Pipeline或stage。为了使用此选项，Jenkinsfile必须从Multibranch Pipeline或“Pipeline from SCM"中加载。

默认目录是在Dockerfile仓库的根目录：agent { dockerfile true }。如果Dockerfile需在另一个目录中建立，可使用dir选项：agent { dockerfile { dir 'someSubDir' } }。

还可以通过docker build ...使用additionalBuildArgs选项，如agent { dockerfile { additionalBuildArgs '--build-arg foo=bar' } }。

##### 通用选项

这些是可以应用于两个或多个agent中的选项。除非明确定义，否则非必需。

**label**

string字符串。标记在哪里运行pipeline或stage

此选项适用于node，docker和dockerfile，并且在node中是必需的。
**customWorkspace**

string字符串。自定义运行的工作空间,它可以是相对路径，在这种情况下，自定义工作区将位于node节点工作空间的根目录下，也可以是绝对路径。例如：

```
agent {
    node {
        label 'my-defined-label'
        customWorkspace '/some/other/path'
    }
}
```

**reuseNode**
一个布尔值，默认为false。如果为true，则在同一工作空间中，此选项适用于docker和dockerfile，并且仅在独立stage中使用agent时才有效。

##### 样例

```
//Jenkinsfile (Declarative Pipeline)
pipeline {
    agent { docker 'maven:3-alpine' } ①
    stages {
        stage('Example Build') {
            steps {
                sh 'mvn -B clean verify'
            }
        }
    }
}
```

**①**使用‘maven:3-alpine’的镜像创建容器，执行pipeline的所有步骤。

```
//Jenkinsfile (Declarative Pipeline)
pipeline {
    agent none ①
    stages {
        stage('Example Build') {
            agent { docker 'maven:3-alpine' } ②
            steps {
                echo 'Hello, Maven'
                sh 'mvn --version'
            }
        }
        stage('Example Test') {
            agent { docker 'openjdk:8-jre' } ③
            steps {
                echo 'Hello, JDK'
                sh 'java -version'
            }
        }
    }
}
```

**①**agent none在Pipeline顶层定义，表示将不会为整个Pipeline运行分配全局agent，每个stage需自己设置agent。

**②**使用‘maven:3-alpine’的镜像创建容器，执行此阶段中的步骤。

**③**使用‘openjdk:8-jre’的镜像创建容器，执行此阶段中的步骤。



#### stages

指定stages阶段，用于连接各个交付过程，如构建，测试和部署等。

| **需要** | 是                               |
| -------- | -------------------------------- |
| **参数** | 无                               |
| **允许** | 只能有一次，在pipeline代码块内。 |

##### 样例

```
//Jenkinsfile (Declarative Pipeline)
pipeline {
    agent any
    stages { ①
        //下载代码
        stage("GetCode"){ //阶段名称
            steps { //步骤
                timeout(time:5, unit:"MINUTES") //步骤超时时间
                script{ //填写运行代码
                    println('获取代码')
                }
            }
        }
        //构建代码
        stage("Build"){
            ...
        }
    }
}
```

**①**stages章节通常跟随在agent,options等指令后面。

#### post

指定构建后操作

| **需要** | 否                                        |
| -------- | ----------------------------------------- |
| **参数** | 无                                        |
| **允许** | 在pipeline顶层代码块或每个stage级代码块中 |

##### 参数列表

**always**

总是执行脚本片段

**changed**

只有当前Pipeline运行的状态与先前完成的Pipeline的状态不同时，才能运行。

**failure**

只有当前Pipeline处于“**失败**”状态时才运行，通常用红色指示的Web UI表示。

**success**

只有当前Pipeline具有“**成功**”状态时才运行，通常用蓝色或绿色指示的Web UI表示。

**unstable**

只有当前Pipeline具有“**不稳定**”状态，一般由测试失败，代码违例等引起，才能运行。通常用黄色指示的Web UI表示。

**aborted**

只有当前Pipeline处于“**中止**”状态时，才会运行，通常是由于Pipeline被手动中止。通常用灰色指示的Web UI表示。

correnBuild是一个全局变量

- description：构建描述

##### 样例

```
//Jenkinsfile (Declarative Pipeline)
pipeline {
    agent any
    stages {
        stage('Example') {
            steps {
                echo 'Hello World'
            }
        }
    }
    //构建后操作
    post { ①
        always { ②
            script{
                println("always")
            }
        }
        success {
            script{
                currentBuild.description += "\n 构建成功！"
            }
        }
        failure {
            script{
                currentBuild.description += "\n 构建失败！"
            }
        }
    }
}
```

①post章节通常会放在pipeline末端。

②post代码块里包括steps章节的内容。

### Directives （指令）

#### environment

environment指令指定一系列键值对，这些键值对将被定义为所有step或stage中step的环境变量，具体取决于environment指令在Pipeline中的位置。

该指令支持一种特殊的方法credentials()，可通过标识符访问Jenkins环境中预定义好的Credential凭证。

对于“Secret Text”类型的凭据，credentials()方法需确保指定的环境变量包含Secret Text内容，对于“Standard username and password"”类型的凭证，指定的环境变量需要设置为username:password。

| **需要** | 否                            |
| -------- | ----------------------------- |
| **参数** | 无                            |
| **允许** | 在pipeline块内或stage指令内。 |

##### 样例

```
//Jenkinsfile (Declarative Pipeline)
environment
pipeline {
    agent any
    environment { ①
        CC = 'clang'
    }
    stages {
        stage('Example') {
            environment { ②
                AN_ACCESS_KEY = credentials('my-prefined-secret-text') ③
            }
            steps {
                sh 'printenv'
            }
        }
    }
}
```

**①**environment指令放在pipeline顶级块中，将适用pipeline所有步骤。

**②**environment指令放在stage中，给定的环境变量将只适用该stage中的步骤。

**③**environment块中使用credentials()方法，可以访问Jenkins环境中预定义的凭证。

#### options

options指令允许在Pipeline内配置Pipeline专用选项。Pipeline本身提供了许多选项，例如buildDiscarder，它们也可以由Jenkins插件提供，例如 timestamps。

| **需要** | 否                               |
| -------- | -------------------------------- |
| **参数** | 无                               |
| **允许** | 只能有一次，在pipeline代码块内。 |

##### 参数列表

**buildDiscarder**

pipeline保持构建的最大个数。例如：

options{buildDiscarder(logRotator(numToKeepStr: '1'))}

**disableConcurrentBuilds**

不允许并行执行Pipeline,可用于防止同时访问共享资源等。例如：

options {disableConcurrentBuilds()}

**skipDefaultCheckout**

默认跳过来自源代码控制的代码。例如：

options {skipDefaultCheckout()}

**skipStagesAfterUnstable**

一旦构建状态进入了“Unstable”状态，就跳过此stage。例如：

options {skipStagesAfterUnstable()}

**timeout**
设置Pipeline运行的超时时间。例如：

options {timeout(time: 1, unit: 'HOURS')}F

**retry**

失败后，重试整个Pipeline的次数。例如：

options {retry(3)}

**timestamps**

预定义由Pipeline生成的所有控制台输出时间。例如：

options {timestamps()}

##### 样例

```
//Jenkinsfile (Declarative Pipeline)
pipeline {
    agent any
    options { 
        timeout(time: 1, unit: 'HOURS') ①
    }
    stages {
        stage('Example') {
            steps {
                echo 'Hello World'
            }
        }
    }
}
```

**①**设置pipeline全局的超时时间为1小时，超时后将会自动终止pipeline运行。

#### parameters

parameters指令提供用户在触发Pipeline时的参数列表。这些参数值通过params对象可用于Pipeline步骤，具体用法如下

| **需要** | 否                               |
| -------- | -------------------------------- |
| **参数** | 无                               |
| **允许** | 只能有一次，在pipeline代码块内。 |

##### 参数列表

**string**

string类型的参数, 例如:

```
parameters { 
string(name: 'DEPLOY_ENV', defaultValue: 'staging', description: '')
 }
```

**booleanParam**

boolean类型的参数, 例如:

```
parameters {
 booleanParam(name: 'DEBUG_BUILD', defaultValue: true, description: '') 
}
```

截至发稿，Jenkins社区目前已支持[booleanParam, choice, credentials, file, text, password, run, string]这几种参数类型，其他高级参数化类型也在陆续完善中。

##### 样例

```
//Jenkinsfile (Declarative Pipeline)
pipeline {
    agent any
    parameters {
        string(name: 'PERSON', defaultValue: 'Mr Jenkins', description: 'Who should I say hello to?')
    }
    stages {
        stage('Example') {
            steps {
                echo "Hello ${params.PERSON}"
            }
        }
    }
}
```

#### triggers

triggers指令定义了Pipeline自动化触发的方式。对于与源代码集成的Pipeline，如GitHub或BitBucket，triggers可能不需要基于webhook的集成也已经存在。目前只有两个可用的触发器：cron、pollSCM和upstream。

| **需要** | 否                               |
| -------- | -------------------------------- |
| **参数** | 无                               |
| **允许** | 只能有一次，在pipeline代码块内。 |

**cron**

接受一个cron风格的字符串来定义Pipeline触发的常规间隔，例如：

triggers {cron('H 4/* 0 0 1-5')}

**pollSCM**
接受一个cron风格的字符串来定义Jenkins检查SCM源更改的常规间隔。如果存在新的更改，则Pipeline将被重新触发。例如：triggers {pollSCM('H 4/* 0 0 1-5')}

**upstream**

可接受多个job名称以及一个threshold设置参数。任何一个job以符合threshold条件完成后，均可以触发Pipeline的运行。举例：{ upstream(upstreamProjects: 'job1,job2', threshold: hudson.model.Result.SUCCESS) }

##### 样例

```
//Jenkinsfile (Declarative Pipeline)
pipeline {
    agent any
    triggers {
        cron('H 4/* 0 0 1-5')
    }
    stages {
        stage('Example') {
            steps {
                echo 'Hello World'
            }
        }
    }
}
```

#### stage

stage指令包含在stages中，包含step、agent（可选）或其他特定包含于stage中的指令。实际上，Pipeline完成的所有实际工作都包含在一个或多个stage指令中。

| **需要** | 至少一个                                  |
| -------- | ----------------------------------------- |
| **参数** | 一个强制参数，一个标识stage名称的字符串。 |
| **允许** | 在stages章节内。                          |

##### 样例

```
//Jenkinsfile (Declarative Pipeline)
pipeline {
    agent any
    stages {
        stage('Example') {
            steps {
                echo 'Hello World'
            }
        }
    }
}
```

#### tools

通过tools可自动安装工具，并放置环境变量到PATH。如果agent none，这将被忽略。

| **需要** | 否                            |
| -------- | ----------------------------- |
| **参数** | 无                            |
| **允许** | 在pipeline块内或stage指令内。 |

**支持的Tools**

maven

jdk

gradle

##### 样例

```
//Jenkinsfile (Declarative Pipeline)
pipeline {
    agent any
    tools {
        maven 'apache-maven-3.0.1' ①
    }
    stages {
        stage('Example') {
            steps {
                sh 'mvn --version'
            }
        }
    }
}
```

**①**调用的tool必须被预置在Jenkins中，可通过**Manage Jenkins**→**Global Tool Configuration配置。**

#### when

when指令允许Pipeline根据给定的条件确定是否执行该阶段。when指令必须至少包含一个条件，如果when指令包含多个条件，则只有所有子条件返回true时才会执行stage，这与子条件嵌套在allOf相同（见下面的例子）。

更复杂的条件结构可使用嵌套条件：not，allOf或anyOf，嵌套条件可以嵌套到任意深度。

| **需要** | 否              |
| -------- | --------------- |
| **参数** | 无              |
| **允许** | 在stage指令内。 |

##### 内置条件

**branch**

当正在构建的分支与给出的分支模式匹配时执行，例如：when { branch 'master' }。请注意，这仅适用于multibranch Pipeline。

**environment**

当指定的环境变量设置为指定值时执行，例如： when { environment name: 'DEPLOY_TO', value: 'production' }

**expression**

当指定的Groovy表达式求值为true时执行，例如： when { expression { return params.DEBUG_BUILD } }

**not**

当嵌套条件为false时执行。必须包含一个条件。例如：when { not { branch 'master' } }

**allOf**

当所有嵌套条件都为true时执行。必须至少包含一个条件。例如：when { allOf { branch 'master'; environment name: 'DEPLOY_TO', value: 'production' } }

**anyOf**

当至少一个嵌套条件为真时执行。必须至少包含一个条件。例如：when { anyOf { branch 'master'; branch 'staging' } }

##### 样例

```
//Jenkinsfile (Declarative Pipeline)
pipeline {
    agent any
    stages {
        stage('Example Build') {
            steps {
                echo 'Hello World'
            }
        }
        stage('Example Deploy') {
            when {
                branch 'production'
            }
            steps {
                echo 'Deploying'
            }
        }
    }
}
//Jenkinsfile (Declarative Pipeline)
pipeline {
    agent any
    stages {
        stage('Example Build') {
            steps {
                echo 'Hello World'
            }
        }
        stage('Example Deploy') {
            when {
                expression { BRANCH_NAME ==~ /(production|staging)/ }
                anyOf {
                    environment name: 'DEPLOY_TO', value: 'production'
                    environment name: 'DEPLOY_TO', value: 'staging'
                }
            }
            steps {
                echo 'Deploying'
            }
        }
    }
}
```

### Parallel(并行)

Declarative Pipeline的stages中可能包含多个嵌套的stage, 对相互不存在依赖的stage可以通过并行的方式执行，以提升pipeline的运行效率。

另外，通过在某个stage中设置“failFast true”，可实现当这个stage运行失败的时候，强迫所有parallel stages中止运行（详见下面的例子）。

##### 样例

```
//Jenkinsfile (Declarative Pipeline)
pipeline {
    agent any
    stages {
        stage('Non-Parallel Stage') {
            steps {
                echo 'This stage will be executed first.'
            }
        }
        stage('Parallel Stage') {
            when {
                branch 'master'
            }
            failFast true
            parallel {
                stage('Branch A') {
                    agent {
                        label "for-branch-a"
                    }
                    steps {
                        echo "On Branch A"
                    }
                }
                stage('Branch B') {
                    agent {
                        label "for-branch-b"
                    }
                    steps {
                        echo "On Branch B"
                    }
                }
            }
        }
    }
}
```

### Steps（步骤）

Declarative Pipeline可使用Pipeline Steps手册中的所有可用步骤，以及以下仅在Declarative Pipeline中支持的步骤。

Pipeline Stepsreference：https://jenkins.io/doc/pipeline/steps/

#### script

script步骤中可以引用script Pipeline语句，并在Declarative Pipeline中执行。对于大多数用例，script在Declarative Pipeline中的步骤不是必须的，但它可以提供一个有用的加强。

##### 样例

```
//Jenkinsfile (Declarative Pipeline)
pipeline {
    agent any
    stages {
        stage('Example') {
            steps {
                echo 'Hello World'
                script {
                    def browsers = ['chrome', 'firefox']
                    for (int i = 0; i < browsers.size(); ++i) {
                        echo "Testing the ${browsers[i]} browser"
                    }
                }
            }
        }
    }
}
```



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

