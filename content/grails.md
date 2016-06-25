# 6. Grails

## 6.1 使用GORM持久化数据

GORM（Grails object-relational mapping）Book实体：

```
package readinglist

import grails.persistence.*

@Entity
class Book {
  Reader reader
  String isbn
  String title
  String author
  String description
}
```

要在Spring Boot项目中使用GORM，加入依赖：

```
compile("org.grails:gorm-hibernate4-spring-boot:1.1.0.RELEASE")
```

这是通过Hibernate，如果你使用MongoDB，也是可以的：

```
compile("org.grails:gorm-mongodb-spring-boot:1.1.0.RELEASE")
```

GORM Reader实体：

```
package readinglist
import grails.persistence.*

import org.springframework.security.core.GrantedAuthority
import org.springframework.security.core.authority.SimpleGrantedAuthority
import org.springframework.security.core.userdetails.UserDetails

@Entity
class Reader implements UserDetails {
  
  String username
  String fullname
  String password
  
  Collection<? extends GrantedAuthority> getAuthorities() {
    Arrays.asList(new SimpleGrantedAuthority("READER"))
  }
  
  boolean isAccountNonExpired() {
    true
  }
  
  boolean isAccountNonLocked() {
    true
  }
  
  boolean isCredentialsNonExpired() {
    true
  }
  
  boolean isEnabled() {
    true
  }
}
```

ReadingListController

```
package readinglist

import org.springframework.beans.factory.annotation.Autowired
import org.springframework.boot.context.properties.ConfigurationProperties
import org.springframework.http.HttpStatus
import org.springframework.stereotype.Controller
import org.springframework.ui.Model
import org.springframework.web.bind.annotation.ExceptionHandler
import org.springframework.web.bind.annotation.RequestMapping
import org.springframework.web.bind.annotation.RequestMethod
import org.springframework.web.bind.annotation.ResponseStatus

@Controller
@RequestMapping("/")
@ConfigurationProperties("amazon")
class ReadingListController {
  
  @Autowired
  AmazonProperties amazonProperties

  @ExceptionHandler(value=RuntimeException.class)
  @ResponseStatus(value=HttpStatus.BANDWIDTH_LIMIT_EXCEEDED)
  def error() {
    "error"
  }
  
  @RequestMapping(method=RequestMethod.GET)
  def readersBooks(Reader reader, Model model) {
    List<Book> readingList = Book.findAllByReader(reader)
    model.addAttribute("reader", reader)
    if (readingList) {
      model.addAttribute("books", readingList)
      model.addAttribute("amazonID", amazonProperties.getAssociateId())
    }
    "readingList"
  }
  
  @RequestMapping(method=RequestMethod.POST)
  def addToReadingList(Reader reader, Book book) {
    Book.withTransaction {
      book.setReader(reader)
      book.save()
    }
    "redirect:/"
  }
}
```

Groovy SecurityConfig：

```
package readinglist

import org.springframework.context.annotation.Configuration
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder
import org.springframework.security.config.annotation.web.builders.HttpSecurity
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter
import org.springframework.security.core.userdetails.UserDetailsService

@Configuration
class SecurityConfig extends WebSecurityConfigurerAdapter {
  
  void configure(HttpSecurity http) throws Exception {
    http
      .authorizeRequests()
        .antMatchers("/").access("hasRole('READER')")
        .antMatchers("/**").permitAll()
      .and()
      .formLogin()
        .loginPage("/login")
        .failureUrl("/login?error=true")
  }
  
  void configure(AuthenticationManagerBuilder auth) throws Exception {
    auth
      .userDetailsService(
        { username -> Reader.findByUsername(username) }
        as UserDetailsService)
  }
}
```

## 6.2 用GSP编写视图

目前为止我们都是使用Thymeleaf作为模板引擎，Grails项目同时提供了Groovy Server Pages（GSP）的自动配置，添加如下依赖：

```
compile("org.grails:grails-gsp-spring-boot:1.0.0")
```

接下来，把Thymeleaf readingList.html改写成readingList.gsp，同样是放在src/main/resources/templates路径下：

```
<!DOCTYPE html>
<html>
  <head>
    <title>Reading List</title>
    <link rel="stylesheet" href="/style.css"></link>
  </head>
  
  <body>
    <h2>Your Reading List</h2>
    
    <g:if test="${books}">
    <g:each in="${books}" var="book">
      <dl>
        <dt class="bookHeadline">
          ${book.title} by ${book.author}
          (ISBN: ${book.isbn}")
        </dt>
        <dd class="bookDescription">
          <g:if test="book.description">
            ${book.description}
          </g:if>
          <g:else>
            No description available
          </g:else>
        </dd>
      </dl>
    </g:each>
    </g:if>
    <g:else>
      <p>You have no books in your book list</p>
    </g:else>
    
    <hr/>
    
    <h3>Add a book</h3>
    
    <form method="POST">
      <label for="title">Title:</label>
      <input type="text" name="title" 
                         value="${book?.title}"/><br/>
      <label for="author">Author:</label>
      <input type="text" name="author"
                         value="${book?.author}"/><br/>
      <label for="isbn">ISBN:</label>
      <input type="text" name="isbn"
                         value="${book?.isbn}"/><br/>
      <label for="description">Description:</label><br/>
      <textarea name="description" rows="5" cols="80">
        ${book?.description}
      </textarea>
      <input type="hidden" name="${_csrf.parameterName}"
           value="${_csrf.token}" />
      <input type="submit" value="Add Book" />
    </form>
  
  </body>
</html>
```

