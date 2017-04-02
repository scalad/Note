# SpringBoot-RabbitMQ 消息队列

这个指南将引导你建立一个RabbitMQ AMQP服务器发布和订阅消息的过程。

### 声明
可以使用本人阿里云安装好的RabbitMQ服务器
	
	host:http://120.27.114.229
	username:root
	password:root
	web management: http://120.27.114.229:15672

### 构建
你会使用 Spring AMQP的 RabbitTemplate构建应用系统来发布消息并且使用一个MessageListenerAdapter POJO来订阅消息

### 需要
* 15分钟
* 一款文本编辑器或者IDE
* [JDK 1.8+](http://www.oracle.com/technetwork/java/javase/downloads/index.html)
* [Gradle2.3+](http://www.gradle.org/downloads) 或者[Maven3.0+](http://maven.apache.org/download.cgi)
* 你也可以从这个项目中导入代码或者可以在导入[Spring Tool Suite(STS)](https://spring.io/guides/gs/sts)(个人非常喜欢的一款eclipse的IDE)中查看
* RabbitMQ服务器

### 如何完成
像许多的Spring [Getting Started guides](https://spring.io/guides)项目,你可以从头开始并完成每一步，或者你可以绕过你已经熟悉的一些步骤，无论是哪种步骤，你最终可以完成代码

从头开始的话，请去看[使用Gradle构建](https://spring.io/guides/gs/messaging-rabbitmq/#scratch)

如果要绕过你熟悉的，按照以下构建：

* [下载](https://github.com/spring-guides/gs-messaging-rabbitmq/archive/master.zip)并解压得到源代码或者从[Git](https://spring.io/understanding/Git)：
	>git clone https://github.com/spring-guides/gs-messaging-rabbitmq.git
* 进入`gs-messaging-rabbitmq/initial`目录
* 跳过[创建RabbitMQ消息接收]()

当你完成时，你可以对比在`gs-messaging-rabbitmq/complete.`目录中的结果和你的结果

### 使用Gradle构建
第一步你需要建立一个基本的脚本，当你构建APP应用时你可以任何你喜欢的构建系统，但这些代码你必须要使用到[Gradle](http://gradle.org/)和[Maven](https://maven.apache.org/),如果你对这两个不熟悉，你可以参考[Building Java Projects with Gradle](https://spring.io/guides/gs/gradle)和[Building Java Projects with Maven](https://spring.io/guides/gs/maven)

#### 1.创建目录结构
在你项目的文件夹中创建如下的子目录结构，例如，在*nix系统中使用命令创建`mkdir -p src/main/java/hello`

	└── src
	    └── main
	        └── java
	            └── hello

#### 2.创建Gradle配置文件build.gradle
以下来自[初始化Gradle配置文件](https://github.com/silence940109/SpringBoot-RabbitMQ/blob/master/build.gradle)

`build.gradle`

	buildscript {
	    repositories {
	        mavenCentral()
	    }
	    dependencies {
	        classpath("org.springframework.boot:spring-boot-gradle-plugin:1.4.3.RELEASE")
	    }
	}
	
	apply plugin: 'java'
	apply plugin: 'eclipse'
	apply plugin: 'idea'
	apply plugin: 'org.springframework.boot'
	
	jar {
	    baseName = 'gs-messaging-rabbitmq'
	    version =  '0.1.0'
	}
	
	repositories {
	    mavenCentral()
	}
	
	sourceCompatibility = 1.8
	targetCompatibility = 1.8
	
	dependencies {
	    compile("org.springframework.boot:spring-boot-starter-amqp")
	    testCompile("junit:junit")
	}

[Spring Boot gradle plugin](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-tools/spring-boot-gradle-plugin)提供了很多方便的特性：

* 它集成了所有在类路径下的jar包并构建成单独的jar包，可执行的`über-jar`使它可以更加方便的执行和在你的服务中进行传输
* 它为`public static void main()`方法寻找可执行的类作为标志
* 它提供了一个内置的依赖解析器来匹配[Spring Boot Dependencies](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-dependencies/pom.xml)依赖版本号，你可以重写任何你希望的版本，但它默认启动时选择的版本集合

### 使用Maven构建
第一步你需要建立一个基本的脚本，当你构建APP应用时你可以任何你喜欢的构建系统，但这些代码你必须要使用到[Maven](https://maven.apache.org/),如果你对Maven不熟悉，你可以参考[Building Java Projects with Maven](https://spring.io/guides/gs/maven)

#### 1.创建目录结	构
在你项目的文件夹中创建如下的子目录结构，例如，在*nix系统中使用命令创建`mkdir -p src/main/java/hello`

	└── src
	    └── main
	        └── java
	            └── hello

`pom.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>org.springframework</groupId>
    <artifactId>gs-messaging-rabbitmq</artifactId>
    <version>0.1.0</version>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.4.3.RELEASE</version>
    </parent>
    <properties>
        <java.version>1.8</java.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-amqp</artifactId>
        </dependency>
    </dependencies>
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
``` 
[Spring Boot gradle plugin](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-tools/spring-boot-gradle-plugin)提供了很多方便的特性：

* 它集成了所有在类路径下的jar包并构建成单独的jar包，可执行的`über-jar`使它可以更加方便的执行和在你的服务中进行传输
* 它为`public static void main()`方法寻找可执行的类作为标志
* 它提供了一个内置的依赖解析器来匹配[Spring Boot Dependencies](https://github.com/spring-projects/spring-boot/blob/master/spring-boot-dependencies/pom.xml)依赖版本号，你可以重写任何你希望的版本，但它默认启动时选择的版本集合

### 使用IDE编译
#### 1.建立RabbitMQ沙箱
在你可以构建你的消息应用前，你需要建发布和订阅消息的服务器

RabbitMQ是一个AMQP(Advanced Message Queuing Protocol,一个提供统一消息服务的应用层标准高级消息队列协议,是应用层协议的一个开放标准,为面向消息的中间件设计)服务器,这个服务器是免费的，你可以在[http://www.rabbitmq.com/download.html](http://www.rabbitmq.com/download.html),你可以手动的下载，或者如果你使用的Mac可以自己制作

	brew install rabbitmq

打开服务器位置并使用默认的配置进行启动

	rabbitmq-server

你可以看到如下的一些输出信息:
#	
	            RabbitMQ 3.1.3. Copyright (C) 2007-2013 VMware, Inc.
	##  ##      Licensed under the MPL.  See http://www.rabbitmq.com/
	##  ##
	##########  Logs: /usr/local/var/log/rabbitmq/rabbit@localhost.log
	######  ##        /usr/local/var/log/rabbitmq/rabbit@localhost-sasl.log
	##########
	            Starting broker... completed with 6 plugins.

#
如果你有运行在本地的docker 你也可以使用[Docker Compose](https://docs.docker.com/compose/)(一个部署多个容器的简单但是非常必要的工具)来快速的启动RabbitMQ服务器，在这个项目的根目录中有一个`docker-compose.yml`，它非常简单：

`docker-compose.yml`

	rabbitmq:
	  image: rabbitmq:management
	  ports:
	    - "5672:5672"
	    - "15672:15672"
	   
如果这个文件在你的当前目录中你可以运行`docker-compose up`来是RabbitMQ运行在容器中

#### 2.创建RabbitMQ消息订阅
任何基于消息的应用程序你都需要创建一个消息订阅来响应消息的发布

`src/main/java/hello/Receiver.java`

```Java
package hello;

import java.util.concurrent.CountDownLatch;
import org.springframework.stereotype.Component;

@Component
public class Receiver {

    private CountDownLatch latch = new CountDownLatch(1);

    public void receiveMessage(String message) {
        System.out.println("Received <" + message + ">");
        latch.countDown();
    }

    public CountDownLatch getLatch() {
        return latch;
    }

}
```

定义一个简单的`Receiver`类，类中receiveMessage方法用来接收消息，当你注册接受消息时，你可以随意命名它

>为了方便，这个POJO类有一个`CountDownLatch`类的属性，它允许当消息接收到时给它一个信号量，这是你在生产环境中你不太可能实现的

#### 3.注册监听并发布消息
Spring AMQP的`RabbitTemplate`提供了你使用RabbitMQ发布和订阅消息所需要的一切，特别的，你需要如下配置：

* 一个消息监听的容器
* 声明队列，交换空间，并且绑定他们
* 一个发送一些信息用来测试监听的组件

>Spring Boot会自动创建连接工场和RabbitTemplate，以便减少你需要编写的代码量

你将会使用`RabbitTemplate`来发送消息，并且你要使用消息监听的容器注册一个`Receiver`来接收消息。连接工场驱动使它们可以连接到RabbitMQ服务器上

`src/main/java/hello/Application.java`

```Java
package hello;

import org.springframework.amqp.core.Binding;
import org.springframework.amqp.core.BindingBuilder;
import org.springframework.amqp.core.Queue;
import org.springframework.amqp.core.TopicExchange;
import org.springframework.amqp.rabbit.connection.ConnectionFactory;
import org.springframework.amqp.rabbit.listener.SimpleMessageListenerContainer;
import org.springframework.amqp.rabbit.listener.adapter.MessageListenerAdapter;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;

@SpringBootApplication
public class Application {

    final static String queueName = "spring-boot";
    
    final static String HOST = "120.27.114.229";

    final static String USERNAME = "root";
    
    final static String PASSWORD = "root";

    final static int PORT = 15672;
    
    @Bean
    Queue queue() {
        return new Queue(queueName, false);
    }

    @Bean
    TopicExchange exchange() {
        return new TopicExchange("spring-boot-exchange");
    }

    @Bean
    Binding binding(Queue queue, TopicExchange exchange) {
        return BindingBuilder.bind(queue).to(exchange).with(queueName);
    }

    @Bean  
    public ConnectionFactory connectionFactory() {  
    	  CachingConnectionFactory connectionFactory = new CachingConnectionFactory();
          connectionFactory.setHost(HOST);
          connectionFactory.setPort(PORT);
          connectionFactory.setUsername(USERNAME);
          connectionFactory.setPassword(PASSWORD);
          connectionFactory.setVirtualHost("/");
          //必须要设置,消息的回掉
          connectionFactory.setPublisherConfirms(true); 
          return connectionFactory;
    } 

    @Bean
    SimpleMessageListenerContainer container(ConnectionFactory connectionFactory,
            MessageListenerAdapter listenerAdapter) {
        SimpleMessageListenerContainer container = new SimpleMessageListenerContainer();
        container.setConnectionFactory(connectionFactory);
        container.setQueueNames(queueName);
        container.setMessageListener(listenerAdapter);
        return container;
    }

    @Bean
    MessageListenerAdapter listenerAdapter(Receiver receiver) {
        return new MessageListenerAdapter(receiver, "receiveMessage");
    }

    public static void main(String[] args) throws InterruptedException {
        SpringApplication.run(Application.class, args);
    }

}
```

`@SpringBootApplication`是一个非常方便的注解，它添加了所有如下的东西：

* `@Configuration`标志着这个类作为资源被这个应用程序而定义的一个bean
* `@EnableAutoConfiguration`告诉Spring Boot启动自动添加bean是基于类路径配置，其他Beans以及各种属性的设置
* 通常你会为Spring MVC的应用程序添加`@EnableWebMvc`注解，但是当Spring Boot看见Spring-webmvc在它的类路径下时它会自动添加，这个标志着这个应用程序是一个web应用程序，并且自动激活并配置例如`DispatcherServlet`的配置
* `@ComponentScan`告诉Spring去寻找其他的组件，配置以及在hello包中的其他services，并且允许它使用controllers组件

`main()`方法是用了Spring Boot的`SpringAPplication.run()`方法来启动一个应用程序，你注意到这里没有使用XML了吗？同样没有web.xml配置文件。这个web应用程序时纯粹的Java开发并且你不必处理任何配置信息

定义在`listenerAdapter()`bean方法在定义`container()`容器时注册成为一个消息监听器，它会为"spring-boot"的消息队列进行监听。因为`Receiver`是一个POJO，在你指定它被`receiveMessage`调用时，它需要被包装到`MessageListenerAdapter`适配器中

>JMS队列和AMQP队列有一些语义上的不同。例如，JMS向队列发送消息时只有一个消费者，然而AMQP队列做同样的事情，它虽然模仿JMS主题的概念，但AMQP生产者并不向队列直接发送消息，反而消息发送给的是交换空间，所以AMQP的消息它可以放到一个队列中，或者展开多个队列中。[更多](https://spring.io/understanding/AMQP)

消息监听容器和接收都是你为了监听到消息所必需的，为了发布一个消息，你也需要一个Rabbit模板

`queue()`方法创建了一个AMQP的队列，`exchang()`方法创建了exchange,`binding()`方法把他们两个绑定在了一起，并且定义了当RabbitTemplate发布给exchange时发生的动作

>Spring AMQP要求`Queue`,`TopicExchang`,`Binding`被Spring按照顺序定义为定理的bean

#### 4.发送文本消息
测试消息是通过`CommandLineRunner`发送的，它也可以等待并锁定接受者并且关闭应用程序：

`src/main/java/hello/Runner.java`

```Java
package hello;

import java.util.concurrent.TimeUnit;

import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.boot.CommandLineRunner;
import org.springframework.context.ConfigurableApplicationContext;
import org.springframework.stereotype.Component;

@Component
public class Runner implements CommandLineRunner {

    private final RabbitTemplate rabbitTemplate;
    private final Receiver receiver;
    private final ConfigurableApplicationContext context;

    public Runner(Receiver receiver, RabbitTemplate rabbitTemplate,
            ConfigurableApplicationContext context) {
        this.receiver = receiver;
        this.rabbitTemplate = rabbitTemplate;
        this.context = context;
    }

    @Override
    public void run(String... args) throws Exception {
        System.out.println("Sending message...");
        rabbitTemplate.convertAndSend(Application.queueName, "Hello from RabbitMQ!");
        receiver.getLatch().await(10000, TimeUnit.MILLISECONDS);
        context.close();
    }

}
```

runner可以在测试中进行模拟，以此，reveive可以单独的进行测试

#### 5.启动应用
`main()`方法通过创建Spring应用环境来启动进程。这个进程启动了消息监听容器，它会开始监听消息.`Runner`bean会自动执行：它从应用上下文中检索`RabbitTemplate`并且往"sping-boot"队列中发送一个`Hello from RabbitMQ!`的消息，最后，它关闭Spring应用程序，程序结束

#### 6.编译可执行的JAR包
你可以使用Gradle或者Maven从命令行运行程序，或者你可以编译成一个包含了所有的依赖，类和资源的可执行的JAR文件，然后就可以直接运行。这使它在不同的环境和在整个应用程序的开发声明周期的部署中变得非常容易

如果你使用的时Gradle,你需要使用`./gradlew bootRun`来运行应用程序，或者你可以使用`./gradlew build`编译成JAR文件，然后你就可以运行JAR文件了

	java -jar build/libs/gs-messaging-rabbitmq-0.1.0.jar

如果你使用的时Maven,你需要使用`./mvnw spring-boot:run`来运行应用程序，或者你可以使用`./mvnw clean package`编译成JAR文件，然后你就可以运行JAR文件了

	java -jar target/gs-messaging-rabbitmq-0.1.0.jar

>上面的结果中会创建一个可执行的JAR文件，你也可以选择[构建一个典型的war文件](https://spring.io/guides/gs/convert-jar-to-war/)

然后你就可以看到如下的输出：

	Sending message...
	Received <Hello from RabbitMQ!>


![](https://github.com/silence940109/Java/blob/master/SpringBoot-RabbitMQ/image/run.jpg)