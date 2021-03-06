
# 3 自定义配置

## 3.1 覆盖Spring Boot的自动配置

我们将向reading-list项目中加入Spring Security，很简单，加入security starter即可：

```
compile("org.springframework.boot:spring-boot-starter-security")
```

然后你再运行项目，访问浏览器，就会有一个HTTP Basic认证的对话框，用户名填“user”，密码需要在启动日志里找，像这样的一行：

```
Using default security password: 297af950-707a-48d1-a2cb-cc29909080f3
```

显然，大部分情况下这不是我们想要的样子，我们可能需要一个登录页面，然后利用数据库认证用户，所以Spring Boot关于security的自动配置不满足我们的要求，我们需要自定义security配置。

我们只需要提供一个继承了WebSecurityConfigurerAdapter的配置类SecurityConfig就可以覆盖自动配置了：

```
package readinglist;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.core.userdetails.UsernameNotFoundException;

@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
  
  @Autowired
  private ReaderRepository readerRepository;
  
  @Override
  protected void configure(HttpSecurity http) throws Exception {
    http
      .authorizeRequests()
        .antMatchers("/").access("hasRole('READER')")
        .antMatchers("/**").permitAll()
      .and()
        .formLogin()
        .loginPage("/login")
        .failureUrl("/login?error=true");
  }
  
  @Override
  protected void configure(AuthenticationManagerBuilder auth) throws Exception {
    auth.userDetailsService(new UserDetailsService() {
      @Override
      public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        return readerRepository.findOne(username);
      }
    });
  }
}
```

加了这个类之后，Spring Boot会跳过security自动配置而使用这个配置类。当然，还需要加入Reader类和ReaderRepository接口：

```
package readinglist;
import java.util.Arrays;
import java.util.Collection;
import javax.persistence.Entity;
import javax.persistence.Id;
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.authority.SimpleGrantedAuthority;
import org.springframework.security.core.userdetails.UserDetails;

@Entity
public class Reader implements UserDetails {
  @Id
  private String username;
  private String fullname;
  private String password;
  // getters and setters
  
  // UserDetails methods
  @Override
  public Collection<? extends GrantedAuthority> getAuthorities() {
    return Arrays.asList(new SimpleGrantedAuthority("READER"));
  }
  
  @Override
  public boolean isAccountNonExpired() {
    return true;
  }
  
  @Override
  public boolean isAccountNonLocked() {
    return true;
  }
  
  @Override
  public boolean isCredentialsNonExpired() {
    return true;
  }
  
  @Override
  public boolean isEnabled() {
    return true;
  }
}
```

```
package readinglist;
import org.springframework.data.jpa.repository.JpaRepository;

public interface ReaderRepository extends JpaRepository<Reader, String> {
}
```

首先，Spring Boot会优先考虑应用提供的配置类，然后才是自动配置类，其次，多亏了SecurityConfig的@EnableWebSecurity注解，这个注解会import WebSecurityConfiguration，然后自动配置的SpringBootWebSecurityConfiguration这个类因为不满足@ConditionalOnMissingBean(WebSecurityConfiguration.class)而不生效。

## 3.2 使用properties外化配置

很多情况下我们不会完全不用自动配置，而是需要修改某几个配置属性，Spring Boot提供了超过300个属性，通过属性来修改自动配置的bean参数会是种更方便的选择，可以通过环境变量、Java系统属性、JNDI、命令行参数或属性文件来提供。

有这些地方可以配置属性，优先级从高到低（列表靠前的覆盖靠后的）：

1. 命令行参数
2. JNDI属性 from java:comp/env
3. JVM系统属性
4. 操作系统环境变量
5. random.\*前缀的随机生成的值（设置其他属性时引用的，如${random.long}）
6. 应用外部的application.properties或application.yml文件
7. 应用内部的application.properties或application.yml文件
8. @PropertySource指定的属性源
9. 默认属性

至于application.properties和application.yml，它们可以位于任意4个位置：

1. 外部，应用运行目录的/config子目录下
2. 外部，应用运行目录下
3. 内部，config包下
4. 内部，classpath根路径下

上面的列表也是优先级从高到低的，当application.properties和application.yml同时存在于同一优先级目录时，yml会覆盖properties文件的属性。

下面介绍一些常用的配置属性。

**关闭模板缓存**

当你修改Thymeleaf模板的时候，你会发现每次都需要重启才会生效，原因就在于，为了提高性能，默认会缓存模板（只需编译一次），你可以通过设置spring.thymeleaf.cache为false来关闭模板缓存，比如通过命令行参数：

```
java -jar readinglist-0.0.1-SNAPSHOT.jar --spring.thymeleaf.cache=false
```

或者通过application.yml：

```
spring:
  thymeleaf:
    cache: false
```

不过务必**确保在生产环境上不要关闭模板缓存**，以免影响性能，当然你也可以通过环境变量：

```
export spring_thymeleaf_cache=false
```

注意把.（点）和-（横杠）替换成\_（下划线）

**配置内置服务器**

配置HTTPS，首先是使用JDK的keytool工具创建一个keystore：

```
keytool -keystore mykeys.jks -genkey -alias tomcat -keyalg RSA
```

你会被问及一些问题，比如名字、组织等（无关紧要的），但是密码很重要，得记住，比如我用“letmein”作为密码，然后在application.yml中配置属性：

```
server:
  port: 8443
  ssl:
    key-store: file:///path/to/mykeys.jks
    key-store-password: letmein
    key-password: letmein
```

这里key-store的位置，可以用file://来指定文件系统的位置（比如file:///Users/coldcutter/mykeys.jks），但是如果keystore文件被打包进了JAR包，得用classpath:来引用它。

**配置Log**

