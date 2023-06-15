## 配置gitlab凭证

在jenkins服务器使用root用户生成公钥和私钥

```shell
ssh-keygen -t rsa
```

把生成的**公钥**放在Gitlab中

<img src="assets/image-20230316163752733.png" alt="image-20230316163752733" style="zoom:67%;" />

在Jenkins中添加凭证，配置**私钥**

<img src="assets/image-20220612121229176-1686638971441.png" alt="image-20220612121229176" style="zoom:67%;" />

在jenkins的job里使用凭证**连接gitlab**：

配置前先说一个坑：

<img src="assets/image-20220612121348286-1686640226659.png" alt="image-20220612121348286" style="zoom:67%;" />

这是因为 jenkins 我们 yum 装的运行用户是 jenkins 用户，此处是 jenkins 用户去 git 仓库进行拉取，而 jenkins 用户的话默认是 /bin/false 的，不但不能登录，也没有 git 命令权限，所以肯定是失败的。

解决此问题两种办法：

- 更改jenkins用户为root用户；

- 更改jenkins用户为正常的普通用户/bin/bash，将其的公钥加入到git服务器的git用户中。

### 配置阿里云镜像仓库凭证

jenkins需要把构建好的docker镜像上传到镜像仓库

<img src="assets/image-20220612121514671-1686640283057.png" alt="image-20220612121514671" style="zoom:67%;" />

## 参数化构建

我们可以在jenkins定义变量，把jenkins的一些命令参数化，便于管理jenkinsfile

**choice parameter**

jenkins中定义变量：

<img src="assets/image-20230316172353724.png" alt="image-20230316172353724" style="zoom:67%;" />

效果：

<img src="assets/image-20220609214943300-1686640287003.png" alt="image-20220609214943300" style="zoom:67%;" />



**git parameter**

![img](assets/1235834-20180905174228683-1325675254.png)

![img](assets/1235834-20180905174238241-1229199729.png)

![img](assets/1235834-20180905174314569-1455405236.png)

![img](assets/1235834-20180905174352937-817873860.png)

**list git branches**

定义变量：

<img src="assets/image-20220609220012202-1686640301429.png" alt="image-20220609220012202" style="zoom:67%;" />

效果：

<img src="assets/image-20220609220220314-1686640303363.png" alt="image-20220609220220314" style="zoom:67%;" />

可以用正则只显示/refs/heads/后面的内容：

<img src="assets/image-20220609220328609-1686640304806.png" alt="image-20220609220328609" style="zoom:80%;" />



**string parameter**

定义变量：

<img src="assets/image-20220609222653988-1686640306518.png" alt="image-20220609222653988" style="zoom:67%;" />

效果：

<img src="assets/image-20220609222716207-1686640308473.png" alt="image-20220609222716207" style="zoom:67%;" />