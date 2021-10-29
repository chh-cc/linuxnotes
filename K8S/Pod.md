# Pod

## 什么是Pod

Pod是k8s最小的单元，由一组、一个或多个容器组成，每个Pod还包含一个Pause容器，Pause容器是Pod的父容器，负责僵尸进程的回收管理，通过Pause容器可以使同一个Pod里面的多个容器共享存储、网络、PID、IPC等。

## 创建一个Pod

