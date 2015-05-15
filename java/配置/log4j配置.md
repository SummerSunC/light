# log4j配置

tags: java

---

``` properties
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <appender
        name="STDOUT"
        class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>
    <appender
        name="LOGFILE"
        class="ch.qos.logback.core.rolling.RollingFileAppender">
        <File>log/fxt-admin.log</File>
        <rollingPolicy
            class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <FileNamePattern>log/%d{yyyy-MM-dd}_fxt-admin.log.zip</FileNamePattern>
        </rollingPolicy>
        <layout class="ch.qos.logback.classic.PatternLayout">
            <pattern>%date [%thread] %-5level %logger{80} - %msg%n</pattern>
        </layout>
    </appender>
    <root level="info">
        <!--控制台输出，大量io ，开发环境下配置，生产环境去掉-->
        <appender-ref ref="STDOUT" />
        <appender-ref ref="LOGFILE" />
    </root>
    <logger name="com.vcooline.fxt">
        <level value="DEBUG" />
    </logger>
</configuration>

```




