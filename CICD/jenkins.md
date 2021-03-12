# jenkins

![image-20210312173405619](https://gitee.com/c_honghui/picture/raw/master/img/20210312173405.png)

## 安装

## 用户管理

针对开发、运维、测试针对不同角色进行不同权限划分

安装Role-based Authorization Strategy插件:

系统管理->管理插件-可选插件->搜索该插件选中直接安装即可

![img](https://gitee.com/c_honghui/picture/raw/master/img/20210312153338.png)

开启该插件功能:

系统管理->全局安全设置-授权策略->选中该插件功能即可->保存

![img](https://gitee.com/c_honghui/picture/raw/master/img/20210312153355.png)

权限划分:

安装 Role-Based Strategy 插件后，系统管理 中多了如图下所示的一个功能，用户权限的划分就是靠他来做的

![img](https://gitee.com/c_honghui/picture/raw/master/img/20210312153422.png)

1、Manage Roles（管理角色）

针对角色赋予不同权限，然后在将该角色分配给用户。角色就**相当于一个组**。其里面又有Global roles（全局）、Project roles（项目）、Slave roles（），来进行不同划分。

![img](https://gitee.com/c_honghui/picture/raw/master/img/20210312154146.png)

在global roles下添加user角色,并给其只读权限:

![img](https://gitee.com/c_honghui/picture/raw/master/img/20210312154201.png)

在project roles下创建一个ItemA项目角色，并给其构建、取消、读取、读取空间权限：

Pattern：是用来做正则匹配的（匹配的内容是Job(项目名)）

![img](https://gitee.com/c_honghui/picture/raw/master/img/20210312154310.png)

2、Assigin roles（分配角色）

给test1用户分配user角色：

![img](https://gitee.com/c_honghui/picture/raw/master/img/20210312154334.png)

将test1用户分配有 ItemA 项目角色：

![img](https://gitee.com/c_honghui/picture/raw/master/img/20210312154348.png)

此时可以在 test1 用户这里看到 ItemA 项目角色所匹配到的项目 A-web1

## 凭证管理

凭证( cridential )是Jenkins进行受限操作时的凭据。比如使用SSH登录远程机器时，用户名和密码或SSH key就是凭证。

### 让jenkins远程登陆gitlab

安装Credentials Binding插件:

![img](https://gitee.com/c_honghui/picture/raw/master/img/20210312141044.png)

安装成功之后，首页左边多了"凭据"菜单，在这里管理所有凭证。可以添加的凭证有5种：

![img](https://gitee.com/c_honghui/picture/raw/master/img/20210312141100.png)

- Username with password：用户名和密码。（常用）
- SSH Username with private key：使用SSH用户和密钥。（常用）

安装ssh插件:

![img](https://gitee.com/c_honghui/picture/raw/master/img/20210312141137.png)

在jenkins使用root用户生成公钥和私钥

```shell
ssh-keygen -t rsa
```

把生成的公钥放在Gitlab中

![img](https://gitee.com/c_honghui/picture/raw/master/img/20210312141317.png)

在Jenkins中添加凭证，配置私钥

![img](https://gitee.com/c_honghui/picture/raw/master/img/20210312141520.png)

安装Git Parameter插件：

![img](https://gitee.com/c_honghui/picture/raw/master/img/20210312141858.png)

配置：

配置前先说一个坑：

![img](https://gitee.com/c_honghui/picture/raw/master/img/20210312141910.png)

这是因为 jenkins 我们 yum 装的运行用户是 jenkins 用户，此处是 jenkins 用户去 git 仓库进行拉取，而 jenkins 用户的话默认是 /bin/false 的，不但不能登录，也没有 git 命令权限，所以肯定是失败的。

解决此问题两种办法：

- 更改jenkins用户为root用户；

- 更改jenkins用户为正常的普通用户/bin/bash，将其的公钥加入到git服务器的git用户中。

## 视图管理

创建视图:

![img](https://gitee.com/c_honghui/picture/raw/master/img/20210312142344.png)

![img](https://gitee.com/c_honghui/picture/raw/master/img/20210312142351.png)

![img](https://gitee.com/c_honghui/picture/raw/master/img/20210312142403.png)

将项目加入视图中:

![img](https://gitee.com/c_honghui/picture/raw/master/img/20210312142648.png)

![img](https://gitee.com/c_honghui/picture/raw/master/img/20210312142840.png)

状态图标变绿:

安装Green Balls插件

看板:

构建可视化，让所有人可以自助查到某个系统在某个环境的部署情况。
Build Monitor View插件(https://plugins.jenkins.io/build-monitor-plugin )可以将Jenkins项目以一块“看板”的形式呈现。

1.单击“+”号添加新视图

2.进入添加表单后，选择“Build Monitor View”选项

![file](https://gitee.com/c_honghui/picture/raw/master/img/20210312143213.png)

3.进入“Build Monitor View”编辑页，可以选择在视图中显示哪些job，以及它们的排序规则

4.编辑完成后页面

![file](https://gitee.com/c_honghui/picture/raw/master/img/20210312143251.png)

当构建失败时，就会出现红块。

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

## 流水线发布项目

创建一个job

![image-20210312174604185](https://gitee.com/c_honghui/picture/raw/master/img/20210312174604.png)

参数构建

配置pipeline或shell脚本(拉取和发布代码的脚本)

构建