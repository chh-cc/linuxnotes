## 操作系统

### cpu

1. 什么是cpu使用率和平均负载

   cpu使用率：cpu非空闲状态下运行的时间占比，反映cpu的繁忙程度

   平均负载：单位时间内系统处于可运行或不可中断的进程数

2. 平均负载太高如何排查

   可以用mpstat或者pidstat命令查看是cpu使用率升高导致的还是iowait升高导致的还是多个进程在抢占cpu导致的

### 进程和线程

1. 

### 内存

1. free命令的total、used、free、buffer、cache之间的关系

   total=used+free

   应用程序可使用的物理内存值为free+buffer+cache





