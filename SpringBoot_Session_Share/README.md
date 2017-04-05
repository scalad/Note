### Spring Boot Session共享 ###
分布式Web网站一般都会碰到集群session共享问题，之前也做过一些Spring3的项目，当时解决这个问题做过两种方案，一是利用nginx，session交给nginx控制，但是这个需要额外工作较多；还有一种是利用一些tomcat上的插件，修改tomcat配置文件，让tomcat自己去把Session放到Redis/Memcached/DB中去。这两种各有优缺，也都能解决问题

但是现在项目全线Spring Boot，并不自己维护Tomcat，而是由Spring去启动Tomcat。这样就会有一个问题：在服务器上并不存在一个持久存在的Tomcat程序，这样也无从去修改Tomcat的配置文件了。经过了一番搜索，发现Spring果然对这个问题有自己的解决方案，那就是Spring-Session

Spring-Session是通过过滤器实现的session共享，具体原理可以自己去官网查，这里只说一下如何配置。整个项目基于Spring Boot，如果不是Boot项目就需要自己去调整了。

项目需要先准备一个Redis服务，在本地启动一个即可。还需要有一个已经使用session但是未做session共享的Spring Boot项目，下面我就讲述一下如何给这个项目加上基于redis的session共享。

#### 引入依赖 ####
首先，要在maven中加入以下依赖：

```xml
<dependencies>
    <dependency>
            <groupId>org.springframework.session</groupId>
            <artifactId>spring-session</artifactId>
            <version>1.2.2.RELEASE</version>
    </dependency>
    <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-redis</artifactId>
    </dependency>
　　　　<dependency>
        <groupId>org.springframework.session</groupId>
        <artifactId>spring-session-data-redis</artifactId>
        <version>1.2.2.RELEASE</version>
        <type>pom</type>
        </dependency>
</dependencies>
```

#### 配置Redis ####
在项目目前在使用的properties文件中，加入如下配置：
	
	spring.redis.host=localhost
	spring.redis.password=secret
	spring.redis.port=6379

#### Spring配置 ####
在项目的目录中，创建一个Config.java文件（名称随意）

```Java
@Configuration
@EnableRedisHttpSession 
public class Config {

        @Bean
        public JedisConnectionFactory connectionFactory() {
                return new JedisConnectionFactory(); 
        }
}
```

@EnableRedisHttpSession这个注解就是最重要的东西，加了它之后，spring生产一个新的拦截器，用来实现Session共享的操作，具体实现这里暂不展开。而配置的这个Bean，则是让Spring根据配置文件中的配置连到Redis。

#### 测试 ####
首先我们开启两个tomcat服务，端口分别为8080和9090，在application.properties中进行设置

	server.port=8080

接下来定义一个Controller：

```Java
@RestController  
@RequestMapping(value = "/admin/v1")  
public class QuickRun {  
    @RequestMapping(value = "/first", method = RequestMethod.GET)  
    public Map<String, Object> firstResp (HttpServletRequest request){  
        Map<String, Object> map = new HashMap<>();  
        request.getSession().setAttribute("request Url", request.getRequestURL());  
        map.put("request Url", request.getRequestURL());  
        return map;  
    }  
  
    @RequestMapping(value = "/sessions", method = RequestMethod.GET)  
    public Object sessions (HttpServletRequest request){  
        Map<String, Object> map = new HashMap<>();  
        map.put("sessionId", request.getSession().getId());  
        map.put("message", request.getSession().getAttribute("map"));  
        return map;  
    }  
} 
```

启动之后进行访问测试，首先访问8080端口的tomcat，返回

> {"request Url":"http://localhost:8080/admin/v1/first"}  

接着，我们访问8080端口的sessions，返回：

> {"sessionId":"efcc85c0-9ad2-49a6-a38f-9004403776b5","message":"http://localhost:8080/admin/v1/first"}  

最后，再访问9090端口的sessions，返回：

> {"sessionId":"efcc85c0-9ad2-49a6-a38f-9004403776b5","message":"http://localhost:8080/admin/v1/first"}  

可见，8080与9090两个服务器返回结果一样，实现了session的共享

如果此时再访问9090端口的first的话，首先返回：

> {"request Url":"http://localhost:9090/admin/v1/first"}

而两个服务器的sessions都是返回：

> {"sessionId":"efcc85c0-9ad2-49a6-a38f-9004403776b5","message":"http://localhost:9090/admin/v1/first"}

     