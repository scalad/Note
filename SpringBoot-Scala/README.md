### SpringBoot Scala

可以说近几年Spark的流行带动了Scala的发展，它集成了面向对象编程和函数式编程的各种特性，Scala具有更纯Lambda表粹的函数式业务逻辑解决方案，其语法比Java8后Lambda更加简洁方便，SpringBoot为spring提供了一种更加方便快捷的方式，不再要求写大量的配置文件，作为一名Scala爱好者，使用SpringBoot结合Scala将大大节省我们开发的时间以及代码量,让程序员可以花更多的时间关注业务逻辑。[项目源代码所在Github地址](https://github.com/silence940109/SpringBoot-Scala)
本文将使用SpringBoot+Scala+Gradle部署Web应用，如果你对Gradle不熟悉，或者IDE中没有Gradle，你可以使用Maven进行管理，原理上是一样的

1、首先是build.gradle文件
	
	buildscript {  
	    ext {  
	        springBootVersion = '1.4.3.RELEASE'  
	    }  
	    repositories {  
	        mavenCentral()  
	    }  
	    dependencies {  
	        classpath("org.springframework.boot:spring-boot-gradle-plugin:${springBootVersion}")  
	    }  
	}  
	apply plugin: 'java'  
	apply plugin: 'scala'  
	apply plugin: 'eclipse'  
	apply plugin: 'application'  
	apply plugin: 'org.springframework.boot'  
	  
	ext {  
	    sourceCompatibility = 1.8  
	    targetCompatibility = 1.8  
	}  
	jar {  
	    baseName = 'spring-boot-scala'  
	    version =  '0.1.0'  
	}  
	  
	repositories {  
	    mavenCentral()  
	    maven { url "https://repo.spring.io/snapshot" }  
	    maven { url "https://repo.spring.io/milestone" }  
	}  
	  
	dependencies {  
	    compile("org.springframework.boot:spring-boot-starter-web")  
	    compile("org.springframework.boot:spring-boot-starter-actuator")  
	    compile("org.springframework.boot:spring-boot-starter-data-jpa")  
	    compile "org.scala-lang:scala-library:2.11.7"  
	    compile "org.scala-lang:scala-compiler:2.11.7"  
	    compile "org.scala-lang:scala-reflect:2.11.7"  
	    testCompile("org.springframework.boot:spring-boot-starter-test")  
	    //mysql driver  
	    compile ("mysql:mysql-connector-java:5.1.38")  
	    testCompile("junit:junit:4.12")  
	}  
	  
	task wrapper(type: Wrapper) {  
	    gradleVersion = '2.9'  
	}  
	  
	bootRepackage.enabled = false  

这里面引入了spring-boot-starter-data-jpa，它依赖于hibernate-core和spring-data-jpa，两者的结合可以更好的帮我们处理数据库层的操作。

2、springboot的配置文件src/main/resources/application.properties

	#spring.datasource.url: jdbc:hsqldb:mem:scratchdb  
	#logging.file: /tmp/logs/app.log  
	  
	#datasource  
	spring.datasource.url = jdbc:mysql://localhost:3306/sss  
	spring.datasource.username = root  
	spring.datasource.password = root  
	spring.datasource.driverClassName = com.mysql.jdbc.Driver  
	  
	# Specify the DBMS  
	spring.jpa.database = MYSQL  
	# Show or not log for each sql query  
	spring.jpa.show-sql = true  
	# Hibernate ddl auto (create, create-drop, update)  
	spring.jpa.hibernate.ddl-auto = update  
	# stripped before adding them to the entity manager)  
	spring.jpa.properties.hibernate.dialect = org.hibernate.dialect.MySQL5Dialect  
	  
	# Server port  
	server.port=8080  

3、dao层代码src/main/scala/com/silence/repository/UserRepository.scala

	package com.silence.repository  
	  
	import com.silence.enties.User  
	import org.springframework.data.jpa.repository.JpaRepository  
	import java.lang.Long  
	  
	trait UserRepository extends JpaRepository[User, Long]  

Scala并没有Java概念中的Interface，不过Scala使用的trait特质来声明，注意，JpaRepository中的Long对象是来自java.lang.Long，而不是Scala特有的Long类型。

4、service层/src/main/scala/com/silence/service/BaseService.scala


```Scala
package com.silence.service  
  
import org.springframework.data.jpa.repository.JpaRepository  
import org.springframework.beans.factory.annotation.Autowired  
import scala.reflect.ClassTag  
import java.lang.Long  
import org.springframework.data.domain.Page  
import java.util.List  
import org.springframework.context.annotation.ComponentScan  
import org.springframework.stereotype.Service  
import org.springframework.context.annotation.Bean  
import javax.transaction.Transactional  
import java.lang.Boolean  
import org.springframework.data.domain.PageRequest  
  
@Service  
abstract class BaseService[T: ClassTag] {  
      
    /** spring data jpa dao*/  
    @Autowired val jpaRepository: JpaRepository[T, Long] = null  
      
    /** 
     * @description 添加记录 
     * @param S <: T  
     * @return T 
     */  
    def save[S <: T](s: S) : T = jpaRepository.save(s)  
      
    /** 
     * @description 根据Id删除数据 
     * @param id 数据Id 
     * @return Unit 
     */  
    @Transactional  
    def delete(id: Long): Unit = jpaRepository.delete(id)  
      
    /** 
     * @description 实体批量删除 
     * @param List[T] 
     * @return Unit 
     */  
    @Transactional  
    def delete(lists: List[T]) : Unit = jpaRepository.delete(lists);  
      
    /** 
     * @description 根据Id更新数据 
     * @param S <: T  
     * @return T 
     */  
    @Transactional  
    def update[S <: T](s: S) : T = jpaRepository.save(s)  
      
    /** 
     * @description 根据Id查询 
     * @param id 数据Id 
     * @return T 
     */  
    def find[S <: T](id: Long) : T = jpaRepository.findOne(id)  
      
    /** 
     * @description 查询所有数据 
     * @return List[T] 
     */  
    def findAll[S <: T]: List[T] = jpaRepository.findAll  
      
    /** 
     * @description 集合Id查询数据 
     * @return List[T] 
     */  
    def findAll[S <: T](ids: List[Long]): List[T] = jpaRepository.findAll(ids)  
      
    /** 
     * @description 统计大小 
     * @return Long 
     */  
    def count : Long = jpaRepository.count  
      
    /** 
     * @description 判断数据是否存在 
     * @param id 数据Id 
     * @return Boolean 
     */  
    def exist(id: Long) : Boolean = jpaRepository.exists(id)  
      
    /** 
     * @description 查询分页 
     * @param page 起始页 
     * @param pageSize 每页大小 
     * @return Page[T] 
     */  
    def page[S <: T](page: Int, pageSize: Int): Page[T] = {  
      var rpage = if (page < 1) 1 else page;  
      var rpageSize = if (pageSize < 1) 5 else pageSize;  
          jpaRepository.findAll(new PageRequest(rpage - 1, pageSize))  
    }      
  
}  

```
该类声明了针对DAO层的一系列操作，包括分页

5、UserService.scala


```Scala
package com.silence.service  
  
import com.silence.enties.User  
import org.springframework.beans.factory.annotation.Autowired  
import com.silence.repository.UserRepository  
import org.springframework.stereotype.Service  
  
@Service  
class UserService extends BaseService[User] {  
    
  @Autowired val userRepository: UserRepository = null  
    
} 

```
UserRepository是继承了JpaRepository接口的trait

6、src/main/scala/com/silence/controller/UserController.scala

```Scala

package com.silence.controller  
  
import org.springframework.stereotype.Controller  
import org.springframework.web.bind.annotation.RequestMapping  
import org.springframework.web.bind.annotation.RequestMethod  
import org.springframework.context.annotation.ComponentScan  
import org.springframework.context.annotation.Configuration  
import org.springframework.web.bind.annotation.ResponseBody  
import org.springframework.beans.factory.annotation.Autowired  
import com.silence.repository.UserRepository  
import org.springframework.web.servlet.ModelAndView  
import com.silence.enties.User  
import java.util.List  
import org.springframework.web.bind.annotation.PathVariable  
import javax.validation.Valid  
import org.springframework.validation.BindingResult  
import org.springframework.web.bind.annotation.RequestParam  
import org.springframework.data.domain.Pageable  
import org.springframework.data.domain.PageRequest  
import org.springframework.data.domain.Page  
import com.silence.service.UserService  
  
@ComponentScan  
@Controller  
@ResponseBody  
class UserController @Autowired()(private val userService : UserService){  
    
    @RequestMapping(value = Array("/list"), method = Array(RequestMethod.GET))  
  def list() : List[User] = {  
      userService.findAll  
  }  
    
  @RequestMapping(value = Array("save"), method = Array(RequestMethod.POST))  
  def save(@Valid user : User) : User = {  
      userService.save(user)  
  }  
    
    @RequestMapping(value = Array("/find/{id}"), method = Array(RequestMethod.GET))  
  def find(@PathVariable(value = "id") id: Long) : User = {  
      userService.find(id)  
  }  
    
  @RequestMapping(value = Array("delete/{id}"), method = Array(RequestMethod.POST))  
  def delete(@PathVariable(value = "id") id: Long) : Unit = {  
      userService.delete(id)  
  }  
    
  @RequestMapping(value = Array("update"), method = Array(RequestMethod.POST))  
  def update(@Valid user : User, bindingResult : BindingResult) : User = {  
      userService.update(user)  
  }  
    
  @RequestMapping(value = Array("page"), method = Array(RequestMethod.GET))  
  def page(@RequestParam("page") page : Int, @RequestParam("pageSize") pageSize : Int) : Page[User] = {  
      userService.page(page, pageSize)  
  }  
    
} 

```

7、SpringBoot启动src/main/scala/com/silence/SpringBootScalaApplication.scala

```Scala
package com.silence  
  
import org.springframework.context.annotation.Configuration  
import org.springframework.boot.autoconfigure.EnableAutoConfiguration  
import org.springframework.context.annotation.ComponentScan  
import org.springframework.boot.SpringApplication  
import org.springframework.boot.autoconfigure.SpringBootApplication  
  
@Configuration  
@EnableAutoConfiguration  
@ComponentScan  
@SpringBootApplication  
class Config  
  
  
object SpringBootScalaApplication extends App {  
  
  SpringApplication.run(classOf[Config])  
    
}  
```

8、运行，如果是使用IDE，可以直接在SpringBootScalaApplication中Run As -> Scala Application，或者在项目的根目录下使用命令行gradle bootRun
 
9、测试，可以使用浏览器打开测试，Linux可以使用crul

	curl http://localhost:8080/list  
	curl http://localhost:8080/find/2  
	curl -d "name=silence&telephone=15345678960&birthday=1994-06-24" http://localhost:8080/save  
	curl -d null http://localhost:8080/delete/505  
	curl http://localhost:8080/page?pageSize=6&page=5  

![](https://github.com/silence940109/Java/blob/master/SpringBoot-Scala/image/springboot-scala.jpg)

项目地址[github.com/silence940109/SpringBoot-Scala](github.com/silence940109/SpringBoot-Scala)