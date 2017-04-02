Spring 框架作为目前非常流行的一个 Java 应用开发框架，它所包含的内容是非常繁多的。Spring 框架包含几十个不同的子项目，涵盖应用开发的不同方面。要在这些子项目之间进行选择，并快速搭建一个可以运行的应用是比较困难的事情。Spring Boot 的目的在于快速创建可以独立运行的 Spring 应用。通过 Spring Boot 可以根据相应的模板快速创建应用并运行。Spring Boot 可以自动配置 Spring 的各种组件，并不依赖代码生成和 XML 配置文件。Spring Boot 可以大大提升使用 Spring 框架时的开发效率。

Spring 框架对于很多 Java 开发人员来说都不陌生。自从 2002 年发布以来，Spring 框架已经成为企业应用开发领域非常流行的基础框架。有大量的企业应用基于 Spring 框架来开发。Spring 框架包含几十个不同的子项目，涵盖应用开发的不同方面。如此多的子项目和组件，一方面方便了开发人员的使用，另外一个方面也带来了使用方面的问题。每个子项目都有一定的学习曲线。开发人员需要了解这些子项目和组件的具体细节，才能知道如何把这些子项目整合起来形成一个完整的解决方案。在如何使用这些组件上，并没有相关的最佳实践提供指导。对于新接触 Spring 框架的开发人员来说，并不知道如何更好的使用这些组件。Spring 框架的另外一个常见问题是要快速创建一个可以运行的应用比较麻烦。Spring Boot 是 Spring 框架的一个新的子项目，用于创建 Spring 4.0 项目。它的开发始于 2013 年。2014 年 4 月发布 1.0.0 版本。它可以自动配置 Spring 的各种组件，并不依赖代码生成和 XML 配置文件。Spring Boot 也提供了对于常见场景的推荐组件配置。Spring Boot 可以大大提升使用 Spring 框架时的开发效率。本文将对 Spring Boot 进行详细的介绍。

### 简介
从 Spring Boot 项目名称中的 Boot 可以看出来，Spring Boot 的作用在于创建和启动新的基于 Spring 框架的项目。它的目的是帮助开发人员很容易的创建出独立运行和产品级别的基于 Spring 框架的应用。Spring Boot 会选择最适合的 Spring 子项目和第三方开源库进行整合。大部分 Spring Boot 应用只需要非常少的配置就可以快速运行起来。

Spring Boot 包含的特性如下：

* 创建可以独立运行的 Spring 应用。
* 直接嵌入 Tomcat 或 Jetty 服务器，不需要部署 WAR 文件.
* 提供推荐的基础 POM 文件来简化 Apache Maven 配置。
* 尽可能的根据项目依赖来自动配置 Spring 框架。
* 提供可以直接在生产环境中使用的功能，如性能指标、应用信息和应用健康检查。
* 没有代码生成，也没有 XML 配置文件。

通过 Spring Boot，创建新的 Spring 应用变得非常容易，而且创建出的 Spring 应用符合通用的最佳实践。只需要简单的几个步骤就可以创建出一个 Web 应用。下面介绍使用 Maven 作为构建工具创建的 Spring Boot 应用。代码清单 1 给出了该应用的 POM 文件。

`清单 1. Spring Boot 示例应用的 POM 文件`

```Xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
 xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
<modelVersion>4.0.0</modelVersion>
<groupId>com.midgetontoes</groupId>
<artifactId>spring-boot-simple</artifactId>
<version>1.0-SNAPSHOT</version>
<properties>
 <spring.boot.version>1.1.4.RELEASE</spring.boot.version>
</properties>
<dependencies>
 <dependency>
 <groupId>org.springframework.boot</groupId>
 <artifactId>spring-boot-starter-web</artifactId>
 <version>${spring.boot.version}</version>
 </dependency>
</dependencies>
<build>
 <plugins>
 <plugin>
 <groupId>org.springframework.boot</groupId>
 <artifactId>spring-boot-maven-plugin</artifactId>
<version>${spring.boot.version}</version>
 <executions>
 <execution>
 <goals>
 <goal>repackage</goal>
 </goals>
 </execution>
 </executions>
 </plugin>
 </plugins>
</build>
</project>
```

