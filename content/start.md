# 起步

## 传统Web应用程序

假设我们要写一个非常简单的Hello World Web应用，至少需要以下这些东西：

* 项目结构，Maven或者Gradle，至少需要依赖Spring MVC和Servlet API
* 在web.xml（或者WebApplicationInitializer实现）中声明Spring的DispatcherServlet
* Spring MVC配置
* 一个Controller，响应“Hello World”
* 一个Web服务器，如Tomcat

以上这么多点中，其实只有Controller才是我们关心的代码，其他的都是一些无聊的模板式的代码，多数Web应用都会用到。

看看Spring Boot怎么写：

```
@RestController
class HelloController {
  
  @RequestMapping("/")
  def hello() {
    return "Hello World"
  }
}
```
