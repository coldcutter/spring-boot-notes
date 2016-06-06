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