从代码清单 1 中的 POM 文件中可以看到，应用所声明的依赖很少，只有一个“org.springframework.boot:spring-boot-starter-web”，而不是像其他 Spring 项目一样需要声明很多的依赖。当使用 Maven 命令“mvn dependency:tree”来查看项目实际的依赖时，会发现其中包含了 Spring MVC 框架、SLF4J、Jackson、Hibernate Validator 和 Tomcat 等依赖。这实际上 Spring 推荐的 Web 应用中使用的开源库的组合。代码清单 2 中给出了示例应用的 Java 代码。

`清单 2. Spring Boot 示例应用的 Java 代码`

```Java
@RestController
@EnableAutoConfiguration
public class Application {
 @RequestMapping("/")
 String home() {
 return "Hello World!";
 }
 public static void main(String[] args) throws Exception {
 SpringApplication.run(Application.class, args);
 }
}
```

代码清单 2 中的 Java 类 Application 是一个简单的可以独立运行的 Web 应用。直接运行该 Java 类会启动一个内嵌的 Tomcat 服务器运行在 8080 端口。访问“http://localhost:8080”可以看到页面上显示“Hello World!”。也就是说，只需要简单的 2 个文件就可以启动一个独立运行的 Web 应用。并不需要额外安装 Tomcat 这样的应用服务器，也不需要打包成 WAR 文件。可以通过“mvn spring-boot:run”在命令行启动该应用。在代码清单 1 中的 POM 文件中添加了“org.springframework.boot:spring-boot-maven-plugin”插件。在添加了该插件之后，当运行“mvn package”进行打包时，会打包成一个可以直接运行的 JAR 文件，使用“java -jar”命令就可以直接运行。这在很大程度上简化了应用的部署，只需要安装了 JRE 就可以运行。

代码清单 2 中的“@EnableAutoConfiguration”注解的作用在于让 Spring Boot 根据应用所声明的依赖来对 Spring 框架进行自动配置，这就减少了开发人员的工作量。注解“@RestController”和”@RequestMapping”由 Spring MVC 提供，用来创建 REST 服务。这两个注解和 Spring Boot 本身并没有关系。

### Spring Boot 推荐的基础 POM 文件
上一节的代码清单 1 中给出的“org.springframework.boot:spring-boot-starter-web”是 Spring Boot 所提供的推荐的基础 POM 文件之一，用来提供创建基于 Spring MVC 的 Web 应用所需的第三方库依赖。除了这个 POM 文件之外，Spring Boot 还提供了其他类似的 POM 文件。所有这些基础 POM 依赖都在“org.springframework.boot”组中。一些重要 POM 文件的具体说明见表 1。

`表 1. Spring Boot 推荐的基础 POM 文件`


	名称								说明
	spring-boot-starter				核心 POM，包含自动配置支持、日志库和对 YAML 配置文件的支持。
	spring-boot-starter-amqp			通过 spring-rabbit 支持 AMQP。
	spring-boot-starter-aop			包含 spring-aop 和 AspectJ 来支持面向切面编程（AOP）。
	spring-boot-starter-batch		支持 Spring Batch，包含 HSQLDB。
	spring-boot-starter-data-jpa		包含 spring-data-jpa、spring-orm 和 Hibernate 来支持 JPA。
	spring-boot-starter-data-mongodb	包含 spring-data-mongodb 来支持 MongoDB。
	spring-boot-starter-data-rest	通过 spring-data-rest-webmvc 支持以 REST 方式暴露 Spring Data 仓库。
	spring-boot-starter-jdbc			支持使用 JDBC 访问数据库。
	spring-boot-starter-security		包含 spring-security。
	spring-boot-starter-test			包含常用的测试所需的依赖，如 JUnit、Hamcrest、Mockito 和 spring-test 等。
	spring-boot-starter-velocity		支持使用 Velocity 作为模板引擎。
	spring-boot-starter-web			支持 Web 应用开发，包含 Tomcat 和 spring-mvc。
	spring-boot-starter-websocket	支持使用 Tomcat 开发 WebSocket 应用。
	spring-boot-starter-ws			支持 Spring Web Services。
	spring-boot-starter-actuator		添加适用于生产环境的功能，如性能指标和监测等功能。
	spring-boot-starter-remote-shell	添加远程 SSH 支持。
	spring-boot-starter-jetty		使用 Jetty 而不是默认的 Tomcat 作为应用服务器。
	spring-boot-starter-log4j		添加 Log4j 的支持。
	spring-boot-starter-logging		使用 Spring Boot 默认的日志框架 Logback。
	spring-boot-starter-tomcat		使用 Spring Boot 默认的 Tomcat 作为应用服务器。

