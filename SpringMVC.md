## Spring MVC
MVC是Web服务端常见的设计模式，Spring MVC就是一个基于Java的实现了MVC设计模式的请求驱动类型的轻量级Web框架，通过把Model，View，Controller分离，将web层进行职责解耦，把复杂的web应用分成逻辑清晰的几部分，简化开发，减少出错，方便组内开发人员之间的配合。

在传统的Servlet API上实现MVC架构是一件非常繁琐的事情，我们需要实现不同的接口，而且JSP开发实在不够友好，我们希望有这么一个MVC框架，能够自动解析请求的类型，自动对应请求中的参数，这也就是Spring MVC所做的众多工作之一。

<img alt="Spring MVC工作流程" src="https://img.foril.fun/20220703164144.png" width=600px style="margin:10px auto"/>

## 参考文献
- https://baijiahao.baidu.com/s?id=1717728194505578801&wfr=spider&for=pc

## 注解 
  * value：指定URL请求的实际地址；
  * method：指定请求类型（GET、POST、DELETE、PUT）；
  * params：指定请求中必须包含的参数。
- @Controller 将该类交给IoC容器管理，同时使其成为控制器，可以接受客户端请求；
- @RequestParam 将请求参数与业务方法的形参绑定；
- @PathVariable 映射在路径中的RESTful风格参数；
- @CookieValue 映射cookie中的值 
- @ResponseBody 将返回值直接作为响应返回，不经过视图解析器处理 
- @RestController 相当于类内所有方法加上了@ResponseBody
- @RequestBody 接收客户端发送的JSON数据并赋值给形参


***
以下都算是JSP的东西
***

## 文件上传下载

### 单文件上传

底层使用Apache fileupload组件完成上传，Spring MVC对其完成了封装。

## Spring数据校验

Spring两种数据校验的方式：
1. 基于Validator接口；
2. 使用Annotation JSR-303标准进行校验。