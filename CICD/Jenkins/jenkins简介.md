# jenkins简介(java开发)

默认端口:8080

jenkins_home默认/var/lib/jenkins

启动用户:默认jenkins

## Devops

- DevOps 是Development和Operations的组合，也就是开发和运维的简写。
- DevOps 是针对企业中的研发人员、运维人员和测试人员的工作理念，是他们在应用开发、代码部署和质量测试等
- 整条生命周期中协作和沟通的最佳实践，DevOps 强调整个组织的合作以及交付和基础设施变更的自动化、从而实现持续集成、持续部署和持续交付。
- DevOps 四大平台：代码托管(gitlab/svn)、项目管理(jira)、运维平台(腾讯蓝鲸/开源平台)、持续交付
  (Jenkins/gitlab)  



- 持续集成(CI)是指多名开发者在开发不同功能代码的过程当中，可以频繁的将代码行合并到一起并切相互不影响工作 

- 持续部署(CD)是基于某种工具或平台实现代码自动化的构建、测试和部署到线上环境以实现交付高质量的产品,持续部署在某种程度上代表了一个开发团队的更新迭代速率。  
- 持续交付(CD)是在持续部署的基础之上，将产品交付到线上环境，因此持续交付是产品价值的一种交付，是产品价值的一种盈利的实现。  



架构:

![image-20210312173405619](https://gitee.com/c_honghui/picture/raw/master/img/20210312173405.png)

自动构建过程 JOB，JOB 的功能主要是：获取 SVN/GIT 源码、自动编译、自动打包、部署分发和自动测试等；

源代码存储库，开发编写代码需上传至 SVN、GIT 代码库中，供 Jenkins 来获取；

Jenkins 持续集成服务器，用于部署 Jenkins UI、存放 JOB 工程、各种插件、编译打包的数据等。

## 应用场景

- 集成svn/git客户端实现源代码下载检出
- 集成maven/ant/gradle/npm等构建工具实现源码编译打包单元测试
- 集成sonarqube对源代码进行质量检查（坏味道、复杂度、新增bug等）
- 集成SaltStack/Ansible实现自动化部署发布
- 集成Jmeter/Soar/Kubernetes/.....
- 可以自定义插件或者脚本通过jenkins传参运行
- 可以说Jenkins比较灵活插件资源丰富，日常运维工作都可以自动化。

## web页面展示

- 管理页面: 系统管理页面包含系统管理、全局安全管理、全局工具配置、节点管理、授权管理、插件管理、系统备份管理、日志监控管理

<img src="https://gitee.com/c_honghui/picture/raw/master/img/20210328223816.png" alt="image-20210328223815917" style="zoom:67%;" />

- 项目管理页面: 项目、项目状态、项目视图、构建队列等信息

<img src="https://gitee.com/c_honghui/picture/raw/master/img/20210328223848.png" alt="image-20210328223848434" style="zoom:67%;" />

- 构建输出页面: 用于查看项目的构建详情，能够看到项目的构建过程及详细日志。

<img src="https://gitee.com/c_honghui/picture/raw/master/img/20210328224026.png" alt="image-20210328224026189" style="zoom:67%;" />



## 插件管理

### 必备插件

<img src="https://gitee.com/c_honghui/picture/raw/master/img/20210320212844.png" alt="image-20210320212837022" style="zoom:67%;" />





## 构建通知

## 触发构建

### Git hook自动触发构建

当gitlab发现源代码有变化时，触发jenkins执行构建。

安装两个插件：Gitlab和Gitlab Hook

配置工程，使其能够实现自动构建

![img](https://gitee.com/c_honghui/picture/raw/master/img/20210312145246.png)

取消启用/project端点授权（Manage Jenkins->Configure System）

![img](https://gitee.com/c_honghui/picture/raw/master/img/20210312145344.png)

Gitlab中开启webhook功能

![img](https://gitee.com/c_honghui/picture/raw/master/img/20210312145357.png)

Gitlab中添加项目webhook地址

点击：项目->Settings->Webhooks

<img src="https://gitee.com/c_honghui/picture/raw/master/img/20210312145443.png" alt="img" style="zoom:67%;" />

