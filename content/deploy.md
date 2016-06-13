# 8. 部署

## 8.1 部署到应用服务器

首先，我们构建一个war包：

```
apply plugin: 'war'

war {
    baseName = 'readinglist'
    version = '0.0.1-SNAPSHOT'
}
```

这样就能打成war包了，但目前这个war包没什么用，因为既没有包含web.xml也没有一个servlet initializer来enable Spring MVC的DispatcherServlet。这时候就需要用到SpringBootServletInitializer了，它是Spring的WebApplicationInitializer的一个实现，除了能配置DispatcherServlet，还能找到Filter、Servlet、ServletContextInitializer类型的beans，把它们绑定到servlet容器：

```
package readinglist;
import org.springframework.boot.builder.SpringApplicationBuilder;
import org.springframework.boot.context.web.SpringBootServletInitializer;

public class ReadingListServletInitializer extends SpringBootServletInitializer {
  @Override
  protected SpringApplicationBuilder configure(SpringApplicationBuilder builder) {
    return builder.sources(Application.class);
  }
}
```

configure方法里source了一个配置类，这个配置类就是Spring Boot的主配置来和启动类，实际上更简洁的做法是让Application类继承SpringBootServletInitializer就好了。

然后构建：

```
gradle build
```

war包就会在build/libs下面了，把它部署到Tomcat服务器就好了，当然，你仍然可以用java -jar运行：

```
java -jar readinglist-0.0.1-SNAPSHOT.war
```

## 8.2 生产数据库

开发的时候我们可以用自动配置的内置H2数据库，不过生产环境你就得用MySQL这样的数据库了：

```
---
spring:
  profiles: prod
  datasource:
    url: jdbc:mysql://localhost:3306/readinglist?useUnicode=true&characterEncoding=utf8
    username: ${MYSQL_USER}
    password: ${MYSQL_PASSWORD}
```

用户名和密码可以设置在系统环境变量里，防止密码暴露在代码里，要启用这段配置，要设置spring.profiles.active为prod，比较方便的做法是设置环境变量：

```
export SPRING_PROFILES_ACTIVE=prod
```

如果你使用Hibernate（JPA）和内置的H2数据库，Spring Boot会默认配置Hibernate自动创建数据库表，更具体地，它会设置Hibernate的hibernate.hbm2ddl.auto为create-drop，表明当Hibernate的SessionFactory创建完之后创建表，关闭的时候删除表。不过如果不使用内置的H2数据库，Spring Boot什么也不会做，表不会被创建，因此查询数据库的时候会报错，所以你要显示设置spring.jpa.hibernate.ddl-auto为create，create-drop或update，尽管设置为update看起来不错，不过在生产中并不推荐这么做，更好的做法是使用数据库迁移工具，Spring Boot为两种流行的工具提供了自动配置：

* [Flyway](http://flywaydb.org)
* [Liquibase](http://www.liquibase.org)




