# Git

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

个人信息配置：用户名和密码（一次即可）
git config --global user.name "zhangya"
git config --global user.email "526195417@qq.com"

管理指定文件（提交文件到暂存区（让git管理文件））：
git add 文件名

生成版本（提交当前暂存区的所有文件到本地仓库）：
git commit -m "描述信息"

查看版本记录：
git log
```

#### 查看历史提交记录

git log

![image-20210312095854466](https://gitee.com/c_honghui/picture/raw/master/img/20210312095854.png)

简单信息一行实现

![image-20210312095920057](https://gitee.com/c_honghui/picture/raw/master/img/20210312095920.png)

#### 回滚到指定版本

git log --oneline

![image-20210312100105356](https://gitee.com/c_honghui/picture/raw/master/img/20210312100105.png)

把当前版本回滚到上一个版本：

```shell
git reset --hard HEAD^

#上上个版本：HEAD^^
```



回滚到指定版本*modified a*：

![image-20210312100437861](https://gitee.com/c_honghui/picture/raw/master/img/20210312100437.png)

如果此时发现回滚错了应该回滚到bbb,但此时查看历史会发现没有bbb,因为回到过去了,那时候bbb还没提交
我们可以使用git reflog查看总历史记录

![image-20210312100530024](https://gitee.com/c_honghui/picture/raw/master/img/20210312100530.png)

然后再指定退回bbb版本

### 分支

每次提交，Git都把它们串成一条时间线，这条时间线就是一个分支。

一开始的时候，master分支是一条线，Git用master指向最新的提交，再用HEAD指向master，就能确定当前分支，以及当前分支的提交点：

![image-20210509181615601](https://gitee.com/c_honghui/picture/raw/master/img/20210509181622.png)

当我们创建新的分支，例如dev时，Git新建了一个指针叫dev，指向master相同的提交，再把HEAD指向dev，就表示当前分支在dev上：

![image-20210509181720984](https://gitee.com/c_honghui/picture/raw/master/img/20210509181721.png)

从现在开始，对工作区的修改和提交就是针对dev分支了，比如新提交一次后，dev指针往前移动一步，而master指针不变：

![image-20210509181840551](https://gitee.com/c_honghui/picture/raw/master/img/20210509181840.png)

假如我们在dev上的工作完成了，就可以把dev合并到master上。

![image-20210509181959683](https://gitee.com/c_honghui/picture/raw/master/img/20210509181959.png)

查看当前分支:`git branch`

在当前分支创建一个新分支:`git branch dev`

在当前分支创建并切换到一个新分支:`git checkout -b dev`     `git branch`

切换到指定分支:`git checkout dev `        `git branch`

合并dev分支到master分支:`git checkout master`        `git merge dev`

删除dev分支：`git branch -d dev`

创建一个远程仓库：`git remote add origin git@github.com:chen/linuxnotes.git`

从远程主分支pull到本地:`git pull origin master`

从本地push到主分支:`git push -u origin master`

冲突例子：

切换到ops_dev分支修改readme.txt文件并提交；切换到master分支修改readme.txt文件并提交

![image-20210509202233538](https://gitee.com/c_honghui/picture/raw/master/img/20210509202233.png)

这种情况下，Git无法执行“快速合并”，只能试图把各自的修改合并起来，但这种合并就可能会有冲突。

修改master分支readme.txt冲突的内容然后重新提交

![image-20210509202436417](https://gitee.com/c_honghui/picture/raw/master/img/20210509202436.png)

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





git commit

```text
Git 仓库中的提交记录保存的是你的目录下所有文件的快照，就像是把整个目录复制，然后再粘贴一样，但比复制粘贴优雅许多！

Git 希望提交记录尽可能地轻量，因此在你每次进行提交时，它并不会盲目地复制整个目录。条件允许的情况下，它会将当前版本与仓库中的上一个版本进行对比，并把所有的差异打包到一起作为一个提交记录。

Git 还保存了提交的历史记录。这也是为什么大多数提交记录的上面都有父节点的原因
```

git branch

```text
Git 的分支也非常轻量。它们只是简单地指向某个提交纪录

早建分支！多用分支！
```

git branch newImage（新创建的分支指向了提交记录c1）

![image-20210708223033201](https://gitee.com/c_honghui/picture/raw/master/img/20210708223040.png)

git checkout newImage;git commit（新的修改已经保存到新的分支）

![image-20210708223240509](https://gitee.com/c_honghui/picture/raw/master/img/20210708223240.png)

git merge bugFix（把bugFix分支的修改合并到当前分支，main现在指向了一个拥有两个父节点的提交记录）

![image-20210708223909531](https://gitee.com/c_honghui/picture/raw/master/img/20210708223909.png)

git rebase main（把当前bugFix分支合并到main分支，提交记录 C3 依然存在，而 C3是我们 Rebase 到 main 分支上的 C3 的副本）

![image-20210708224857037](https://gitee.com/c_honghui/picture/raw/master/img/20210708224857.png)

git checkout main;git rebase bugFix（由于 `bugFix` 继承自 `main`，所以 Git 只是简单的把 `main` 分支的引用向前移动了一下而已。）

![image-20210708225142001](https://gitee.com/c_honghui/picture/raw/master/img/20210708225142.png)

HEAD

```text
HEAD 总是指向当前分支上最近一次提交记录。
HEAD 通常情况下是指向分支名的（如 bugFix）,在你提交时，改变了 bugFix 的状态
```

