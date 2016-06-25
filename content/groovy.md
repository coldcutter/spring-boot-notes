# 5. Groovy

Spring Boot CLI可以很方便地使用Groovy编写Spring应用程序。

## 5.2 创建CLI项目

首先创建一个项目目录：

```
mkdir readinglist
```

进去创建静态资源目录和Thymeleaf模板目录：

```
$ cd readinglist
$ mkdir static
$ mkdir templates
```

分别将之前的style.css和readingList.html放进去。

在根目录下添加Boot.groovy，无需访问修饰符、行末分号、getter和setter：

```
class Book {
  Long id
  String reader
  String isbn
  String title
  String author
  String description
}
```

还有一个区别是这里没有Spring Data JPA的注解，事实上我们将使用Spring的JdbcTemplate，而不是Spring Data JPA，因为JPA需要.class文件来实现repository接口，当你通过命令行运行Groovy脚本的时候，CLI在内存中编译脚本且不会生成.class文件，所以使用CLI开发的时候JPA不太好用。不过你可以使用CLI的jar命令打包成JAR文件，JAR包里面就有.class文件了，不过开发的时候为了方便，就用JdbcTemplate了。

接下来写ReadingListRepository接口：

```
interface ReadingListRepository {
  List<Book> findByReader(String reader)
  void save(Book book)
}
```

写对应的接口实现：

```
@Repository
class JdbcReadingListRepository implements ReadingListRepository {

  @Autowired
  JdbcTemplate jdbc
  
  List<Book> findByReader(String reader) {
    jdbc.query(
        "select id, reader, isbn, title, author, description from Book where reader=?",
        { rs, row ->
            new Book(id: rs.getLong(1),
                reader: rs.getString(2),
                isbn: rs.getString(3),
                title: rs.getString(4),
                author: rs.getString(5),
                description: rs.getString(6))
        } as RowMapper,
        reader)
  }
  
  void save(Book book) {
    jdbc.update("insert into Book " +
                "(reader, isbn, title, author, description) " +
                "values (?, ?, ?, ?, ?)",
        book.reader,
        book.isbn,
        book.title,
        book.author,
        book.description)
  }
}
```

创建schema.sql：

```
create table Book (
  id identity,
  reader varchar(20) not null,
  isbn varchar(10) not null,
  title varchar(50) not null,
  author varchar(50) not null,
  description varchar(2000) not null
);
```

创建ReadingListController.groovy：

```
@Controller
@RequestMapping("/")
class ReadingListController {

  String reader = "Craig"
  
  @Autowired
  ReadingListRepository readingListRepository
  
  @RequestMapping(method=RequestMethod.GET)
  def readersBooks(Model model) {
    List<Book> readingList = readingListRepository.findByReader(reader)
    if (readingList) {
      model.addAttribute("books", readingList)
    }
    "readingList"
  }
  
  @RequestMapping(method=RequestMethod.POST)
  def addToReadingList(Book book) {
    book.setReader(reader)
    readingListRepository.save(book)
    "redirect:/"
  }
}
```

创建Grabs.groovy：

```
@Grab("h2")
@Grab("spring-boot-starter-thymeleaf")
class Grabs {}
```

下面就可以运行了，到项目根目录：

```
$ spring run .
```

发生了什么？

当你用CLI运行应用的时候，CLI试图使用内置的Groovy编译器编译Groovy代码，不过因为有些类型不认识（如JdbcTemplate, Controller, RequestMapping），所以失败了，不过它没放弃，它知道JdbcTemplate可以通过Spring Boot JDBC starter得到，Spring MVC的类型可以通过Spring Boot web starter得到，所以它会从Maven仓库（默认是Maven Central）获取这些依赖。此时还是失败，因为没有import，不过CLI知道许多常用类型所在的包，它会添加到Groovy编译器的默认包列表，这时候如果没有别的语法错误，编译通过，并且CLI通过内部的启动方法来运行应用，此时，Spring Boot自动配置介入。

## 5.2 获取依赖

@Grab注解来自于Groovy’s Grape（Groovy Adaptable Packaging Engine or Groovy Advanced Packaging Engine），用法：

