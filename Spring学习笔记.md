## 什么是 Spring

Spring 是一个开源框架，它由 Rod Johnson 创建。它是为了解决企业应用开发的复杂性而创建的。

- **目的**：解决企业应用开发的复杂性
- **功能**：使用基本的 JavaBean 代替 EJB，并提供了更多的企业应用功能
- **范围**：任何 Java 应用  
  它是一个*容器框架*，用来装 javabean（java 对象），中间层框架（万能胶）可以起一个连接作用，比如说把 Struts 和 hibernate 粘合在一起运用。简单来说，Spring 是一个轻量级的控制反转(IoC)和面向切面(AOP)的容器框架。

## 为什么要用 Spring

## IoC 和 AOP
IoC 和 AOP 是 Spring 的两大核心机制。
### IoC
Spring 的核心就是提供了一个 **IoC 容器**，它可以管理所有轻量级的 JavaBean 组件，提供的底层服务包括组件的生命周期管理、配置和组装服务、AOP 支持，以及建立在 AOP 基础上的声明式事务服务等。

> 什么是容器？容器是一种为某种特定组件的运行提供必要支持的一个软件环境。例如， Tomcat 就是一个 Servlet 容器，它可以为 Servlet 的运行提供运行环境。类似 Docker 这样的软件也是一个容器，它提供了必要的 Linux 环境以便运行一个特定的 Linux 进程。  
> 通常来说，使用容器运行组件，除了提供一个组件运行环境之外，容器还**提供了许多底层服务**。例如， Servlet 容器底层实现了 TCP 连接，解析 HTTP 协议等非常复杂的服务，如果没有容器来提供这些服务，我们就无法编写像 Servlet 这样代码简单，功能强大的组件。早期的 JavaEE 服务器提供的 EJB 容器最重要的功能就是通过声明式事务服务，使得 EJB 组件的开发人员不必自己编写冗长的事务处理代码，所以极大地简化了事务处理。

IoC(Inversion of Control)就是控制反转，不是什么技术，而是一种**设计思想**。在 Java 开发中，<u>Ioc 意味着将你设计好的对象交给容器控制，而不是传统的在你的对象内部直接控制</u>。

正向控制  
<img src="https://img.foril.fun/%E6%AD%A3%E5%90%91%E6%8E%A7%E5%88%B6.jpg" width=400 style="margin:0 auto"/>

控制反转  
<img src="https://img.foril.fun/%E6%8E%A7%E5%88%B6%E5%8F%8D%E8%BD%AC.jpg" width=400 style="margin:0 auto"/>

举例来说，假如我们想生成一个实体类，在传统的 Java 开发中，我需要在程序中`new`一个对象出来对他进行操作，而有了 IoC 容器后，生成对象的操作只需要在容器中完成，在代码中只需要拿到容器中的对象即可，这个过程有点类似连接池中的连接对象，只需要从池中拿到连接对象即可。在 Spring 中，我们可以通过 xml 在容器中创建对象，然后在代码中引入容器并从中拿到对象即可。

#### IoC 的好处