需要注意的是，我们有一个隐藏域来放CSRF（Cross-Site Request Forgery）token，Spring Security要求在POST请求中有这个token，Thymeleaf会自动在渲染的HTML中加入这个token，但在GSP中，你得显式指定。

## 6.3 混合Spring Boot和Grails 3

Grails是一个建立在Spring，Groovy，Hibernate等之上的高层框架，目前，Grails 3建立在Spring Boot之上，所以对于Grails开发者和Spring Boot开发者来说，都有很好的开发体验。

在Mac OS X和大部分Unix系统上，安装Grails最方便的方法是使用SDKMAN：

```
$ sdk install grails
```

在Windows或者不能使用SDKMAN的系统上，你只好下载，解压并把bin目录添加到环境变量里去，检查安装是否成功：

```
$ grails -version
```

**创建Grails项目**

```
$ grails create-app readinglist
```

运行：

```
$ grails run-app
```

因为Grails 3的Gradle脚本使用了Spring Boot插件，所以也可以使用bootRun运行：

```
$ gradle bootRun
```

当然也可以打包运行：

```
$ gradle build
...
$ java -jar build/lib/readingList-0.1.jar
```

**定义domain**

```
$ grails create-domain-class Book
```

它会创建两个文件：Book.groovy和BookSpec.groovy，后面一个文件是测试Book类的Spock specification，初始是空的，Book.groovy在grails-app/domain/readingList目录下，像这样：

```
package readinglist

class Book {
  
  static constraints = {
  }
}
```

我们添加一些字段：

```
package readinglist

class Book {
  
  static constraints = {
  }
          
  String reader
  String isbn
  String title
  String author
  String description
}
```

静态变量constraints是用来定义验证约束的，这里我们不关心。

**编写Grails控制器**

有几种命令可以创建Controller：

* create-controller——创建一个空的Controller
* generate-controller——生成包含指定domain类型CRUD操作的骨架Controller
* generate-all——生成包含指定domain类型CRUD操作的骨架Controller以及视图

我们使用第一个：

```
$ grails create-controller ReadingList
```

该命令会在grails-app/controllers/readingList目录下创建一个ReadingListController：

```
package readinglist

class ReadingListController {

  def index() { }
}
```

此时该控制器已经可以处理/readingList请求了，会返回grails-app/views/readingList/index.gsp视图（暂时还不存在）

```
package readinglist

import static org.springframework.http.HttpStatus.*
import grails.transaction.Transactional

class ReadingListController {
  
  def index() {
    respond Book.list(params), model:[book: new Book()]
  }
  
  @Transactional
  def save(Book book) {
    book.reader = 'Craig'
    book.save flush:true
    redirect(action: "index")
  }
}
```

接下来就是编写GSP页面，和6.2中的差不多：

```
<!DOCTYPE html>
<html>
  <head>
    <meta name="layout" content="main"/>
    <title>Reading List</title>
    <link rel="stylesheet"
          href="/assets/main.css?compile=false"  />
    <link rel="stylesheet"
          href="/assets/mobile.css?compile=false"  />
    <link rel="stylesheet"
          href="/assets/application.css?compile=false"  />
  </head>
  
  <body>
  <h2>Your Reading List</h2>
  
  <g:if test="${bookList && !bookList.isEmpty()}">
    <g:each in="${bookList}" var="book">
    <dl>
      <dt class="bookHeadline">
        ${book.title}</span> by ${book.author}
        (ISBN: ${book.isbn}")
      </dt>
      <dd class="bookDescription">
        <g:if test="${book.description}">
        ${book.description}
        </g:if>
        <g:else>
        No description available
        </g:else>
      </dd>
    </dl>
    </g:each>
  </g:if>
  <g:else>
    <p>You have no books in your book list</p>
  </g:else>
  
  <hr/>
  
  <h3>Add a book</h3>
  
  <g:form action="save">
  <fieldset class="form">
    <label for="title">Title:</label>
    <g:field type="text" name="title" value="${book?.title}"/><br/>
    <label for="author">Author:</label>
    <g:field type="text" name="author" value="${book?.author}"/><br/>
    <label for="isbn">ISBN:</label>
    <g:field type="text" name="isbn" value="${book?.isbn}"/><br/>
    <label for="description">Description:</label><br/>
    <g:textArea name="description" value="${book?.description}" rows="5" cols="80"/>
  </fieldset>
  <fieldset class="buttons">
    <g:submitButton name="create" class="save"
      value="${message(code: 'default.button.create.label', default: 'Create')}" />
  </fieldset>
  </g:form>
  
  </body>
</html>
```
