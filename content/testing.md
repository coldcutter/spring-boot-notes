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