所有这些 POM 依赖的好处在于为开发 Spring 应用提供了一个良好的基础。Spring Boot 所选择的第三方库是经过考虑的，是比较适合产品开发的选择。但是 Spring Boot 也提供了不同的选项，比如日志框架可以用 Logback 或 Log4j，应用服务器可以用 Tomcat 或 Jetty。

### 自动配置
Spring Boot 对于开发人员最大的好处在于可以对 Spring 应用进行自动配置。Spring Boot 会根据应用中声明的第三方依赖来自动配置 Spring 框架，而不需要进行显式的声明。比如当声明了对 HSQLDB 的依赖时，Spring Boot 会自动配置成使用 HSQLDB 进行数据库操作。

Spring Boot 推荐采用基于 Java 注解的配置方式，而不是传统的 XML。只需要在主配置 Java 类上添加“@EnableAutoConfiguration”注解就可以启用自动配置。Spring Boot 的自动配置功能是没有侵入性的，只是作为一种基本的默认实现。开发人员可以通过定义其他 bean 来替代自动配置所提供的功能。比如当应用中定义了自己的数据源 bean 时，自动配置所提供的 HSQLDB 就不会生效。这给予了开发人员很大的灵活性。既可以快速的创建一个可以立即运行的原型应用，又可以不断的修改和调整以适应应用开发在不同阶段的需要。可能在应用最开始的时候，嵌入式的内存数据库（如 HSQLDB）就足够了，在后期则需要换成 MySQL 等数据库。Spring Boot 使得这样的切换变得很简单。

### 外部化的配置
在应用中管理配置并不是一个容易的任务，尤其是在应用需要部署到多个环境中时。通常会需要为每个环境提供一个对应的属性文件，用来配置各自的数据库连接信息、服务器信息和第三方服务账号等。通常的应用部署会包含开发、测试和生产等若干个环境。不同的环境之间的配置存在覆盖关系。测试环境中的配置会覆盖开发环境，而生产环境中的配置会覆盖测试环境。Spring 框架本身提供了多种的方式来管理配置属性文件。Spring 3.1 之前可以使用 PropertyPlaceholderConfigurer。Spring 3.1 引入了新的环境（Environment）和概要信息（Profile）API，是一种更加灵活的处理不同环境和配置文件的方式。不过 Spring 这些配置管理方式的问题在于选择太多，让开发人员无所适从。Spring Boot 提供了一种统一的方式来管理应用的配置，允许开发人员使用属性文件、YAML 文件、环境变量和命令行参数来定义优先级不同的配置值。

Spring Boot 所提供的配置优先级顺序比较复杂。按照优先级从高到低的顺序，具体的列表如下所示。
* 命令行参数。

* 通过 System.getProperties() 获取的 Java 系统参数。

* 操作系统环境变量。

* 从 java:comp/env 得到的 JNDI 属性。

* 通过 RandomValuePropertySource 生成的“random.*”属性。

* 应用 Jar 文件之外的属性文件。

* 应用 Jar 文件内部的属性文件。

* 在应用配置 Java 类（包含“@Configuration”注解的 Java 类）中通过“@PropertySource”注解声明的属性文件。

* 通过“SpringApplication.setDefaultProperties”声明的默认属性。

Spring Boot 的这个配置优先级看似复杂，其实是很合理的。比如命令行参数的优先级被设置为最高。这样的好处是可以在测试或生产环境中快速地修改配置参数值，而不需要重新打包和部署应用。

SpringApplication 类默认会把以“--”开头的命令行参数转化成应用中可以使用的配置参数，如 “--name=Alex” 会设置配置参数 “name” 的值为 “Alex”。如果不需要这个功能，可以通过 “SpringApplication.setAddCommandLineProperties(false)” 禁用解析命令行参数。

