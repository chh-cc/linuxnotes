# Git

## 开发流程

![image-20210312093309084](https://gitee.com/c_honghui/picture/raw/master/img/20210312093315.png)

## 持续集成CI

1.开发的代码持续的集成到代码仓库里就是持续集成，不用等所有人都开发完毕在合并，可以多个开发人员同时工作。

2.开发将代码提交到代码仓库，由ci服务器自动将代码拉下来进行编译，测试，然后将结果返回给开发人员。

3.持续集成的目的是可以频繁的将开发的功能进行合并，提高工作效率。

## 持续交付CD

1.持续交付就是将编译开发好的代码持续的交付到测试环境进行测试。

2.在预发布环境我们可以对代码进行质量扫描和漏洞扫描，并且将测试结果返回给测试人员。

3.如果测试的代码有问题，测试人员就通知开发人员进行修复，如果没有问题则进入下一个部署环节。

## 持续部署CD

1.代码测试没有问题之后就可以进入预发布环境进行进一步测试，如果预发布环境也没有问题可以通过jenkins服务器持续的部署到生产服务器

2.如果新部署的服务发现有问题，通过jenkins服务器可以快速的回滚到正常的代码。

## git

### 概念

- 工作区：就是你在电脑里能看到的目录。
- 暂存区：英文叫stage, 或index。一般存放在"git目录"下的index文件（.git/index）中，所以我们把暂存区有时也叫作索引（index）。
- 版本库：工作区有一个隐藏目录.git，这个不算工作区，而是Git的版本库。

### 安装

`yum install git -y`

### 配置

配置使用git的用户

```shell
git config --global user.name "zhangya"
```

配置使用git的邮箱

```shell
git config --global user.email "526195417@qq.com"
```

设置语法高亮

```shell
git config --global color.ui true
```

查看配置

```shell
[root@gitlab ~]# git config --list
user.name=zhangya
user.email=526195417@qq.com
color.ui=true

[root@gitlab ~]# cat .gitconfig 
[user]
        name = zhangya
        email = 526195417@qq.com
[color]
        ui = true
```

### 命令

想要让git对一个目录进行版本控制需要以下步骤：

```shell
创建并进入工作目录:
mkdir /git_data
cd /git_data

初始化:
git init    #让git帮助我们管理当前文件夹

查看工作目录的状态:
git status
# 位于分支 master
## 初始提交
## 未跟踪的文件:

查看隐藏目录:
ls -l .git/
branches                #分支目录 
config                  #定义项目的特有配置 
description         #描述 
HEAD                        #当前分支 
hooks                       #git钩子文件 
info                        #包含一个全局排除文件 
objects                 #存放所有数据，包含info和pack两个子文件夹 
refs                        #存放指向数据（分支）的提交对象的指针 
index                   #保存暂存区信息，在执行git init的时候，这个文件还没有

管理指定文件（提交文件到暂存区（让git管理文件））：
git add 文件名

个人信息配置：用户名和密码（一次即可）
git config --global user.name "zhangya"
git config --global user.email "526195417@qq.com"

生成版本（提交当前暂存区的所有文件到本地仓库）：
git commit -m "描述信息"

查看版本记录：
git log

撤回提交到暂存区的文件:git rm --cached 文件名
删除提交到暂存区的文件:git rm --cached 文件名    rm -f 文件名
```

#### 查看历史提交记录

git log

![image-20210312095854466](https://gitee.com/c_honghui/picture/raw/master/img/20210312095854.png)

简单信息一行实现

![image-20210312095920057](https://gitee.com/c_honghui/picture/raw/master/img/20210312095920.png)

#### 回滚到指定版本

git log --oneline

![image-20210312100105356](https://gitee.com/c_honghui/picture/raw/master/img/20210312100105.png)

回滚到指定版本*modified a*

![image-20210312100437861](https://gitee.com/c_honghui/picture/raw/master/img/20210312100437.png)

如果此时发现回滚错了应该回滚到bbb,但此时查看历史会发现没有bbb,因为回到过去了,那时候bbb还没提交
我们可以使用git reflog查看总历史记录

![image-20210312100530024](https://gitee.com/c_honghui/picture/raw/master/img/20210312100530.png)

然后再指定退回bbb版本

### 分支

把你的工作从开发主线上分离出来,以免影响开发主线

查看当前分支:`git branch`

在当前分支创建一个新分支:`git branch testing`

在当前分支创建并切换到一个新分支:`git checkout -b testing`     `git branch`

切换到指定分支:`git checkout testing`         `git branch`

合并testing分支到当前分支:`git branch`       `git merge testing`

从远程主分支pull到本地:`git pull origin master`

从本地push到主分支:`git push -u origin master`



初次在公司新电脑下载代码:

```
克隆代码仓库:
git clone 远程仓库地址
进入工作目录:
cd 工作目录
git config --global user.name "zhangya"
git config --global user.email "526195417@qq.com"
切换到dev分支:
git checkout dev
把master分支合并到dev(仅一次):
git merge master
修改代码
提交代码:
git add .
git commit -m 'xxx'
git push origin dev
```

下次继续开发:

```
切换到dev分支开发:
git checkout dev
拉最新代码(不必clone)
git pull origin dev
继续开发
提交代码
```

开发完要上线

```
把dev分支合并到master:
git checkout master
git merge dev
git push origin master
```

### 总结

![img](https://gitee.com/c_honghui/picture/raw/master/img/20210312102203.png)

![img](https://gitee.com/c_honghui/picture/raw/master/img/20210312102219.jpeg)