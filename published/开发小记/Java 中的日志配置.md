---
description: "有关 Java 中的日志框架和相关配置，需要初步了解的内容。"
time: 2021-12-19
heroImage: ""
tags: []
---

## 什么是日志

什么是日志？日志就是 Logging，它的目的是为了取代 `System.out.println()`。  
输出日志，而不是用 `System.out.println()`，好处众多
1. 可以设置输出样式，避免自己每次都写 `"ERROR: " + var`；
2. 可以设置输出级别，禁止某些级别输出。例如，只输出错误日志；
3. 可以被重定向到文件，这样可以在程序运行结束后查看日志；
4. 可以按包名控制日志级别，只输出某些包打的日志；
5. ...

## 日志接口和日志实现

首先需要区分的概念是 **日志接口** 和 **日志实现**。

### 日志接口（slf4j）

slf4j 是对所有日志框架制定的一种规范、标准、接口，并不是一个框架的具体的实现，因为接口并不能独立使用，需要和具体的日志框架实现配合使用（如 log4j、 logback）。

### 日志实现（log4j、logback、log4j2）

log4j 是 apache 实现的一个开源日志组件。  
logback 同样是由 log4j 的作者设计完成的，拥有更好的特性，用来取代 log4j 的一个日志框架，是 slf4j 的原生实现。  
log4j2 是 log4j 1.x 和 logback 的改进版，据说采用了一些新技术（无锁异步、等等），使得日志的吞吐量、性能比 log4j 1.x 提高 10 倍，并解决了一些死锁的 bug，而且配置更加简单灵活，[官网地址](http://logging.apache.org/log4j/2.x/manual/configuration.html)。

## 为什么需要日志接口，而不直接使用具体的实现

接口用于定制规范，可以有多个实现，使用时是面向接口的（导入的包都是 slf4j 的包而不是具体某个日志框架中的包），即直接和接口交互，不直接使用实现，所以可以任意的更换实现而不用更改代码中的日志相关代码。

比如：slf4j 定义了一套日志接口，项目中使用的日志框架是 logback，**开发中调用的所有接口都是 slf4j 的，不直接使用 logback**，调用是自己的工程调用 slf4j 的接口，slf4j 的接口去调用 logback 的实现，可以看到整个过程应用程序并没有直接使用 logback，当项目需要更换更加优秀的日志框架时（如 log4j2）只需要引入 log4j2 的 jar 和 log4j2 对应的配置文件即可，完全不用更改 Java 代码中的日志相关的代码 `logger.info("xxx")`，也不用修改日志相关的类的导入的包（`import org.slf4j.Logger; import org.slf4j.LoggerFactory;`）

***使用日志接口便于更换为其他日志框架。***

log4j、logback、log4j2 都是一种日志具体实现框架，所以既可以单独使用也可以结合 slf4j 一起搭配使用）

## 在项目中配置日志

我们在项目中使用 slf4j 作为日志接口，log4j2 作为日志实现。  
首先需要加入依赖：

```groovy
dependencies {
    //log4j核心
    implementation group: 'org.apache.logging.log4j', name: 'log4j-api', version: '2.17.0'
    implementation group: 'org.apache.logging.log4j', name: 'log4j-core', version: '2.17.0'
    //slf4j-api
    implementation 'org.slf4j:slf4j-api:1.7.25'
    //桥接包log4j-slf4j-impl起到适配的作用，因为市面上的日志实现互不兼容，日志框架slf4j要想适用于日志实现log4j2，就需要使用桥接包
    implementation 'org.apache.logging.log4j:log4j-slf4j-impl:2.9.1'
}
```

在代码中我们只需要用 slf4j 的接口便可打印日志。

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

class Main {
    final Logger logger = LoggerFactory.getLogger(getClass());
    public void logTest(){
        logger.info("xxxxxxxx");
    }
}
```

此时我们还没有配置日志的配置文件，具体关于 log4j2 的配置文件可以参考[这里](https://blog.csdn.net/zheng0518/article/details/69558893)。  

### 关于配置文件的名称以及在项目中的存放位置

log4j 2.x 版本不再支持像 1.x 中的 `.properties` 后缀的文件配置方式，2.x 版本配置文件后缀名只能为 ".xml", ".json" 或者 ".jsn"。  

系统选择配置文件的优先级（从先到后）如下：
1. `.classpath` 下的名为 `log4j2-test.json` 或者 `log4j2-test.jsn` 的文件；
2. `.classpath` 下的名为 `log4j2-test.xml` 的文件；
3. `.classpath` 下名为 `log4j2.json` 或者 `log4j2.jsn` 的文件；
4. `.classpath` 下名为 `log4j2.xml` 的文件。

我们一般默认使用 `log4j2.xml` 进行命名。如果本地要测试，可以把 `log4j2-test.xml` 放到 `classpath`，而正式环境使用 `log4j2.xml`，则在打包部署的时候不要打包 `log4j2-test.xml` 即可。

可在 resources 下加入配置文件，以 XML 为例：

```xml
<!-- log4j2.xml -->
<?xml version="1.0" encoding="UTF-8"?>
<configuration status="OFF">
    <appenders>
        <Console name="Console" target="SYSTEM_OUT">
            <!--只接受程序中DEBUG级别的日志进行处理-->
            <ThresholdFilter level="DEBUG" onMatch="ACCEPT" onMismatch="DENY"/>
            <PatternLayout pattern="[%d{HH:mm:ss.SSS}] %-5level %class{36} %L %M - %msg%xEx%n"/>
        </Console>
        <!--处理DEBUG级别的日志，并把该日志放到logs/debug.log文件中-->
        <!--打印出DEBUG级别日志，每次大小超过size，则这size大小的日志会自动存入按年份-月份建立的文件夹下面并进行压缩，作为存档-->
        <RollingFile name="RollingFileDebug" fileName="./logs/debug.log"
                     filePattern="logs/$${date:yyyy-MM}/debug-%d{yyyy-MM-dd}-%i.log.gz">
            <Filters>
                <!--只接受DEBUG级别的日志，其余的全部拒绝处理-->
                <ThresholdFilter level="DEBUG"/>
                <ThresholdFilter level="INFO" onMatch="DENY" onMismatch="NEUTRAL"/>
            </Filters>
            <!--输出日志的格式-->
            <PatternLayout
                    pattern="[%d{yyyy-MM-dd HH:mm:ss}] %-5level %class{36} %L %M - %msg%xEx%n"/>
            <Policies>
                <SizeBasedTriggeringPolicy size="500 MB"/>
                <TimeBasedTriggeringPolicy/>
            </Policies>
        </RollingFile>
        <!--处理INFO级别的日志，并把该日志放到logs/info.log文件中-->
        <RollingFile name="RollingFileInfo" fileName="./logs/info.log"
                     filePattern="logs/$${date:yyyy-MM}/info-%d{yyyy-MM-dd}-%i.log.gz">
            <Filters>
                <!--只接受INFO级别的日志，其余的全部拒绝处理-->
                <ThresholdFilter level="INFO"/>
                <ThresholdFilter level="WARN" onMatch="DENY" onMismatch="NEUTRAL"/>
            </Filters>
            <PatternLayout
                    pattern="[%d{yyyy-MM-dd HH:mm:ss}] %-5level %class{36} %L %M - %msg%xEx%n"/>
            <Policies>
                <SizeBasedTriggeringPolicy size="500 MB"/>
                <TimeBasedTriggeringPolicy/>
            </Policies>
        </RollingFile>
        <!--处理WARN级别的日志，并把该日志放到logs/warn.log文件中-->
        <RollingFile name="RollingFileWarn" fileName="./logs/warn.log"
                     filePattern="logs/$${date:yyyy-MM}/warn-%d{yyyy-MM-dd}-%i.log.gz">
            <Filters>
                <ThresholdFilter level="WARN"/>
                <ThresholdFilter level="ERROR" onMatch="DENY" onMismatch="NEUTRAL"/>
            </Filters>
            <PatternLayout
                    pattern="[%d{yyyy-MM-dd HH:mm:ss}] %-5level %class{36} %L %M - %msg%xEx%n"/>
            <Policies>
                <SizeBasedTriggeringPolicy size="500 MB"/>
                <TimeBasedTriggeringPolicy/>
            </Policies>
        </RollingFile>
        <!--处理error级别的日志，并把该日志放到logs/error.log文件中-->
        <RollingFile name="RollingFileError" fileName="./logs/error.log"
                     filePattern="logs/$${date:yyyy-MM}/error-%d{yyyy-MM-dd}-%i.log.gz">
            <ThresholdFilter level="ERROR"/>
            <PatternLayout
                    pattern="[%d{yyyy-MM-dd HH:mm:ss}] %-5level %class{36} %L %M - %msg%xEx%n"/>
            <Policies>
                <SizeBasedTriggeringPolicy size="500 MB"/>
                <TimeBasedTriggeringPolicy/>
            </Policies>
        </RollingFile>
        <!--druid的日志记录追加器-->
        <RollingFile name="druidSqlRollingFile" fileName="./logs/druid-sql.log"
                     filePattern="logs/$${date:yyyy-MM}/api-%d{yyyy-MM-dd}-%i.log.gz">
            <PatternLayout pattern="[%d{yyyy-MM-dd HH:mm:ss}] %-5level %L %M - %msg%xEx%n"/>
            <Policies>
                <SizeBasedTriggeringPolicy size="500 MB"/>
                <TimeBasedTriggeringPolicy/>
            </Policies>
        </RollingFile>
    </appenders>
    <!--     然后定义logger，只有定义了logger并引入的appender，appender才会生效 -->
    <loggers>
        <!--默认的root的logger-->
        <root level="DEBUG">
            <appender-ref ref="Console"/>
            <appender-ref ref="RollingFileInfo"/>
            <appender-ref ref="RollingFileWarn"/>
            <appender-ref ref="RollingFileError"/>
            <appender-ref ref="RollingFileDebug"/>
        </root>
        <!--额外配置的logger-->
        <!--记录druid-sql的记录-->
        <logger name="druid.sql.Statement" level="debug" additivity="false">
            <appender-ref ref="druidSqlRollingFile"/>
        </logger>
        <!--log4j2 自带过滤日志-->
        <Logger name="org.apache.catalina.startup.DigesterFactory" level="error"/>
        <Logger name="org.apache.catalina.util.LifecycleBase" level="error"/>
        <Logger name="org.apache.coyote.http11.Http11NioProtocol" level="warn"/>
        <logger name="org.apache.sshd.common.util.SecurityUtils" level="warn"/>
        <Logger name="org.apache.tomcat.util.net.NioSelectorPool" level="warn"/>
        <Logger name="org.crsh.plugin" level="warn"/>
        <logger name="org.crsh.ssh" level="warn"/>
        <Logger name="org.eclipse.jetty.util.component.AbstractLifeCycle" level="error"/>
        <Logger name="org.hibernate.validator.internal.util.Version" level="warn"/>
        <logger name="org.springframework.boot.actuate.autoconfigure.CrshAutoConfiguration" level="warn"/>
        <logger name="org.springframework.boot.actuate.endpoint.jmx" level="warn"/>
        <logger name="org.thymeleaf" level="warn"/>
    </loggers>
</configuration>
```

之后便可以愉快地使用日志了！

## 参考

* https://www.liaoxuefeng.com/wiki/1252599548343744/1264738568571776 （推荐阅读）
* https://blog.csdn.net/vbirdbest/article/details/71751835