title: SpringBoot 使用org.slf4j.Logger 记录日志
author: Elijah Zheng
tags:
  - springboot
  - java
categories: []
date: 2020-09-30 18:13:00
---
  我们在本地运行应用程序时，可以通过控制台查看程序运行情况。但我们把应用部署到生产服务器时，或者对应用测试时需要查看程序运行的情况，我们可以把运行的情况记录到日志中，这样我们就可以通过日志查询程序运行的好坏情况，以及方便后续程序出错时找出错误点及时修复。
  
  以下是在``springboot``通过``org.slf4j.Logger``记录日志,``slf4j``是``springboot``默认自带的日志框架，所以不需要在``pom``文件中引入。
  
  # 定义配置
  首先需要定义日志的配置文件，配置日志的等级、输出格式、输出位置、保存时间、保存大小等。
   
   配置文件可以定义在``项目路径/src/main/resources/logback.xml``
   
``` java
<?xml version="1.0" encoding="UTF-8" ?>
<!-- 日志级别从低到高分为TRACE < DEBUG < INFO < WARN < ERROR < FATAL，如果设置为WARN，则低于WARN的信息都不会输出 -->
<!-- scan:当此属性设置为true时，配置文档如果发生改变，将会被重新加载，默认值为true -->
<!-- scanPeriod:设置监测配置文档是否有修改的时间间隔，如果没有给出时间单位，默认单位是毫秒。
                 当scan为true时，此属性生效。默认的时间间隔为1分钟。 -->
<!-- debug:当此属性设置为true时，将打印出logback内部日志信息，实时查看logback运行状态。默认值为false。 -->
<configuration debug="false" scan="true" scanPeriod="30 seconds">
    <!--    输出到控制台的配置-->
    <appender name="console_logger" class="ch.qos.logback.core.ConsoleAppender">
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <pattern>%date [%level] [%thread] %logger{80} [%file : %line] %msg%n</pattern>
        </encoder>

        <!--日志不会记录比INFO小的日志，TRACE、DEBUG、INFO、WARN、ERROR-->
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <level>INFO</level>
        </filter>
    </appender>

    <!--输出到日志文件-->
    <appender name="project_logger" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <encoder>
            <pattern>%date [%level] [%thread] %logger{26} [%file : %line] %msg%n</pattern>
        </encoder>

        <!--日志不会记录比INFO小的日志，TRACE、DEBUG、INFO、WARN、ERROR-->
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <level>INFO</level>
        </filter>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>project-logs/project-log.%d{yyyy-MM-dd}.log</fileNamePattern>
            <maxHistory>30</maxHistory>
            <totalSizeCap>1GB</totalSizeCap>
        </rollingPolicy>
    </appender>

    <root level="INFO">
        <appender-ref ref="console_logger"/>
        <appender-ref ref="project_logger"/>
    </root>
</configuration>
```

# 开始使用
定义好配置之后，我们就可以使用了，通过引入和在需要记录的地方使用即可。

需要注意的地方时，我们在上面配置信息里，定义了记录日志的等级为``<level>INFO</level>``，意思是低于``INFO``等级的记录我们不会记录。
  
``` java
// 引入
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

// 定义全局日志变量
private static Logger logger = LoggerFactory.getLogger(ILogService.class);

// 将信息记录到日志中
String info = "记录日志信息";
logger.info(info);
```
然后``info``的信息，可以在日志文件``.log``中查看了。