Spring Boot默认使用Logback，打日志到控制台，日志级别为INFO，如果你不想使用Logback（使用log4j），可以这么配置：

```
configurations {
  all*.exclude group:'org.springframework.boot', module:'spring-boot-starter-logging'
}

compile("org.springframework.boot:spring-boot-starter-log4j")
```

想进一步配置log，可以在classpath根路径下（src/main/resources）创建一个logback.xml，例子：

```
<configuration>
  <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
    <encoder>
      <pattern>
        %d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n
      </pattern>
    </encoder>
  </appender>
  <logger name="root" level="INFO"/>
  <root level="INFO">
    <appender-ref ref="STDOUT" />
  </root>
</configuration>
```

尽管如此，我们常见的对于log的配置无非就是修改日志级别和日志打到什么地方，我们可以借助Spring Boot的配置属性（不用创建logback.xml），配置日志级别：

```
logging:
  level:
    root: WARN
    org:
      springframework:
        security: DEBUG
```

把root的日志级别设为WARN，Spring Security的日志级别设为DEBUG，你可以把Spring Security的包名写在一行：

```
logging:
  level:
    root: WARN
    org.springframework.security: DEBUG
```

你还可以配置log的输出路径和文件（得确保应用有那个路径的写权限）：

```
logging:
  path: /var/logs/
  file: BookWorm.log
```

配置log配置文件的位置，一般就用默认的logback.xml好了，不过当你需要根据不同的profile使用不同的日志配置时会很有用：

```
logging:
  config:
    classpath:logging-config.xml
```

**配置数据源**

比如MySQL，这么配：

```
spring:
  datasource:
    url: jdbc:mysql://localhost/readinglist
    username: dbuser
    password: dbpass
    driver-class-name: com.mysql.jdbc.Driver
```

driver-class-name其实没有必要，Spring Boot会根据url来推断。Spring Boot会使用这些配置来自动配置DataSource bean，DataSource会被池化，如果Tomcat的pooling DataSource在classpath下，就用它，否则会使用其他连接池实现：

* HikariCP
* Commons DBCP
* Commons DBCP 2

还可以通过JNDI：

```
spring:
  datasource:
    jndi-name: java:/comp/env/jdbc/readingListDS
```

如果你设置了spring.datasource.jndi-name属性，其他datasource连接属性会被忽略。

**配置自己的组件**

你可以在某个bean上加上@ConfigurationProperties注解来注入自定义的属性，比如：

```
@Controller
@RequestMapping("/")
@ConfigurationProperties(prefix = "amazon")
public class ReadingListController {

  private String associateId;
  
  public void setAssociateId(String associateId) {
    this.associateId = associateId;
  }
  
  // omitted
}
```

这样这个associateId（前缀amazon）属性会通过setter方法注入进来，Spring Boot的属性解析器足够智能，配置属性的时候驼峰（amazon.associateId）、连字符（amazon.associate-id）、下划线（amazon.associate_id）都可以。

当然你最好把相关属性聚合到一个类中，比如：

```
package readinglist;

import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;

@Component
@ConfigurationProperties("amazon")
public class AmazonProperties {

  private String associateId;
  
  public void setAssociateId(String associateId) {
    this.associateId = associateId;
  }
  
  public String getAssociateId() {
    return associateId;
  }
}
```

**配置profiles**

不同的环境需要不同的属性，Spring 3.1引入了基于profile的配置，profiles其实就是一种条件配置，根据运行时哪些profiles被激活来决定使用或忽略哪些beans和配置类，比如可以加注解@Profile("production")表示只在production profile激活的时候才应用配置。你可以使用spring.profiles.active来配置激活哪些profiles。

Spring Boot提供了基于profile的属性配置，更加方便。

如果你使用properties文件，你可以创建特定profile的配置文件，文件格式为：“application-{profile}.properties”，比如dev环境的属性可以放在application-dev.properties文件里，prod环境的属性放在application-prod.properties文件里，同时，不与特定profile有关的（default）属性还是放在application.properties文件里。

如果你是用YAML配置文件，你一样可以按照上面的文件命名方式来配置，不过你还有一中选择，就是把所有profile的属性都配置在一个application.yml文件里，比如：

```
logging:
  level:
    root: INFO
---
spring:
  profiles: development
logging:
  level:
    root: DEBUG
---
spring:
  profiles: production
logging:
  path: /tmp/
  file: BookWorm.log
  level:
    root: WARN
```

不同的配置section用“---”分隔。

## 3.3 自定义错误页面

Spring Boot默认提供一个“whitelabel”错误页，默认的错误处理器会寻找一个名为“error”的视图，如果没找到，就使用默认错误页，这个视图根本上是由视图解析器决定的：

* bean id是“error”且实现了Spring View接口的bean（Spring的BeanNameViewResolver）
* 如果是Thymeleaf，则为error.html
* 如果是FreeMarker，则为error.ftl
* 如果是Velocity，则为error.vm
* 如果是JSP，则为error.jsp

比如Thymeleaf，我们在src/main/resources/templates下创建一个error.html：

```
<html>
  <head>
    <title>Oops!</title>
    <link rel="stylesheet" th:href="@{/style.css}"></link>
  </head>
  <body>
    <div class="errorPage">
      <span class="oops">Oops!</span><br/>
      <img th:src="@{/MissingPage.png}"></img>
      <p>There seems to be a problem with the page you requested
         (<span th:text="${path}"></span>).</p>
      <p th:text="${'Details: ' + message}"></p>
    </div>
  </body>
</html>
```

我们可以在error模板中直接使用如下信息：

* timestamp，错误发生的时间
* path，错误发生的URL路径
* status，HTTP状态码
* error，错误原因
* exception，异常类名
* message，异常消息
* trace，异常堆栈追踪
* errors，BindingResult异常的errors
