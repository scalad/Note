###SpringBoot Scala

可以说近几年Spark的流行带动了Scala的发展，它集成了面向对象编程和函数式编程的各种特性，Scala具有更纯Lambda表粹的函数式业务逻辑解决方案，其语法比Java8后Lambda更加简洁方便，SpringBoot为Spring提供了一种更加方便快捷的方式，不再要求写大量的配置文件，作为一名Scala爱好者，使用SpringBoot结合Scala将大大节省我们开发的时间以及代码量

`build.gradle`
	
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
	    compile("org.thymeleaf:thymeleaf-spring4")
	    compile("nz.net.ultraq.thymeleaf:thymeleaf-layout-dialect")
	    compile "org.scala-lang:scala-library:2.11.7"
	    compile "org.scala-lang:scala-compiler:2.11.7"
	    compile "org.scala-lang:scala-reflect:2.11.7"
	    testCompile("org.springframework.boot:spring-boot-starter-test")
	    testCompile("junit:junit")
	}
	
	task wrapper(type: Wrapper) {
	    gradleVersion = '2.9'
	}
	
	bootRepackage.enabled = false

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

```Scala

package com.silence.controller

import org.springframework.stereotype.Controller
import org.springframework.web.bind.annotation.RequestMapping
import org.springframework.web.bind.annotation.RequestMethod
import org.springframework.context.annotation.ComponentScan
import org.springframework.context.annotation.Configuration
import org.springframework.web.bind.annotation.ResponseBody

@ComponentScan
@Controller
class UserController {
  
  @ResponseBody
	@RequestMapping(value = Array("/index"), method = Array(RequestMethod.GET))
  def list(str : String) = {
	  print("hello world")
    "index"
  }
  
}

```

###build

	gradle bootRun

![](https://github.com/silence940109/Java/blob/master/SpringBoot-Scala/image/springboot-scala.jpg)

项目地址[github.com/silence940109/SpringBoot-Scala](github.com/silence940109/SpringBoot-Scala)