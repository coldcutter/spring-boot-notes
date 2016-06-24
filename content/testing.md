# 4. 测试

## 4.1 集成测试

```
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = AddressBookConfiguration.class)
public class AddressServiceTests {

  @Autowired
  private AddressService addressService;
  
  @Test
  public void testService() {
    Address address = addressService.findByLastName("Sheman");
    assertEquals("P", address.getFirstName());
    // ...
  }
}
```

SpringJUnit4ClassRunner类用来启用Spring集成测试，它是一个加载Spring应用上下文和在测试类中启用自动注入的JUnit类运行器（从Spring 4.2起，你也可以使用SpringClassRule和SpringMethodRule这种基于规则的替换方案），@ContextConfiguration指定了如何加载Spring应用上下文。

不过Spring Boot应用是由SpringApplication加载的（或SpringBootServletInitializer），它不仅加载应用上下文，还启用日志，加载外部配置文件等Spring Boot特性，但是如果使用@ContextConfiguration，这些特性是没有的，所以一般是使用Spring Boot的@SpringApplicationConfiguration来替换@ContextConfiguration，它会像SpringApplication那样。

## 4.2 测试Web应用

测试Web应用，有两个选择：

1. Spring Mock MVC——模拟一个Servlet容器来测试Controller
2. Web集成测试——在内置Servlet容器（Tomcat或Jetty）中启动应用来测试

**Spring Mock MVC**

```
@RunWith(SpringJUnit4ClassRunner.class)
@SpringApplicationConfiguration(classes = ReadingListApplication.class)
@WebAppConfiguration
public class MockMvcWebTests {
  
  @Autowired
  private WebApplicationContext webContext;
  
  private MockMvc mockMvc;
  
  @Before
  public void setupMockMvc() {
    mockMvc = MockMvcBuilders
        .webAppContextSetup(webContext)
        .build();
  }
}
```

测试HTTP GET请求

```
@Test
public void homePage() throws Exception {
  mockMvc.perform(MockMvcRequestBuilders.get("/readingList"))
      .andExpect(MockMvcResultMatchers.status().isOk())    
      .andExpect(MockMvcResultMatchers.view().name("readingList"))
      .andExpect(MockMvcResultMatchers.model().attributeExists("books"))
      .andExpect(MockMvcResultMatchers.model().attribute("books", Matchers.is(Matchers.empty())));
}
```

加入静态引入，可以更简洁：

```
import static org.hamcrest.Matchers.*;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;

@Test
public void homePage() throws Exception {
  mockMvc.perform(get("/readingList"))
      .andExpect(status().isOk())
      .andExpect(view().name("readingList"))
      .andExpect(model().attributeExists("books"))
      .andExpect(model().attribute("books", is(empty())));
}
```

测试HTTP POST请求，添加一本书：

```
@Test
public void postBook() throws Exception {
  mockMvc.perform(post("/readingList")
      .contentType(MediaType.APPLICATION_FORM_URLENCODED)
      .param("title", "BOOK TITLE")
      .param("author", "BOOK AUTHOR")
      .param("isbn", "1234567890")
      .param("description", "DESCRIPTION"))
      .andExpect(status().is3xxRedirection())
      .andExpect(header().string("Location", "/readingList"));
  
  Book expectedBook = new Book();
  expectedBook.setId(1L);
  expectedBook.setReader("craig");
  expectedBook.setTitle("BOOK TITLE");
  expectedBook.setAuthor("BOOK AUTHOR");
  expectedBook.setIsbn("1234567890");
  expectedBook.setDescription("DESCRIPTION");
  
  mockMvc.perform(get("/readingList"))
      .andExpect(status().isOk())
      .andExpect(view().name("readingList"))
      .andExpect(model().attributeExists("books"))
      .andExpect(model().attribute("books", hasSize(1)))
      .andExpect(model().attribute("books", contains(samePropertyValuesAs(expectedBook))));
}
```

测试安全性：

引入包：

```
testCompile("org.springframework.security:spring-security-test")
```

然后加一行：

```
@Before
public void setupMockMvc() {
  mockMvc = MockMvcBuilders
      .webAppContextSetup(webContext)
      .apply(springSecurity())
      .build();
}
```

springSecurity()是SecurityMockMvcConfigurers的一个静态方法，它会遵照你的安全配置。

```
@Test
public void homePage_unauthenticatedUser() throws Exception {
  mockMvc.perform(get("/"))
      .andExpect(status().is3xxRedirection())
      .andExpect(header().string("Location", "http://localhost/login"));
}
```

如何测试一个通过认证的请求？Spring Security提供两个注解：

* @WithMockUser——加载一个UserDetails，使用指定的用户名、密码和权限
* @WithUserDetails——根据指定的用户名查找一个UserDetails对象并加载

```
@Test
@WithMockUser(username="craig", password="password", roles="READER")
public void homePage_authenticatedUser() throws Exception {
   ...
}
```

@WithUserDetails注解使用配置好的UserDetailsService来加载UserDetails对象：

```
@Test
@WithUserDetails("craig")
public void homePage_authenticatedUser() throws Exception {

  Reader expectedReader = new Reader();
  expectedReader.setUsername("craig");
  expectedReader.setPassword("password");
  expectedReader.setFullname("Craig Walls");
  
  mockMvc.perform(get("/"))
      .andExpect(status().isOk())
      .andExpect(view().name("readingList"))
      .andExpect(model().attribute("reader", samePropertyValuesAs(expectedReader)))
      .andExpect(model().attribute("books", hasSize(0)))
}
```

## 测试运行中的应用

Spring Boot’s @WebIntegrationTest注解，不仅可以创建一个应用上下文，还可以启动内置Servlet容器。

```
@RunWith(SpringJUnit4ClassRunner.class)
@SpringApplicationConfiguration(classes = ReadingListApplication.class)
@WebIntegrationTest
public class SimpleWebTest {
  
  @Test(expected = HttpClientErrorException.class)
  public void pageNotFound() {
    try {
      RestTemplate rest = new RestTemplate();
      rest.getForObject("http://localhost:8080/bogusPage", String.class);
      fail("Should result in HTTP 404");
    } catch (HttpClientErrorException e) {
      assertEquals(HttpStatus.NOT_FOUND, e.getStatusCode());
      throw e;
    }
  }
}
```

默认启动8080端口，你可以设置如下之一来选择随机端口：

```
@WebIntegrationTest(value={"server.port=0"})
@WebIntegrationTest("server.port=0")
@WebIntegrationTest(randomPort=true)
```

现在端口是随机了，但是如何获取这个端口呢？你可以注入进来：

```
@Value("${local.server.port}")
private int port;

rest.getForObject("http://localhost:{port}/bogusPage", String.class, port);
```

**使用Selenium测试HTML页面**