```
@Grab(group="com.h2database", module="h2", version="1.4.190")
@Grab("com.h2database:h2:1.4.185")
```

Spring Boot CLI扩展了@Grab，使它用起来更简洁，许多依赖都不用指定版本（版本根据CLI的版本来确定），对一些常用的依赖，甚至都不用指定group：

```
@Grab("com.h2database:h2")
@Grab("h2")
```

**覆盖默认依赖版本**

Spring Boot引入了@GrabMetadata注解：

```
@GrabMetadata(“com.myorg:custom-versions:1.0.0”)
```

如上，它会从Maven仓库的com/myorg目录加载一个名为custom-versions.properties的属性文件，每行的键是group ID和module ID，值是版本号：

```
com.h2database:h2=1.4.186
```

[Spring IO platform](http://platform.spring.io/platform/)提供了一套兼容的依赖版本集，它是Spring Boot依赖版本集的超集：

```
@GrabMetadata('io.spring.platform:platform-versions:1.0.4.RELEASE')
```

**添加依赖仓库**

@GrabResolver注解可以添加依赖仓库，比如：

```
@GrabResolver(name='jboss', root='https://repository.jboss.org/nexus/content/groups/public-jboss')
```

## 5.3 运行测试

CLI提供了test命令来运行测试，测试可以放在项目的任何位置，不过建议放在一起，比如tests目录下：

```
$ mkdir tests
```

创建ReadingListControllerTest.groovy：

```
import org.springframework.test.web.servlet.MockMvc
import static org.springframework.test.web.servlet.setup.MockMvcBuilders.*
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*
import static org.mockito.Mockito.*

class ReadingListControllerTest {
  
  @Test
  void shouldReturnReadingListFromRepository() {
    List<Book> expectedList = new ArrayList<Book>()
    expectedList.add(new Book(
        id: 1,
        reader: "Craig",
        isbn: "9781617292545",
        title: "Spring Boot in Action",
        author: "Craig Walls",
        description: "Spring Boot in Action is ..."
    ))
    
    def mockRepo = mock(ReadingListRepository.class)
    when(mockRepo.findByReader("Craig")).thenReturn(expectedList)
    
    def controller = new ReadingListController(readingListRepository: mockRepo)
    MockMvc mvc = standaloneSetup(controller).build()
    mvc.perform(get("/"))
        .andExpect(view().name("readingList"))
        .andExpect(model().attribute("books", expectedList))
  }
}
```

运行测试：

```
$ spring test tests/ReadingListControllerTest.groovy
```

如果有多个测试，你也可以只提供一个目录：

```
$ spring test tests
```

如果你更倾向于写Spock测试，而不是JUnit，test命令也可以执行Spock测试：

```
import org.springframework.test.web.servlet.MockMvc
import static org.springframework.test.web.servlet.setup.MockMvcBuilders.*
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*
import static org.mockito.Mockito.*

class ReadingListControllerSpec extends Specification {
  
  MockMvc mockMvc
  List<Book> expectedList
  
  def setup() {
    expectedList = new ArrayList<Book>()
    expectedList.add(new Book(
      id: 1,
      reader: "Craig",
      isbn: "9781617292545",
      title: "Spring Boot in Action",
      author: "Craig Walls",
      description: "Spring Boot in Action is ..."
    ))
    
    def mockRepo = mock(ReadingListRepository.class)
    when(mockRepo.findByReader("Craig")).thenReturn(expectedList)
    
    def controller = new ReadingListController(readingListRepository: mockRepo)
    mockMvc = standaloneSetup(controller).build()
  }
  
  def "Should put list returned from repository into model"() {
    when:
      def response = mockMvc.perform(get("/"))
    
    then:
      response.andExpect(view().name("readingList"))
              .andExpect(model().attribute("books", expectedList))
  }
}
```

## 5.4 创建可部署的包

在项目根目录下，执行：

```
$ spring jar ReadingList.jar .
```

然后就可以运行了：

```
$ java -jar ReadingList.jar
```
