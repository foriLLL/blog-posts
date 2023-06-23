---
description: "在对日志 logger 的学习使用中想到，为什么每次构造 logger 实例都需要传入类的类型。本文记录网上搜索的一些答案以及一些自己简单的思考。"
time: 2022-02-15 14:35:21+08:00
heroImage: ""
tags: []
---
在对日志 logger 的学习使用中想到，为什么每次构造 logger 实例都需要传入类的类型。比如
```java
 Logger logger = Logger.getLogger("com.foo.Bar");
 ```
就在思考为什么一定要传入一个类？为什么不使用一个单例模式，声明一个 public static 的 logger，只需要在其他地方引入然后直接使用就行，在实际试验后发现这样似乎也没有什么显式的问题，所有日志都能够正常记录，也不会显示出这个 logger 究竟是传入了什么类，然后就在网上搜索相关问题。

```java
//private final static Logger logger = LoggerFactory.getLogger(javaParserUtils.class);
private final static Logger logger = LoggerFactory.getLogger("sdafas");
```
得到的答案大概就是这个可以作为一个习惯做法，log4j 和其他日志框架的约定是为每个类定义一个静态日志 logger。  
1. 你完全可以不使用类作为 logger 的名称，传入任何一个字符串都可以。
2. 传入一个类的好处可以总结为：
   * 使用方便。在复杂的 Java EE 应用程序中，您不必担心 logger 名称重复。如果其他人也使用你的 logger 名称，你可能有一个不仅包括你的类的输出的日志文件；
   * 易于检查 logging 类，因为 **logger名称将显示在日志文件中**。您可以快速导航到特定的课程;
   * 当你分发你的类时，人们可能希望从你的 class 直接重定向到一个特定的文件。在这种情况下，如果您使用特殊的 logger 名称，我们可能需要检查源代码，如果源代码不可用，则无法执行此操作。

## 参考
* https://stackoverflow.com/questions/14596690/why-log4js-logger-getlogger-need-pass-a-class-type