IOC就是把创建对象的过程通过`.xml`写好，交给容器注入对象中，不需要在类中声明。
```java
ApplicationContext context = new ClassPathXmlApplicationContext("application.xml");
```
之后就直接利用对象从容器中获取对象。

关键词：
```
spring-context
ApplicationContext
xml配置 利用反射
bean->id class
property->name value ref
```

```java
public class Main {
    public static void main(String[] args) {
        ApplicationContext context = new ClassPathXmlApplicationContext("application.xml");
        UserService userService = context.getBean(UserService.class);
        User user = userService.login("bob@example.com", "password");
        System.out.println(user.getName());
    }
}```
使用注解而不是xml完成Bean的配置，简洁不繁琐。  
`@Component` 完成Bean的声明， `@Autowired` 声明在域或构造函数变量上，完成从容器中对Bean的获取。  
`@ComponentScan` 告诉容器，自动搜索当前类所在的包以及子包，把所有标注为`@Component的Bean自动创建出来，并根据@Autowired进行装配