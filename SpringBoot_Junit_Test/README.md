### SpringBoot Junit单元测试

如果是使用spring-boot 1.4以下的版本,使用@SpringApplicationConfiguration注解

```Java
@RunWith(SpringJUnit4ClassRunner.class)
@SpringApplicationConfiguration(classes = 启动类.class)
public class ApplicationTest {

    //代码省略
}
```

如果spring-boot是1.4以上的版本,使用@SpringBootTest注解

```Java
@RunWith(SpringJUnit4ClassRunner.class)
@SpringBootTest(classes = 启动类.class)
public class ApplicationTest {

    //代码省略
}
```

> 注意，1.4版本支持 @SpringBootTest注解,或者直接使用 @SpringBootContextLoader注解