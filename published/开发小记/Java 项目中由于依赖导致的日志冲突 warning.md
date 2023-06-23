---
description: "本文记录SLF4J 在 classpath 上存在多个绑定发出警告的问题。"
time: 2022-04-19
heroImage: ""
tags: []
---

在 Gradle 项目中安排了两个子模块，一个负责提供核心功能包以及 CLI API，另一个 Spring Boot 负责提供作为 REST 服务器为前端提供服务。在运行时发现警告：
`org.apache.logging.log4j.LoggingException: log4j-slf4j-impl cannot be present with log4j-to-slf4j`，在查阅[ slf4j 文档 ](https://www.slf4j.org/codes.html#multiple_bindings)后大意是说 SLF4J API 被设计为一次绑定一个且只绑定一个日志框架。如果 class path 上存在多个绑定，SLF4J 将发出警告，列出这些绑定的位置。  
思考后应该是 Spring Boot **以核心包模块作为依赖**，而这个包里使用了 `log4j2` 作为日志实现以便于在 CLI API 使用时输出日志，而 Spring Boot 本身也提供日志实现，两个日志实现发生冲突，则报出上述警告。  

> 需要注意的是，SLF4J 发出的警告仅仅是一个警告。即使存在多个绑定，SLF4J 也将选择一个日志框架/实现并与其绑定。SLF4J 选择绑定的方式是由 JVM 决定的，出于所有实际目的，应该认为是随机的。从版本1.6.6开始，SLF4J 将命名它实际绑定到的框架/实现类。

我想要达到的效果是：
* 在 CLI API 上使用对应模块的日志输出，也就是log4j-slf4j-impl，按照配置将日志输出到文件中
* 在服务器中按照 Spring Boot 的默认日志配置，将日志打印到控制台

那么我只需要在 Spring Boot 模块中排除核心模块的日志框架即可，在 `build.gradle` 中加入
```groovy
configurations {
    ...
    all {
        // avoid two bindings of slf4j
        exclude group: 'org.apache.logging.log4j', module: 'log4j-slf4j-impl'
    }
}
```
即可实现不同的API方式使用不同的日志输出方法；  

如果要在服务器提供服务过程中同样使用核心包的日志输出配置，那么就可以改为
```groovy
exclude group: 'org.springframework.boot', module: 'spring-boot-starter-logging'
```
这样在服务器运行过程中生成的日志也会和 CLI 一样被记录在文件中。

## 参考
* https://stackoverflow.com/questions/59629214/caused-by-org-apache-logging-log4j-loggingexception-log4j-slf4j-impl-cannot-be
* https://blog.csdn.net/supingemail/article/details/112944282
* https://www.slf4j.org/codes.html#multiple_bindings