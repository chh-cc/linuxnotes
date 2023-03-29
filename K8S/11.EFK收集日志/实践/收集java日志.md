在SpringBoot项目中，我们首先配置Logback，确定日志文件的位置

```java
<appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
	<file>${user.dir}/logs/order.log</file>
	<rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
	 <fileNamePattern>${user.dir}/logs/order.%d{yyyy-MM-dd}.log</fileNamePattern>
	 <maxHistory>7</maxHistory>
	</rollingPolicy>
	<encoder>
	 <pattern></pattern>
	</encoder>
</appender>
```

filebeat配置

```yaml
filebeat.inputs:
- type: log
  enable: true
  paths:
    - xxx.log
  #json.keys_under_root: true
  #json.overwrite_keys: true
  tags: ["xxx"]
  fields:
    app: xxx
    
output.logstash:
  hosts: ["10.10.0.100:5044"]
```

