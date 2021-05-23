# 后端项目发布流水线

获取代码→编译打包→构建镜像→发布部署

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

