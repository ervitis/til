# Logging

## Log4j

Setup example for `log4j2.xml` for `my.package`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration status="warn" packages="org.apache.logging.log4j.core,io.sentry.log4j2">
    <appenders>
        <Console name="Console" target="SYSTEM_OUT">
            <LogstashLayout dateTimeFormatPattern="yyyy-MM-dd'T'HH:mm:ss,SSS" prettyPrintEnabled="false" stackTraceEnabled="true"/>
        </Console>

        <Sentry name="Sentry" />
    </appenders>

    <loggers>
        <root level="${env:SERVICE_LOG_LEVEL:-INFO}">
            <appender-ref ref="Console" />
            <appender-ref ref="Sentry" level="ERROR" />
        </root>

        <logger name="my.package" level="${SERVICE_LOG_LEVEL:-INFO}"/>
        <logger name="org.apache.kafka" level="INFO"/>
        <logger name="org.mongodb" level="WARN"/>
        <logger name="org.springframework" level="WARN"/>
        <logger name="org.hibernate" level="WARN"/>
        <logger name="io.undertow" level="WARN"/>
        <logger name="com.zaxxer" level="WARN"/>
    </loggers>
</configuration>
```

The `LogstashLayout` is a plugin to inject logs inside logstash using JSON format. To use it it needs to add the following dependency

```xml
<dependency>
    <groupId>com.vlkan.log4j2</groupId>
    <artifactId>log4j2-logstash-layout</artifactId>
    <version>0.18</version>
</dependency>
```