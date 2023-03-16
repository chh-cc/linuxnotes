## 凭证管理

需要安装Credentials Binding插件



每个凭据会有一个唯一的ID，这个ID很重要，可以自定义也可以随机生成。（后续流水线中引用都是通过ID引用的）



Jenkins的凭据有很多种类型，可以将一些敏感信息存储为凭据，然后在项目中引用。 

- `Username with password`类型存储Harbor或者其他系统的用户名和密码。
- `GitLab API token`类型存储Gitlab的用户API token。
- `Secret text`类型可以用来存储OpenShift等系统中的token。
- `Certificate`类型可以用户存储证书，例如k8s的用户证书。

### 配置gitlab凭证

在jenkins服务器使用root用户生成公钥和私钥

```shell
ssh-keygen -t rsa
```

把生成的**公钥**放在Gitlab中

<img src="assets/image-20230316163752733.png" alt="image-20230316163752733" style="zoom:67%;" />

在Jenkins中添加凭证，配置**私钥**

<img src="assets/image-20220612121229176.png" alt="image-20220612121229176" style="zoom:67%;" />



在jenkins的job里使用凭证**连接gitlab**：

配置前先说一个坑：

<img src="assets/image-20220612121348286.png" alt="image-20220612121348286" style="zoom:67%;" />

这是因为 jenkins 我们 yum 装的运行用户是 jenkins 用户，此处是 jenkins 用户去 git 仓库进行拉取，而 jenkins 用户的话默认是 /bin/false 的，不但不能登录，也没有 git 命令权限，所以肯定是失败的。

解决此问题两种办法：

- 更改jenkins用户为root用户；

- 更改jenkins用户为正常的普通用户/bin/bash，将其的公钥加入到git服务器的git用户中。

### 配置阿里云镜像仓库凭证

jenkins需要把构建好的docker镜像上传到镜像仓库

<img src="assets/image-20220612121514671.png" alt="image-20220612121514671" style="zoom:67%;" />

## 参数化构建

我们可以在jenkins定义变量，把jenkins的一些命令参数化，便于管理jenkinsfile

**choice parameter**

jenkins中定义变量：

<img src="assets/image-20230316172353724.png" alt="image-20230316172353724" style="zoom:67%;" />

效果：

<img src="assets/image-20220609214943300.png" alt="image-20220609214943300" style="zoom:67%;" />



**git parameter**

![img](assets/1235834-20180905174228683-1325675254.png)

![img](assets/1235834-20180905174238241-1229199729.png)

![img](assets/1235834-20180905174314569-1455405236.png)

![img](assets/1235834-20180905174352937-817873860.png)

**list git branches**

定义变量：

<img src="assets/image-20220609220012202.png" alt="image-20220609220012202" style="zoom:67%;" />

效果：

<img src="assets/image-20220609220220314.png" alt="image-20220609220220314" style="zoom:67%;" />

可以用正则只显示/refs/heads/后面的内容：

<img src="assets/image-20220609220328609.png" alt="image-20220609220328609" style="zoom:80%;" />



**string parameter**

定义变量：

<img src="assets/image-20220609222653988.png" alt="image-20220609222653988" style="zoom:67%;" />

效果：

<img src="assets/image-20220609222716207.png" alt="image-20220609222716207" style="zoom:67%;" />

