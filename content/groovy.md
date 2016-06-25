# 5. Groovy

Spring Boot CLI可以很方便地使用Groovy编写Spring应用程序。

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