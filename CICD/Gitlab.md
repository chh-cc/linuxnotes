# Gitlab

## 安装

## 权限

### 概念

```text
1.项目由项目组来创建，而不是由用户创建
2.用户通过加入到不同的组，来实现对项目的访问或者提交
3.项目可以设置为只有项目组可以查看，所有登陆用户可以查看和谁都可以看三种
```

Gitlab用户在组中有五种权限：Guest、Reporter、Developer、Master、Owner

- Guest：可以创建issue、发表评论，不能读写版本库
- Reporter：可以克隆代码，不能提交，QA、PM可以赋予这个权限
- Developer：可以克隆代码、开发、提交、push，RD可以赋予这个权限
- Master：可以创建项目、添加tag、保护分支、添加项目成员、编辑项目，核心RD负责人可以赋予这个权限
- Owner：可以设置项目访问权限 - Visibility Level、删除项目、迁移项目、管理组成员，开发组leader可以赋予这个权限

Gitlab中的组和项目有三种访问权限：Private、Internal、Public

- Private：只有组成员才能看到
- Internal：只要登录的用户就能看到
- Public：所有人都能看到

开源项目和组设置的是Internal

### 操作流程

```text
1.创建组
2.基于组创建项目
3.创建用户，分配组，分配权限
```

![image-20210312105318823](https://gitee.com/c_honghui/picture/raw/master/img/20210312105318.png)

### 实操

#### 创建组

点击new group

![img](https://gitee.com/c_honghui/picture/raw/master/img/20210312105621.png)

填组名:

<img src="https://gitee.com/c_honghui/picture/raw/master/img/20210312105643.png" alt="img" style="zoom: 67%;" />

检查:

![img](https://gitee.com/c_honghui/picture/raw/master/img/20210312105829.png)

#### 创建项目

点击new project

<img src="https://gitee.com/c_honghui/picture/raw/master/img/20210312110054.png" alt="img" style="zoom:67%;" />

选择dev组，创建game项目

<img src="https://gitee.com/c_honghui/picture/raw/master/img/20210312110119.png" alt="img" style="zoom:50%;" />

#### 创建用户

点击创建用户

<img src="https://gitee.com/c_honghui/picture/raw/master/img/20210312110343.png" alt="img" style="zoom:67%;" />

创建cto用户

![img](https://gitee.com/c_honghui/picture/raw/master/img/20210312110425.png)

<img src="https://gitee.com/c_honghui/picture/raw/master/img/20210312110438.png" alt="img" style="zoom: 50%;" />

创建密码

![img](https://gitee.com/c_honghui/picture/raw/master/img/20210312110829.png)

<img src="https://gitee.com/c_honghui/picture/raw/master/img/20210312110859.png" alt="img" style="zoom: 50%;" />

#### 授权

dev组添加用户:

![img](https://gitee.com/c_honghui/picture/raw/master/img/20210312111838.png)

添加cto用户:

<img src="https://gitee.com/c_honghui/picture/raw/master/img/20210312112054.png" alt="img" style="zoom: 50%;" />

检查

![image-20210312112139706](https://gitee.com/c_honghui/picture/raw/master/img/20210312112139.png)

取消用户注册

![img](https://gitee.com/c_honghui/picture/raw/master/img/20210312112216.png)