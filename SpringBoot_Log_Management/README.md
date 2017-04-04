### Spring Boot日志管理 ###

Spring Boot在所有内部日志中使用[Commons Logging](http://commons.apache.org/proper/commons-logging/)，但是默认配置也提供了对常用日志的支持，如：[Java Util Logging](http://docs.oracle.com/javase/7/docs/api/java/util/logging/package-summary.html)，[Log4J](http://logging.apache.org/log4j/), [Log4J2](http://logging.apache.org/log4j/)和[Logback](http://logback.qos.ch/)。每种Logger都可以通过配置使用控制台或者文件输出日志内容。

#### 格式化日志 ####

默认的日志输出如下：

	2017-04-04 09:58:22.233  INFO 5972 --- [           main] com.silence.Application$                 : Starting Application. on silence with PID 5972 (E:\github\LayIM\bin started by asus in E:\github\LayIM)
	2017-04-04 09:58:22.241  INFO 5972 --- [           main] com.silence.Application$                 : No active profile set, falling back to default profiles: default
	2017-04-04 09:58:22.490  INFO 5972 --- [           main] ationConfigEmbeddedWebApplicationContext : Refreshing org.springframework.boot.context.embedded.AnnotationConfigEmbeddedWebApplicationContext@74235045: startup date [Tue Apr 04 09:58:22 CST 2017]; root of context hierarchy


输出内容元素具体如下：

* 时间日期 — 精确到毫秒
* 日志级别 — ERROR, WARN, INFO, DEBUG or TRACE
* 进程ID
* 分隔符 — --- 标识实际日志的开始
* 线程名 — 方括号括起来（可能会截断控制台输出）
* Logger名 — 通常使用源代码的类名
* 日志内容

#### 控制台输出 ####

在Spring Boot中默认配置了ERROR、WARN和INFO级别的日志输出到控制台。

我们可以通过两种方式切换至DEBUG级别：

* 在运行命令后加入--debug标志，如：$ java -jar myapp.jar --debug
* 在application.properties中配置debug=true，该属性置为true的时候，核心Logger（包含嵌入式容器、hibernate、spring）会输出更多内容，但是你自己应用的日志并不会输出为DEBUG级别。

#### 多彩输出 ####

如果你的终端支持ANSI，设置彩色输出会让日志更具可读性。通过在`application.properties`中设置`spring.output.ansi.enabled`参数来支持。

* NEVER：禁用ANSI-colored输出（默认项）
* DETECT：会检查终端是否支持ANSI，是的话就采用彩色输出（推荐项）
* ALWAYS：总是使用ANSI-colored格式输出，若终端不支持的时候，会有很多干扰信息，不推荐使用

#### 文件输出 ####

Spring Boot默认配置只会输出到控制台，并不会记录到文件中，但是我们通常生产环境使用时都需要以文件方式记录

若要增加文件输出，需要在`application.properties`中配置`logging.file`或`logging.path`属性

* logging.file，设置文件，可以是绝对路径，也可以是相对路径。如：logging.file=my.log
* logging.path，设置目录，会在该目录下创建spring.log文件，并写入日志内容，如：logging.path=/var/log

日志文件会在10Mb大小的时候被截断，产生新的日志文件，默认级别为：ERROR、WARN、INFO

#### 级别控制 ####

在Spring Boot中只需要在application.properties中进行配置完成日志记录的级别控制。

配置格式：logging.level.*=LEVEL

* logging.level：日志级别控制前缀，*为包名或Logger名
* LEVEL：选项TRACE, DEBUG, INFO, WARN, ERROR, FATAL, OFF

举例：

	logging.level.com.didispace=DEBUG：com.didispace包下所有class以DEBUG级别输出
	logging.level.root=WARN：root日志以WARN级别输出

> 注意，如果想要输出Mybatis的SQL语句可以logging.level.com.silence.mapper=debug

#### 自定义日志配置 ####
由于日志服务一般都在ApplicationContext创建前就初始化了，它并不是必须通过Spring的配置文件控制。因此通过系统属性和传统的Spring Boot外部配置文件依然可以很好的支持日志控制和管理。

根据不同的日志系统，你可以按如下规则组织配置文件名，就能被正确加载：

* Logback：logback-spring.xml, logback-spring.groovy, logback.xml, logback.groovy
* Log4j：log4j-spring.properties, log4j-spring.xml, log4j.properties, log4j.xml
* Log4j2：log4j2-spring.xml, log4j2.xml
* JDK (Java Util Logging)：logging.properties

Spring Boot官方推荐优先使用带有-spring的文件名作为你的日志配置（如使用logback-spring.xml，而不是logback.xml）

#### 自定义输出格式 ####
在Spring Boot中可以通过在application.properties配置如下参数控制输出格式：

* logging.pattern.console：定义输出到控制台的样式（不支持JDK Logger）
* logging.pattern.file：定义输出到文件的样式（不支持JDK Logger）