传统应用程序都是由我们在类内部主动创建依赖对象，从而导致类与类之间高耦合，难于测试；有了 IoC 容器后，把创建和查找依赖对象的控制权交给了容器，由容器进行注入组合对象，所以对象与对象之间是**松散耦合**，这样也方便测试，利于功能复用，更重要的是使得程序的整个体系结构变得非常灵活。[推荐阅读](https://www.liaoxuefeng.com/wiki/1252599548343744/1282381977747489)  
举个例子，在很多情况下我们会想要用到单例模式，比如一些 Service 对象，其实我们只需要在全局有一个实例而不需要在到处 new 一个出来，利用 IoC 容器可以让我们很灵活的创建这样的对象。

#### setup
首先配置`pom.xml`引入spring-context。
```xml
<properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
    <maven.compiler.source>11</maven.compiler.source>
    <maven.compiler.target>11</maven.compiler.target>
    <java.version>11</java.version>

    <spring.version>5.2.3.RELEASE</spring.version>
</properties>

<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>${spring.version}</version>
    </dependency>
</dependencies>
```
> spring 中的IoC也是容器，那么他需要一个运行的上下文，spring-context就负责提供这个上下文。[更多关于](https://juejin.cn/post/6844903923703087111)

之后在maven标准目录下`src/resources`中添加`application.xml`（），创建对象。
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="userService" class="com.itranswarp.learnjava.service.UserService">
        <property name="mailService" ref="mailService" />
    </bean>

    <bean id="mailService" class="com.itranswarp.learnjava.service.MailService" />
</beans>
```
**注意：**
- 每个`<bean ...>`都有一个`id`标识，相当于Bean的唯一ID；
- `property`即是给成员变量赋值（set方法）；
- 可以通过`constructor-arg`来调用构造函数；
- `property`中`ref`可以引用容器创建的其他对象作为属性
- Bean的顺序不重要，Spring根据依赖关系会自动正确初始化；

如果注入的不是Bean，而是boolean、int、String这样的数据类型，则通过value注入，例如，创建一个HikariDataSource：
```xml
<bean id="dataSource" class="com.zaxxer.hikari.HikariDataSource">
    <property name="jdbcUrl" value="jdbc:mysql://localhost:3306/test" />
    <property name="username" value="root" />
    <property name="password" value="password" />
    <property name="maximumPoolSize" value="10" />
    <property name="autoCommit" value="true" />
</bean>
```
接下来我们需要创建一个Spring的IoC容器实例，然后加载配置文件，让Spring容器为我们创建并装配好配置文件中指定的所有Bean，这只需要一行代码：
```java
ApplicationContext context = new ClassPathXmlApplicationContext("application.xml");

```
最后就可以在程序中取出在容器中装配好的Bean：
```java
// 获取Bean:
UserService userService = context.getBean(UserService.class);
// 正常调用:
User user = userService.login("bob@example.com", "password");
```
完整main:
```java
public class Main {
    public static void main(String[] args) {
        ApplicationContext context = new ClassPathXmlApplicationContext("application.xml");
        UserService userService = context.getBean(UserService.class);
        User user = userService.login("bob@example.com", "password");
        System.out.println(user.getName());
    }
}
```
从上面代码可以看出 Spring 容器就是 `ApplicationContext`，它是一个接口，有很多实现类，这里我们选择`ClassPathXmlApplicationContext`，表示它会自动从 classpath 中查找指定的 XML 配置文件。

获得了`ApplicationContext`的实例，就获得了IoC容器的引用。从`ApplicationContext`中我们可以根据Bean的 ID 获取Bean（需要加入类型强转），但更多的时候我们根据Bean的类型获取Bean的引用（当Bean中存在多个同一个类时会抛出异常）。

#### Bean的Scope
scope 表示 bean 的作用域，有4种类型︰
- singleton，单例，表示通过Spring容器获取的对象是唯一的，默认值。
- prototype，原型，表示通过Spring容器获取的对象是不同的。
- request，请求，表示在一次HTTP请求内有效。
- session，会话，表示在一个用户会话内有效。

requset，session适用于Web项目。  
singleton模式下，只要加载loC容器，无论是否从loC中取出 bean，配置文件中的 bean都会被创建。  
prototype模式下，如果不从 loC中取 bean，则不创建对象，取一次 bean，就会创建一个对象。

#### Bean的继承
Spring继承不同于Java中的继承，区别: Java中的继承是针对于类的，Spring 的继承是针对于对象(bean)。  
Spring 的继承中，子bean可以继承父bean中的所有成员变量的值。  
通过设置bean标签的`parent`属性建立继承关系，同时子bean可以覆盖父bean的属性值。  
Spring的继承是**针对对象**的，所以子bean和父bean并不需要属于同一个数据类型，**只要其成员变量列表一致即可**（如果它看起来像鸭子、游泳像鸭子、叫声像鸭子，那么它可能就是只鸭子）。

#### Bean的依赖
与继承类似，依赖也是描述bean与bean之间的另一种关系。利用`depends-on`属性。

#### Spring读取外部资源
可以在`xml`资源文件中加入如`context:property-placeholder`标签，利用EL表达式`${}`读取`properties`文件中的数据。
#### Spring p命名空间
p命名空间可以⽤来代替properties，简化bean的配置。
```xml
<bean id="student" class="edu.ssdut.springTest.entity.Student" p:id="1" p:name="foril">
```

#### 使用注解配置
使用XML配置的优点是所有的Bean都能一目了然地列出来，并通过配置注入能直观地看到每个Bean的依赖。它的缺点是写起来非常繁琐，每增加一个组件，就必须把新的Bean配置到XML中。  
我们可以使用注解来配置IoC容器，完全不需要XML，让Spring自动扫描Bean并组装它们。
- @Component

```java
@Component
public class MailService {
    ...
}
```
这个`@Component`注解就相当于定义了一个Bean，它有一个可选的名称，默认是mailService，即小写开头的类名。

- @Autowired  

使用@Autowired就相当于把指定类型的Bean注入到指定的字段中。和XML配置相比，@Autowired大幅简化了注入，因为它不但可以写在set()方法上，还可以直接写在字段上，甚至可以写在构造方法中：
```java
@Component
public class UserService {
    @Autowired // 字段上
    MailService mailService;

    ...
}
// 或者
@Component
public class UserService {
    MailService mailService;

    // 构造方法上
    public UserService(@Autowired MailService mailService) {
        this.mailService = mailService;
    }
    ...
}
```
- @Configuration @ComponentScan

编写一个`AppConfig`类启动容器：
```java
@Configuration
@ComponentScan
public class AppConfig {
    public static void main(String[] args) {
        ApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);
        UserService userService = context.getBean(UserService.class);
        User user = userService.login("bob@example.com", "password");
        System.out.println(user.getName());
    }
}
```
使用的实现类是`AnnotationConfigApplicationContext`，必须传入一个标注了`@Configuration`的类名。  
AppConfig还标注了@ComponentScan，它告诉容器，自动搜索当前类所在的包以及子包，把所有标注为@Component的Bean自动创建出来，并根据@Autowired进行装配。

- @Scope

`@Scope`注解修改Bean的Scope。
```java
@Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE) // @Scope("prototype")
```
- @Order

List被注入多个对象时，Spring会**自动**把所有对应类型的Bean装配为一个List注入进来，这样一来，我们每新增一个对应类型的类，就自动被Spring装配到List中了。  
因为Spring是通过扫描classpath获取到所有的Bean，而List是有序的，要指定List中Bean的顺序，可以加上@Order注解：

- @Autowired(required = false)
  
可选注入，这个参数告诉Spring容器，如果找到一个对应类型的Bean就注入，如果找不到就忽略。

更多注解[参考阅读](https://www.liaoxuefeng.com/wiki/1252599548343744/1308043627200545)

#### Spring 工厂方法
Spring ⼯⼚⽅法 IoC 通过⼯⼚模式创建 bean 有两种⽅式：
- 静态⼯⼚⽅法
- 实例⼯⼚⽅法

区别在于静态⼯⼚类不需要实例化，实例⼯⼚类需要实例化。  
spring.xml 中 class + factory-method 的形式是直接调⽤类中的⼯⼚⽅法； 
```xml
<bean id="car" class="edu.ssdut.factory.StaticCarFactory" factory-method="getCar">
    <constructor-arg value="2"></constructor-arg>
