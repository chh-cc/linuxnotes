# 发布java项目实战

## 实战1

一条完整的pipeline交付流水线通常会包括代码获取、单元测试、静态检查、打包部署、接口层测试、UI层测试、性能专项测试（可能还有安全、APP等专项）、人工验收等研发测试环节，还会包括灰度发布、正式发布等发布环节。

1.此项目的部署还是使用传统虚拟机服务器的方式，暂未采用docker容器。
2.灰度发布、正式发布等发布环节，由于涉及到线上发布系统对接，该项目暂未包括。
3.采用jenkins官方推荐的declarative pipeline方式实现。



jenkins一些配置：

![image-20210603105044736](https://gitee.com/c_honghui/picture/raw/master/img/20210603105140.png)

![image-20210603105118319](https://gitee.com/c_honghui/picture/raw/master/img/20210603105118.png)

参数化构建界面：

![img](https://gitee.com/c_honghui/picture/raw/master/img/20210328200832.png)

jenkinsfile：

```groovy
#!groovy
pipeline{
    //在任何可用的代理上执行Pipeline
    agent any
    
    options {
        //日志会有时间戳
        timestamps()
        //禁止并行
        disableConcurrentBuilds()
        buildDiscarder(
            logRotator(
                numToKeepStr: '20',
                daysToKeepStr: '30',
            )
        )
    }
    
    //参数化变量，目前只支持[booleanParam, choice, credentials, file, text, password, run, string]这几种参数类型，其他高级参数化类型还需等待社区支持。
    parameters {
        //git代码路径【参数值对外隐藏】
        string(name:'repoUrl', defaultValue: 'git@192.168.71.136:root/service.git', description: 'git代码路径')
        //repoBranch参数后续替换成git parameter不再依赖手工输入,JENKINS-46451【git parameters目前还不支持pipeline】
        string(name:'repoBranch', defaultValue: 'master', description: 'git分支名称')
        //pom.xml的相对路径
        string(name:'pomPath', defaultValue: 'pom.xml', description: 'pom.xml的相对路径')
        //war包的相对路径
        string(name:'jarLocation', defaultValue: '/target/*.jar', description: 'jar包的相对路径 ')
        //服务器参数采用了组合方式，避免多次选择，使用docker为更佳实践【参数值对外隐藏】
        choice(name: 'server',choices:'192.168.1.107,9090,*****\n192.168.1.60,9090,*****,*****', description: '测试服务器列表选择(IP,JettyPort)')
        //测试服务器的dubbo服务端口
        string(name:'dubboPort', defaultValue: '31100', description: '测试服务器的dubbo服务端口')
        //单元测试代码覆盖率要求，各项目视要求调整参数
        string(name:'lineCoverage', defaultValue: '20', description: '单元测试代码覆盖率要求(%)，小于此值pipeline将会失败！')
        //若勾选在pipelie完成后会邮件通知测试人员进行验收
        booleanParam(name: 'isCommitQA',description: '是否邮件通知测试人员进行人工验收',defaultValue: false )
    }
    
    //环境变量，初始确定后一般不需更改
    tools {
        maven 'maven3'
        jdk   'jdk8'
    }
    
    //常量参数，初始确定后一般不需更改
    environment{
        //git服务全系统只读账号cred_id【参数值对外隐藏】
        CRED_ID='3319285d-4697-4907-9f6a-3f5ddce4cc4a'
        //测试人员邮箱地址【参数值对外隐藏】
        QA_EMAIL='*****@*****.com'
        //接口测试（网络层）的job名，一般由测试人员编写
        ITEST_JOBNAME='Guahao_InterfaceTest_ExpertPatient'
    }
    
    options {
        //保持构建的最大个数
        buildDiscarder(logRotator(numToKeepStr: '10')) 
    }
    
    //定期检查开发代码更新，工作日每晚4点做daily build
    triggers {
        pollSCM('H 4 * * 1-5')
    }
    
    //pipeline运行结果通知给触发者
    post{
        success{
            script { 
                wrap([$class: 'BuildUser']) {
                mail to: "${BUILD_USER_EMAIL }",
                subject: "PineLine '${JOB_NAME}' (${BUILD_NUMBER}) result",
                body: "${BUILD_USER}'s pineline '${JOB_NAME}' (${BUILD_NUMBER}) run success\n请及时前往${env.BUILD_URL}进行查看"
                }
            }
        }
        failure{
            script { 
                wrap([$class: 'BuildUser']) {
                mail to: "${BUILD_USER_EMAIL }",
                subject: "PineLine '${JOB_NAME}' (${BUILD_NUMBER}) result",
                body: "${BUILD_USER}'s pineline  '${JOB_NAME}' (${BUILD_NUMBER}) run failure\n请及时前往${env.BUILD_URL}进行查看"
                }
            }

        }
        unstable{
            script { 
                wrap([$class: 'BuildUser']) {
                mail to: "${BUILD_USER_EMAIL }",
                subject: "PineLine '${JOB_NAME}' (${BUILD_NUMBER})结果",
                body: "${BUILD_USER}'s pineline '${JOB_NAME}' (${BUILD_NUMBER}) run unstable\n请及时前往${env.BUILD_URL}进行查看"
                }
            }
        }
    }
    
    //pipeline的各个阶段场景
    stages {
        stage('代码获取') {
            steps {
                //根据param.server分割获取参数,包括IP,jettyPort,username,password
                script {
                    def split=params.server.split(",")
                    serverIP=split[0]
                    jettyPort=split[1]
                }
                echo "starting fetchCode from ${params.repoUrl}......"
                // Get some code from a GitHub repository
                git credentialsId:CRED_ID, url:params.repoUrl, branch:params.repoBranch
            }
        }
        
        stage('单元测试') {
            steps {
                echo "starting unitTest......"
                //注入jacoco插件配置,clean test执行单元测试代码. All tests should pass.
                sh "mvn org.jacoco:jacoco-maven-plugin:prepare-agent -f ${params.pomPath} clean test -Dautoconfig.skip=true -Dmaven.test.skip=false -Dmaven.test.failure.ignore=true"
                junit '**/target/surefire-reports/*.xml'
                //配置单元测试覆盖率要求，未达到要求pipeline将会fail,code coverage.LineCoverage>20%.
                jacoco changeBuildStatus: true, maximumLineCoverage:"${params.lineCoverage}"
            }
        }
        
        stage('静态检查') {
            steps {
                echo "starting codeAnalyze with SonarQube......"
                //sonar:sonar.QualityGate should pass
                withSonarQubeEnv('SonarQube') {
                    //固定使用项目根目录${basedir}下的pom.xml进行代码检查
                    sh "mvn -f pom.xml clean compile sonar:sonar"
                }
                script {
                    timeout(10) {
                        //利用sonar webhook功能通知pipeline代码检测结果，未通过质量阈，pipeline将会fail
                        def qg = waitForQualityGate()
                        if (qg.status != 'OK') {
                            error "未通过Sonarqube的代码质量阈检查，请及时修改！failure: ${qg.status}"
                        }
                    }
                }
            }
        }
        
        stage('部署测试环境') {
            steps {
                echo "starting deploy to ${serverIP}......"
                //编译和打包
                sh "mvn  -f ${params.pomPath} clean package -Dautoconfig.skip=true -Dmaven.test.skip=true"
                archiveArtifacts jarLocation
                script {
                    //需要安装build user vars插件
                    wrap([$class: 'BuildUser']) {
                        //发布jar包到指定服务器，如果没有sshpass需要安装
                        sh "scp ${params.jarLocation} ${serverName}@${serverIP}:htdocs/war"
                        //这里增加了一个小功能，在服务器上记录了基本部署信息，方便多人使用一套环境时问题排查，storge in {WORKSPACE}/deploy.log  & remoteServer:htdocs/war
                        Date date = new Date()
                        def deploylog="${date.toString()},${BUILD_USER} use pipeline  '${JOB_NAME}(${BUILD_NUMBER})' deploy branch ${params.repoBranch} to server ${serverIP}"
                        println deploylog
                        sh "echo ${deploylog} >>${WORKSPACE}/deploy.log"
                        sh "scp ${WORKSPACE}/deploy.log ${serverName}@${serverIP}:htdocs/war"
                        //jetty restart，重启jetty
                        sh "ssh ${serverName}@${serverIP} 'bin/jettyrestart.sh' "
                    }
                }
            }
        }
        
        stage('接口自动化测试') {
            steps{
                echo "starting interfaceTest......"
                script {
                 //为确保jetty启动完成，加了一个判断，确保jetty服务器启动可以访问后再执行接口层测试。
                 timeout(5) {
                     waitUntil {
                        try {
                            //确保jetty服务的端口启动成功
                            sh "nc -z ${serverIP} ${jettyPort}"
                            //sh "wget -q http://${serverIP}:${jettyPort} -O /dev/null"
                            return true
                        } catch (exception) {
                            return false
                            }
                        }
                    }
                //将参数IP和Port传入到接口测试的job，需要确保接口测试的job参数可注入
                 build job: ITEST_JOBNAME, parameters: [string(name: "dubbourl", value: "${serverIP}:${params.dubboPort}")]
                }
            }
        }
        
        stage('UI自动化测试') { 
             steps{
             echo "starting UITest......"
             //这个项目不需要UI层测试，UI自动化与接口测试的pipeline脚本类似
             }
        }
        
        stage('性能自动化测试 ') { 
            steps{
                 echo "starting performanceTest......"
                //视项目需要增加性能的冒烟测试，具体实现后续专文阐述
                }
        }
        
        stage('通知人工验收'){
            steps{
                script{
                    wrap([$class: 'BuildUser']) {
                    if(params.isCommitQA==false){
                        echo "不需要通知测试人员人工验收"
                    }else{
                        //邮件通知测试人员人工验收
                         mail to: "${QA_EMAIL}",
                         subject: "PineLine '${JOB_NAME}' (${BUILD_NUMBER})人工验收通知",
                         body: "${BUILD_USER}提交的PineLine '${JOB_NAME}' (${BUILD_NUMBER})进入人工验收环节\n请及时前往${env.BUILD_URL}进行测试验收"
                    }

                    }
                }
            }
        }
        // stage('发布系统') { 
        //     steps{
        //         echo "starting deploy......"
        //     }
        // }
    }
}
```

这里只实验了代码拉取和发布，结果如下：

![image-20210603110418527](https://gitee.com/c_honghui/picture/raw/master/img/20210603110418.png)

## 实战2

jenkinsfile:

```groovy
pipeline {
    //运行节点
    agent { label 'slave'}
    
    options {
        //日志会有时间戳
        timestamps()
        //禁止并行
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
    /*=======================================常修改变量-end=======================================*/
    stages {
        stage('Deploy'){
            when {
                expression { return params.DEPLOY_FLAG == 'deploy' }
            }
            stages {
                stage('Pre Env'){
                    steps {
                        echo "======================================项目名称 = ${env.JOB_NAME}"
                        echo "======================================项目 URL = ${gitUrl}"
                        echo "======================================项目分支 = ${branchName}"
                        echo "======================================当前编译版本号 = ${env.BUILD_NUMBER}"
                    }
                }
                stage('Git clone'){
                    steps {
                        git branch: "${branchName}",
                        credentialsId: "${gitlabCredentialsId}",
                        url: "${gitUrl}"
                    }
                }
                stage('Mvn Build'){
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
        
        stage('Rollback') {
            when {
                expression { return params.DEPLOY_FLAG == 'rollback' }
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

结果如下：

![image-20210603160814307](https://gitee.com/c_honghui/picture/raw/master/img/20210603160814.png)