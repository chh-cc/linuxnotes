springboot项目引入依赖

修改pom.xml添加如下代码，引入Actuator的相关依赖

```yaml
<!-- spring-boot-actuator依赖 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
<!-- prometheus依赖 -->
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-registry-prometheus</artifactId>
</dependency>
```

在application.properties中添加相关配置暴露监测数据端口，如果是application.yaml格式，需要转成yaml

```yaml
management.endpoints.web.exposure.include=prometheus,health
management.metrics.tags.application=${spring.application.name}
```

启动服务后，访问http://192.168.101.141:8081/actuator/prometheus



修改Prometheus配置

```yaml
vim /usr/local/prometheus/prometheus.yml
scrape_configs:
  - job_name: 'springboot-demo'
    metrics_path: '/actuator/prometheus'
    static_configs:
    - targets: ['192.168.101.141:8081']
```



监控指标

```yaml
// hikaricp 是 Spring Boot2.x 选择的默认数据库连接池
    "hikaricp.connections", // 数据库连接池总连接数
    "hikaricp.connections.acquire", //单位时间内获取连接时间统计
    "hikaricp.connections.active", // 活跃连接数
    "hikaricp.connections.creation", // 创建连接时间
    "hikaricp.connections.idle", // 空闲连接
    "hikaricp.connections.max", // 最大连接数
    "hikaricp.connections.min", // 最小连接数
    "hikaricp.connections.pending", // 等待连接数
    "hikaricp.connections.timeout", // 获取连接超时总数
    "hikaricp.connections.usage", // 连接使用时间统计
    "http.server.requests", // 访问应用的总请求数
    "jdbc.connections.active", // 数据源分配的当前活跃连接数
    "jdbc.connections.idle", // 已和数据源建立的空闲连接数
    "jdbc.connections.max", // 同一时间最大可以建立的数据库连接数
    "jdbc.connections.min", // 数据库连接池中最小空闲连接数
    "jvm.buffer.count", // Java虚拟机缓冲区的个数估计
    "jvm.buffer.memory.used", // Java虚拟机用于这个缓冲池的内存的估计
    "jvm.buffer.total.capacity", // Java虚拟机中缓冲区总容量估计
    "jvm.classes.loaded", // 当前装入Java虚拟机中的类的数量
    "jvm.classes.unloaded", // 自虚拟机启动卸载的类总数
    "jvm.gc.live.data.size", // gc后老年代的大小
    "jvm.gc.max.data.size", // 老年代的最大内存
    "jvm.gc.memory.allocated", // 年轻代在一次gc后到下一次gc前过程中增长的内存大小
    "jvm.gc.memory.promoted", // 老年代有效gc次数
    "jvm.gc.pause", // gc暂停所花费的时间
    "jvm.memory.committed", // 提交给Java虚拟机使用的内存量(以字节为单位)
    "jvm.memory.max", // 可以用于jvm内存管理的最大内存数量(以字节为单位)
    "jvm.memory.used", // jvm已使用的内存数量
    "jvm.threads.daemon", // 当前活动守护进程线程的数量
    "jvm.threads.live", // 当前活动线程的数量，包括守护线程和非守护线程
    "jvm.threads.peak", // 自Java虚拟机启动后的最大活跃线程数
    "jvm.threads.states", // 当前处于BLOCKED状态的线程数
    "logback.events", // 记录到日志中的事件的数量
    "process.cpu.usage", // Java虚拟机进程的“最近cpu使用量”
    "process.files.max", // 最大文件描述符数量
    "process.files.open", // 当前打开的文件描述符数量
    "process.start.time", // 进程启动时间
    "process.uptime", // Java虚拟机的正常运行时间
    "spring.data.repository.invocations", // Spring中DAO访问数据库统计信息
    "system.cpu.count", // Java虚拟机可用的cpu数量
    "system.cpu.usage", // 整个系统的“最近的cpu使用量”
    "system.load.average.1m", // 一段时间内的CPU负载
    "tomcat.sessions.active.current", // tomcat服务器当前活跃会话数
    "tomcat.sessions.active.max", // tomcat最大会话数
    "tomcat.sessions.alive.max", // tomcat会话最大存活时间
    "tomcat.sessions.created", // tomcat会话已创建数量
    "tomcat.sessions.expired", // tomcat会话已创建数量
    "tomcat.sessions.rejected" // tomcat会话拒绝数量
```



配置触发器

```yaml
groups:
- name: SpringBoot
  rules:
  - alert: 堆内存使用超过50%
    expr: jvm_memory_used_bytes{ job="springboot", area="heap"} / jvm_memory_max_bytes * 100 > 50
    for: 1m
    labels:
      severity: warning
    annotations:
      summary: "实例{{ $labels.instance }}堆内存使用超过50%"
      description: "堆内存使用超过50%，当前值为：{{ $value }}"
   - alert: 堆内存使用超过80%
    expr: jvm_memory_used_bytes{ job="springboot", area="heap"} / jvm_memory_max_bytes * 100 > 80
    for: 1m
    labels:
      severity: warning
    annotations:
      summary: "实例{{ $labels.instance }}堆内存使用超过80%"
      description: "堆内存使用超过80%，当前值为：{{ $value }}"     
```



登录grafana，导入10286模板