</bean>
<!-- 得到的直接是bean -->
``` 
spring.xml 中 factory-bean + factory-method 的形式则是调⽤⼯⼚ bean 中的⼯⼚⽅法，就必须先创建⼯⼚ bean
```xml
<bean id="carFactory" class="edu.ssdut.factory.InstanceCarFactory"></bean>
<!-- 需要在IoC中创建两个bean，先创建工厂的实例，在创建car -->
<bean id="car2" factory-bean="carFactory" factory-method="getCar">
    <constructor-arg value="1"></constructor-arg>
</bean>
```

#### 自动装载 autowire
⾃动装载是 Spring 提供的⼀种更加简便的⽅式来完成 DI，不需要⼿动配置property，IoC 容器会⾃动选择 bean 完成注⼊。  
⾃动装载有两种⽅式：
- byName，通过属性名完成⾃动装载。
- byType，通过属性对应的数据类型完成⾃动装载。
```xml
<!-- 手动装载 -->
<bean id="car" class="edu.ssdut.entity.Car" p:id="1" p:name="宝马"></bean>
<bean id="person" class="edu.ssdut.entity.Person" p:id="1" p:name="张三" >
    <property name="car" ref="car"></property>
</bean>
```

```xml
<!--自动装载，byName通过名字去找id为car的对象 -->
<bean id="person" class="edu.ssdut.entity.Person" p:id="1" p:name="张三" autowire="byName" ></bean>
```

### AOP
AOP（Aspect Oriented Programming），即面向切面编程，是一种新的编程方式，它和OOP不同，OOP把系统看作多个对象的交互，AOP把系统分解为不同的关注点，或者称之为切面（Aspect）。

把业务中大量重复的业务代码，比如每个业务中的权限检查，提取出来。

底层原理：Spring的AOP实现就是基于JVM的动态代理。  
```java
public class RealSubject implements Subject{

    @Override
    public int add(int a, int b) {
        return a+b;
    }
}

// 动态代理
/**
 * 这个类实际上负责生成代理对象（代理的爹）
 */
public class MyInvocationHandler implements InvocationHandler {
    private Object obj; // 存放委托对象

