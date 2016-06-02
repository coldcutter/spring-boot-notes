# 1 起步

## 1.1 传统Web应用程序

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

没有配置，没有web.xml，也没有构建脚本，甚至没有server，如果你安装了CLI，你就可以这么运行：

```
spring run HelloController.groovy
```

## 1.2 主要特性

### 1.2.1 Auto Configuration

如果你要用JDBC访问关系型数据库，就需要配置一个JdbcTemplate，像这样：

```
@Bean
public JdbcTemplate jdbcTemplate(DataSource dataSource) {
  return new JdbcTemplate(dataSource);
}

@Bean
public DataSource dataSource() {
  return new EmbeddedDatabaseBuilder()
          .setType(EmbeddedDatabaseType.H2)
          .addScripts('schema.sql', 'data.sql')
          .build();
}
```

为了访问个数据库，每次都要自己配置这样的Bean，太烦了，Spring Boot的自动配置机制非常棒，当它发现项目中有H2依赖包，它自动给你配置一个H2的DataSource，当它发现JdbcTemplate在类路径下，自动给你配置一个JdbcTemplate，并且会帮你自动注入，你就不用自己配置上面两个Bean了，直接拿来用就好了。