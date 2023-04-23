步骤：

1. 引入maven依赖
2. application.properties配置
3. Prometheus配置采集springboot2.x应用提供的监控样本数据
4. grafana将Prometheus作为数据源进行可视化展示



引入maven依赖

修改pom.xml添加如下代码，引入spring boot actuator的相关依赖

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

application.properties配置

出于安全考虑，默认情况下除了/health和/info之外的所有actuator都是关闭的

```yaml
management.endpoints.web.exposure.include=prometheus,health
#management.endpoints.web.exposure.include=*
#management.endpoints.web.exposure.exclude=env,beans
management.metrics.tags.application=${spring.application.name}
```

prometheus配置

```yaml
- job_name: 'springboot-demo'
  metrics_path: '/actuator/prometheus'
  scrape_interval: 5s
  static_configs:
  - targets: ['xx.xx.xx.xx:8081']
```

监控指标

```yaml
process_files_max_files # 最大文件处理数量
tomcat_sessions_active_current_sessions # Tomcat 当前活跃 session 数量
tomcat_sessions_alive_max_seconds # Tomcat session 最大存活时间

jvm_buffer_total_capacity_bytes  # 预估的池中缓冲区的总容量
jvm_threads_daemon_threads # 当前守护进程的线程数量

tomcat_global_request_max_seconds{name="http-nio-8080"} # 全局最长一次请求的时间
tomcat_sessions_active_max_sessions # 最大活跃 session 数量
system_cpu_usage # CPU 利用率
jvm_buffer_memory_used_bytes # 预估 Java 虚机用于此缓冲池的内存
jvm_classes_loaded_classes # 当前在 Java 虚拟机中加载的类的数量

jvm_memory_committed_bytes #为 Java 虚拟机提交的内存量(以字节为单位)
jvm_threads_live_threads # 当前线程数，包括守护进程和非守护进程的线程
tomcat_threads_config_max_threads # 配置的 Tomcat 的最大线程数
tomcat_global_received_bytes_tota] # Tomcat 接收到的数据量
tomcat_global_sent_bytes_tota] # Tomcat 发送的数据量
tomcat_threads_current_threads # Tomcat 当前的线程数
tomcat_sessions_created_sessions_total # Tomcat 创建的 session 数
system_1oad_average_1m # 在一段时间内，排队到可用处理器的可运行实体数量和可用处理器上运行的可运行实体数量的总和的平均值

tomcat_sessions_expired_sessions_tota] # 过期的 session 数量
jvm_buffer_count_buffers # 预估的池中的缓冲区数量
jivm_memory_used_bytes # JVM 内存使用展
process_uptime_seconds # Java 拟机的正常运行时间
jvm_gc_memory_allocated_bytes_total # 增加一个 GC 到下一个 GC 之后年轻代内存池的大小增加
jvm_gc_pause_seconds_count # GC暂停耗时数量和总时间
jvm_gc_pause_seconds_sum
jvm_gc_pause_seconds_max

tomcat_sessions_rejected_sessions_tota] # 被拒绝的 session 总数
jvm_gc_live_data_size_bytes # Fu11 GC 后的老年代内存池的大小
tomcat_threads_busy_threads # Tomcat 繁忙线程数

jvm_threads_peak_threads # 自Java 虚拟机启动或峰值重置以来的最高活动线程数
jvm_threads_states_threads # 当前具有 NEW 状态的线程数
jvm_gc_max_data_size_bytes # jvm_gc内存池的最大大小
http_server_requests_seconds_count #某个接口的请求数量和请求总时间
http_server_requests_seconds_sum
http_server_requests_seconds_max
jvm_gc_memory_promoted_bytes_total # GC之前到GC之后的老年代内存池大小的正增加计数

# 日志按级别计数
logback_events_total{application="prometheus-demo",level="info"} 7.0
logback_events_total{application="prometheus-demo",level="trace",} 0.0
logback_events_total{application="prometheus-demo",level="warn",} 0.0
logback_events_totallapplication="prometheus-demo",level="debug",} 0.0
logback_events_totallapplication="prometheus-demo",level="error",} 0.0

process_start_time_seconds # 启动时间
process_files_open_files # 打开文件述符的数量
tomcat_global_error_total # 异常数量
jvm_memory_max_bytes # 可用于内存管理的最大内存量(以字节为单位)

process_cpu_usage # 最近的 CPU 利用率
jvm_classes_unloaded_classes_total # 自 Java 虚拟机开始执行以来卸的类总数
system_cpu_count # CPU核数
tomcat_globa]_request_seconds_count # 全局请求总数和总耗时
tomcat_global_request_seconds_sum
```

