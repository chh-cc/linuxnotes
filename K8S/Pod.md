# Pod

## 为什么用Pod

一个应用不可能由单个容器支撑的，肯定有多个微服务，如果用k8s直接去控制裸容器的话，不能保证俩容器在同一宿主机上，不能保证俩容器能共享一个目录

k8s不是只编排docker容器

## 什么是Pod

Pod是k8s最小的单元，由一组、一个或多个容器组成，每个Pod还包含一个Pause容器，Pause容器是Pod的父容器，负责僵尸进程的回收管理，通过Pause容器可以使同一个Pod里面的多个容器共享存储、网络、PID、IPC等。

<img src="https://gitee.com/c_honghui/picture/raw/master/img/20211114000209.png" alt="image-20211114000209381" style="zoom:67%;" />

一个pod对应一个pause容器：

![image-20211113235653751](https://gitee.com/c_honghui/picture/raw/master/img/20211113235653.png)

## 创建一个Pod

![image-20211114160610771](https://gitee.com/c_honghui/picture/raw/master/img/20211114160617.png)

![image-20211114160741362](https://gitee.com/c_honghui/picture/raw/master/img/20211114160741.png)

![image-20211114160820240](https://gitee.com/c_honghui/picture/raw/master/img/20211114160820.png)

![image-20211114160851421](https://gitee.com/c_honghui/picture/raw/master/img/20211114160851.png)

![image-20211114160924393](https://gitee.com/c_honghui/picture/raw/master/img/20211114160924.png)

## Pod探针

### 探针种类

StartupProbe：k8s1.16版本新加的探测方式，用于判断容器内的应用程序是否已经启动，如果配置了StartupProbe，就会先禁止其他的探针，直到它成功为止，成功后将不再进行探测。（程序启动慢可以配这个）

LivenessProbe：用于探测容器是否运行，如果探测失败，kubelet会根据配置的重启策略进行相应处理，若没有配置该探针，默认就是success。

ReadinessProbe：一般用于探测容器内的程序是否健康，它的返回值是success就代表这个容器已经启动完成，并且程序可以接收流量的状态。

### 探针的检测方式

ExecAction：在容器内执行一个命令，如果返回值为0，则认为容器健康（类似linux的echo $?命令）

TCPSocketAction：通过TCP连接检查容器内的端口是否是通的，如果是通的就认为容器健康。

**HttpGetAction**：通过应用程序暴露的API地址来检查程序是否正常，如果状态码在200~400之间则认为容器健康。

### 探针检查参数

```yaml
initialDelay：10 #初始化时间
timeoutSeconds: 2 #超时时间
periodSeconds: 10 #检测间隔
successThreshold: 1 #检查成功1次表示就绪
failureThreshold: 1 #检查失败1次表示未就绪
```

### 为什么用StartupProbe

startupProbe 和 livenessProbe 最大的区别就是startupProbe在探测成功之后就不会继续探测了，而livenessProbe在pod的生命周期中一直在探测。

假如配置了如下LivenessProbe

```yaml
livenessProbe:
  httpGet:
    path: /test
    prot: 80
  failureThreshold: 1
  initialDelay：10
  periodSeconds: 10
```

上面配置的意思是容器启动10s后每10s检查一次，允许失败的次数是1次。如果失败次数超过1则会触发restartPolicy。

但如果服务启动很慢的话比如60s，这个时候如果还是用上面的探针就会进入死循环，因为上面的探针10s后就开始探测，这时候我们服务并没有起来，发现探测失败就会触发restartPolicy。

如果改成如下配置

```yaml
livenessProbe:
  httpGet:
    path: /test
    prot: 80
  failureThreshold: 4
  initialDelay：30
  periodSeconds: 10
```

确实这样pod能够启动起来了，但在后期的探测中，你需要10*4=40s才能发现这个pod不可用。

在这时候我们把`startupProbe`和`livenessProbe`结合起来使用就可以很大程度上解决我们的问题。

```yaml
livenessProbe:
  httpGet:
    path: /test
    prot: 80
  failureThreshold: 1
  initialDelay：10
  periodSeconds: 10

startupProbe:
  httpGet:
    path: /test
    prot: 80
  failureThreshold: 10
  initialDelay：10
  periodSeconds: 10
```

`startupProbe`配置的是10s\*10+10s，也就是说只要应用在110s内启动都是OK的，一旦启动探针探测成功之后，就会被livenessProbe接管，这样应用挂掉了10s就会发现问题。

## Pod退出流程

用户执行删除操作，发送delete pod命令

执行`pod get` 命令显示pod状态变为Terminating，Endpoint删除该Pod的IP地址

如果pod中的一个容器定义了 preStop hook，就在容器中调用它。如果过了宽限期（terminationGracePeriodSeconds）preStop还在运行，则延长一个短的宽限期（2秒）。如果要延长preStop，你要修改terminationGracePeriodSeconds

宽限期过期时，pod中所有运行的进程被SIGKILL杀死