RandomValuePropertySource 可以用来生成测试所需要的各种不同类型的随机值，从而免去了在代码中生成的麻烦。RandomValuePropertySource 可以生成数字和字符串。数字的类型包含 int 和 long，可以限定数字的大小范围。以“random.”作为前缀的配置属性名称由 RandomValuePropertySource 来生成，如代码清单 3 所示。

清单 3. 使用 RandomValuePropertySource 生成的配置属性
	
	user.id=${random.value}
	user.count=${random.int}
	user.max=${random.long}
	user.number=${random.int(100)}
	user.range=${random.int[100, 1000]}

#### 属性文件
属性文件是最常见的管理配置属性的方式。Spring Boot 提供的 SpringApplication 类会搜索并加载 application.properties 文件来获取配置属性值。SpringApplication 类会在下面位置搜索该文件。

* 当前目录的“/config”子目录。

* 当前目录。

* classpath 中的“/config”包。

* classpath

上面的顺序也表示了该位置上包含的属性文件的优先级。优先级按照从高到低的顺序排列。可以通过“spring.config.name”配置属性来指定不同的属性文件名称。也可以通过“spring.config.location”来添加额外的属性文件的搜索路径。如果应用中包含多个 profile，可以为每个 profile 定义各自的属性文件，按照“application-{profile}”来命名。

对于配置属性，可以在代码中通过“@Value”来使用，如代码清单 4 所示。
清单 4. 通过“@Value”来使用配置属性

	@RestController
	@EnableAutoConfiguration
	public class Application {
	 @Value("${name}")
	 private String name;
	 @RequestMapping("/")
	 String home() {
	 return String.format("Hello %s!", name);
	 }
	}

#### YAML
相对于属性文件来说，YAML 是一个更好的配置文件格式。YAML 在 Ruby on Rails 中得到了很好的应用。SpringApplication 类也提供了对 YAML 配置文件的支持，只需要添加对 SnakeYAML 的依赖即可。代码清单 5 给出了 application.yml 文件的示例。

清单 5. 使用 YAML 表示的配置属性

	spring:
	 profiles: development
	db:
	 url: jdbc:hsqldb:file:testdb
	 username: sa
	 password:
	---
	spring:
	 profiles: test
	db:
	 url: jdbc:mysql://localhost/test
	 username: test
	 password: test


代码清单 5 中的 YAML 文件同时给出了 development 和 test 两个不同的 profile 的配置信息，这也是 YAML 文件相对于属性文件的优势之一。除了使用“@Value”注解绑定配置属性值之外，还可以使用更加灵活的方式。代码清单 6 给出的是使用代码清单 5 中的 YAML 文件的 Java 类。通过“@ConfigurationProperties(prefix="db")”注解，配置属性中以“db”为前缀的属性值会被自动绑定到 Java 类中同名的域上，如 url 域的值会对应属性“db.url”的值。只需要在应用的配置类中添加“@EnableConfigurationProperties”注解就可以启用该自动绑定功能。

清单 6. 使用 YAML 文件的 Java 类

	@Component
	@ConfigurationProperties(prefix="db")
	public class DBSettings {
	 private String url;
	 private String username;
	 private String password;
	}

### 开发 Web 应用
Spring Boot 非常适合于开发基于 Spring MVC 的 Web 应用。通过内嵌的 Tomcat 或 Jetty 服务器，可以简化对 Web 应用的部署。Spring Boot 通过自动配置功能对 Spring MVC 应用做了一些基本的配置，使其更加适合一般 Web 应用的开发要求。

#### HttpMessageConverter
Spring MVC 中使用 HttpMessageConverter 接口来在 HTTP 请求和响应之间进行消息格式的转换。默认情况下已经通过 Jackson 支持 JSON 和通过 JAXB 支持 XML 格式。可以通过创建自定义 HttpMessageConverters 的方式来添加其他的消息格式转换实现。

#### 静态文件
默认情况下，Spring Boot 可以对 “/static”、“/public”、“/resources” 或 “/META-INF/resources” 目录下的静态文件提供支持。同时 Spring Boot 还支持 Webjars。路径“/webjars/**”下的内容会由 webjar 格式的 Jar 包来提供。
