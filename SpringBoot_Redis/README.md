### SpringBoot Redis配置以及使用Redis作缓存 ###

Spring Boot是为了简化Spring开发而生，从Spring 3.x开始，Spring社区的发展方向就是弱化xml配置文件而加大注解的戏份。最近召开的SpringOne2GX2015大会上显示：Spring Boot已经是Spring社区中增长最迅速的框架，前三名是：Spring Framework，Spring Boot和Spring Security，这个应该是未来的趋势。

我学习Spring Boot，是因为通过cli工具，spring boot开始往flask（python）、express(nodejs)等web框架发展和靠近，并且Spring Boot几乎不需要写xml配置文件。感兴趣的同学可以根据spring boot quick start这篇文章中的例子尝试下。

学习新的技术最佳途径是看官方文档，现在Spring boot的release版本是1.5.2-RELEASE，相应的参考文档是[Spring Boot Reference Guide(1.5.2-REALEASE)](http://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/)，如果有绝对英文比较吃力的同学，可以参考中文版Spring Boot参考指南。在前段时间阅读一篇技术文章，介绍如何阅读ios技术文档，我从中也有所收获，那就是我们应该重视spring.io上的guides部分——Getting Started Guides，这部分都是一些针对特定问题的demo，值得学习。

#### Spring Boot的项目结构 ####

	com
	 +- example
	     +- myproject
	         +- Application.java
	         |
	         +- domain
	         |   +- Customer.java
	         |   +- CustomerRepository.java
	         |
	         +- service
	         |   +- CustomerService.java
	         |
	         +- web
	             +- CustomerController.java

如上所示，Spring boot项目的结构划分为web->service->domain，其中domain文件夹可类比与业务模型和数据存储，即xxxBean和Dao层；service层是业务逻辑层，web是控制器。比较特别的是，这种类型的项目有自己的入口，即主类，一般命名为Application.java。Application.java不仅提供入口功能，还提供一些底层服务，例如缓存、项目配置等等。

#### 1. 自定义配置 ####
Spring Boot允许外化配置，这样你可以在不同的环境下使用相同的代码。你可以使用properties文件、yaml文件，环境变量和命令行参数来外化配置。使用@Value注解，可以直接将属性值注入到你的beans中。

Spring Boot使用一个非常特别的PropertySource来允许对值进行合理的覆盖，按照优先考虑的顺序排位如下：

1. 命令行参数
2. 来自java:comp/env的JNDI属性
3. Java系统属性（System.getProperties()）
4. 操作系统环境变量
5. 只有在random.*里包含的属性会产生一个RandomValuePropertySource
6. 在打包的jar外的应用程序配置文件（application.properties,包含YAML和profile变量）
7. 在打包的jar内的应用程序配置文件（application.properties,包含YAML和profile变量）
8. 在@Configuration类上的@PropertySource注解
9. 默认属性（使用SpringApplication.setDefaultProperties指定）

使用场景：可以将一个application.properties打包在Jar内，用来提供一个合理的默认name值；当运行在生产环境时，可以在Jar外提供一个application.properties文件来覆盖name属性；对于一次性的测试，可以使用特病的命令行开关启动，而不需要重复打包jar包。

具体的例子操作过程如下：

* 新建配置文件（application.properties）

	spring.redis.database=0
	spring.redis.host=localhost
	spring.redis.password= # Login password of the redis server.
	spring.redis.pool.max-active=8
	spring.redis.pool.max-idle=8
	spring.redis.pool.max-wait=-1
	spring.redis.pool.min-idle=0
	spring.redis.port=6379
	spring.redis.sentinel.master= # Name of Redis server.
	spring.redis.sentinel.nodes= # Comma-separated list of host:port pairs.
	spring.redis.timeout=0

* 使用@PropertySource引入配置文件

```Java
@Configuration
@PropertySource(value = "classpath:/redis.properties")
@EnableCaching
public class CacheConfig extends CachingConfigurerSupport {
  ......
}
```

* 使用@Value引用属性值

```Java
@Configuration
@PropertySource(value = "classpath:/redis.properties")
@EnableCaching
public class CacheConfig extends CachingConfigurerSupport {
  @Value("${spring.redis.host}")
  private String host;
  @Value("${spring.redis.port}")
  private int port;
  @Value("${spring.redis.timeout}")
  private int timeout;
  ......
}
```

#### 2. redis使用 ####
* 添加pom配置

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-redis</artifactId>
</dependency>
```

* 编写CacheConfig

```Java
@Configuration
@PropertySource(value = "classpath:/redis.properties")
@EnableCaching
public class CacheConfig extends CachingConfigurerSupport {
  @Value("${spring.redis.host}")
  private String host;
  @Value("${spring.redis.port}")
  private int port;
  @Value("${spring.redis.timeout}")
  private int timeout;
  @Bean
  public KeyGenerator wiselyKeyGenerator(){
      return new KeyGenerator() {
          @Override
          public Object generate(Object target, Method method, Object... params) {
              StringBuilder sb = new StringBuilder();
              sb.append(target.getClass().getName());
              sb.append(method.getName());
              for (Object obj : params) {
                  sb.append(obj.toString());
              }
              return sb.toString();
          }
      };
  }
  @Bean
  public JedisConnectionFactory redisConnectionFactory() {
      JedisConnectionFactory factory = new JedisConnectionFactory();
      factory.setHostName(host);
      factory.setPort(port);
      factory.setTimeout(timeout); //设置连接超时时间
      return factory;
  }
  @Bean
  public CacheManager cacheManager(RedisTemplate redisTemplate) {
      RedisCacheManager cacheManager = new RedisCacheManager(redisTemplate);
      // Number of seconds before expiration. Defaults to unlimited (0)
      cacheManager.setDefaultExpiration(10); //设置key-value超时时间
      return cacheManager;
  }
  @Bean
  public RedisTemplate<String, String> redisTemplate(RedisConnectionFactory factory) {
      StringRedisTemplate template = new StringRedisTemplate(factory);
      setSerializer(template); //设置序列化工具，这样ReportBean不需要实现Serializable接口
      template.afterPropertiesSet();
      return template;
  }
  private void setSerializer(StringRedisTemplate template) {
      Jackson2JsonRedisSerializer jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer(Object.class);
      ObjectMapper om = new ObjectMapper();
      om.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
      om.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
      jackson2JsonRedisSerializer.setObjectMapper(om);
      template.setValueSerializer(jackson2JsonRedisSerializer);
  }
}
```

* 启动缓存，使用@Cacheable注解在需要缓存的接口上即可

```Java
@Service
public class ReportService {
  @Cacheable(value = "reportcache", keyGenerator = "wiselyKeyGenerator")
  public ReportBean getReport(Long id, String date, String content, String title) {
      System.out.println("无缓存的时候调用这里---数据库查询");
      return new ReportBean(id, date, content, title);
  }
}
```

* 测试验证
	* 运行方法如下：
		* mvn clean package
		* java -jar target/dailyReport-1.0-SNAPSHOT.jar
	* 验证缓存起作用：
		* 访问：http://localhost:8080/report/test
		* 访问：http://localhost:8080/report/test2
	* 验证缓存失效（10s+后执行）：
		* 访问：http://localhost:8080/report/test2

#### 清楚缓存 ####
清除缓存是为了保持数据的一致性，CRUD (Create 创建，Retrieve 读取，Update 更新，Delete 删除) 操作中，除了 R 具备幂等性，其他三个发生的时候都可能会造成缓存结果和数据库不一致。为了保证缓存数据的一致性，在进行 CUD 操作的时候我们需要对可能影响到的缓存进行更新或者清除

```java
    //清除缓存
    @CacheEvict(value = Array("findUsers" ), allEntries = true)  
    def saveUser(user: User): Int = {
        userMapper.saveUser(user)
    }
```

本示例用的都是 @CacheEvict 清除缓存。如果你的 CUD 能够返回 City 实例，也可以使用 @CachePut 更新缓存策略。笔者推荐能用 @CachePut 的地方就不要用 @CacheEvict，因为后者将所有相关方法的缓存都清理掉，比如上面三个方法中的任意一个被调用了的话，provinceCities 方法的所有缓存将被清除。