    public Object bind(Object object){  // 返回委托对象
        this.obj = object;
        return Proxy.newProxyInstance(obj.getClass().getClassLoader(), obj.getClass().getInterfaces(),this);
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable { // 实现InvocationHandler接口
        // 实现对于方法的调用，同时可以完成对方法的增强
        // ---在这里可以完成对方法调用前的日志打印、安全检查等操作
        System.out.println("方法名是："+method.getName());
        // ---
        Object result = method.invoke(this.obj, args);  // 利用发射对实际对象的调用
        // ---在这里可以完成对方法调用后的增强
        System.out.println(method.getName()+"的结果是："+ result);
        // ---
        return result;
    }
}

// 客户端代码
public static void main(String[] args) {
        Subject realSubject = new RealSubject();    // 实例化真实对象
        MyInvocationHandler handler = new MyInvocationHandler();
        Subject proxy = (Subject)handler.bind(realSubject); // 传入实际对象，得到代理对象
        System.out.println(proxy.add(1, 1));
    }

// 实际输出
方法名是：add
add的结果是：2
2
```
推荐阅读：
- [理解与底层原理](https://www.liaoxuefeng.com/wiki/1252599548343744/1266265125480448)
- [Spring AOP动态代理](https://segmentfault.com/a/1190000037596406)

Spring框架不需要创建InvocationHandler，只需要创建一个切面对象，将所有的非业务代码在切面对象中完成即可，Spring框架底层会自动根据切面类以及目标类生成一个代理对象。
#### 优点
- 可以降低模块之间的耦合性提高代码的复用性
- 提高代码的维护性
- 集中管理非业务代码，便于维护
- 业务代码不受非业务代码的影响，逻辑更加清晰

#### 术语
- Aspect：切面，即一个横跨多个核心逻辑的功能，或者称之为系统关注点；
- Joinpoint：连接点，即定义在应用程序流程的何处插入切面的执行；
- Pointcut：切入点，即一组连接点的集合；
- Advice：增强，指特定连接点上执行的动作；
- Introduction：引介，指为一个已有的Java对象动态地增加新的接口；
- Weaving：织入，指将切面整合到程序的执行流程中；
- Interceptor：拦截器，是一种实现增强的方式；
- Target Object：目标对象，即真正执行业务的核心逻辑对象；
- AOP Proxy：AOP代理，是客户端持有的增强后的对象引用。

#### Spring 中使用
```dev
<!-- 引入Spring中AOP支持 -->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-aspects</artifactId>
    <version>${spring.version}</version>
</dependency>
```
1. 定义切面类
```java
@Aspect
@Component
public class LoggingAspect {
    // 在执行UserService的每个方法前执行:
    @Before("execution(public * com.itranswarp.learnjava.service.UserService.*(..))")
    public void doAccessCheck() {
        System.err.println("[Before] do access check...");
    }

    // 在执行MailService的每个方法前后执行:
    @Around("execution(public * com.itranswarp.learnjava.service.MailService.*(..))")
    public Object doLogging(ProceedingJoinPoint pjp) throws Throwable {
        System.err.println("[Around] start " + pjp.getSignature());
        Object retVal = pjp.proceed();
        System.err.println("[Around] done " + pjp.getSignature());
        return retVal;
    }
}
```
- @Component，将切面类加载到IoC容器中。
- @Aspect，表示该类是一个切面类。

2. 加入拦截器
- @Before，表示方法的执行时机是在业务方法之前，execution 表达式表示切入点是Callmpl类中的add 方法。**(可使用通配符)**
- @After，表示方法的执行时机是在业务方法结束之后，execution表达式表示切入点是Callmpl类中的add方法。
- @AfterReturning，表示方法的执行时机是在业务方法返回结果之后，execution表达式表示切入点是Callmpl类中的add方法,returning是将业务方法的返回值与切面类方法的形参进行绑定。
- @AfterThrowing，表示方法的执行时机是在业务方法抛出异常之后，execution表达式表示切入点是Callmpl类中的add方法，throwing是将业务方法的异常与切面类方法的形参进行绑定。
- @Around，能完全控制目标代码是否执行，并可以在执行前后、抛异常后执行任意拦截代码，可以说是包含了上面所有功能。

在实际项目中，这种execution写法其实很少使用，一般我们可以自己写一个注解，然后使用注解标识需要加入对应切面的方法或类：
```java
// 自定义注解
@Target(METHOD)
@Retention(RUNTIME)
public @interface MetricTime {
    String value();
}
/************/
// 声明切面类
@Aspect
@Component
public class MetricAspect {
    @Around("@annotation(metricTime)")  // 使用注解代替execution表达式
    public Object metric(ProceedingJoinPoint joinPoint, MetricTime metricTime) throws Throwable {
        String name = metricTime.value();
        long start = System.currentTimeMillis();
        try {
            return joinPoint.proceed();
        } finally {
            long t = System.currentTimeMillis() - start;
            // 写入日志或发送至JMX:
            System.err.println("[Metrics] " + name + ": " + t + "ms");
        }
    }
}
/************/
// 再需要插入切面的地方加入注解
@Component
public class UserService {
    // 监控register()方法性能:
    @MetricTime("register")
    public User register(String email, String password, String name) {
        ...
    }
    ...
}
```


## 参考

- https://zhuanlan.zhihu.com/p/147340068
- https://juejin.cn/post/6844903923703087111
- https://zhuanlan.zhihu.com/p/36840573
- https://www.liaoxuefeng.com/wiki/1252599548343744/1266265100383840
