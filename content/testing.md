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