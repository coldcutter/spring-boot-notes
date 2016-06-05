
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