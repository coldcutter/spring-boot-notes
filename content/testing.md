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
  
  

```