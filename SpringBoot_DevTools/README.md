### DevTools in Spring Boot 热部署 ###

本文主要了解Spring Boot 1.3.0新添加的spring-boot-devtools模块的使用，该模块主要是为了提高开发者开发Spring Boot应用的用户体验。

要想使用该模块需要在Maven中添加：

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-devtools</artifactId>
    </dependency>
</dependencies>
```

或者在Gradle配置文件中添加：

```groovy
dependencies {
    compile("org.springframework.boot:spring-boot-devtools")
}
```

#### 1、默认属性 ####

在Spring Boot集成Thymeleaf时，`spring.thymeleaf.cache`属性设置为false可以禁用模板引擎编译的缓存结果。现在，devtools会自动帮你做到这些，禁用所有模板的缓存，包括Thymeleaf, Freemarker, Groovy Templates, Velocity, Mustache等。更多的属性，请参考[DevToolsPropertyDefaultsPostProcessor。](http://github.com/spring-projects/spring-boot/tree/v1.3.2.RELEASE/spring-boot-devtools/src/main/java/org/springframework/boot/devtools/env/DevToolsPropertyDefaultsPostProcessor.java)

#### 2、自动重启 ####

你可能使用过 JRebel 或者 Spring Loaded来自动重启应用，现在只需要引入devtools就可以了，当代码变动时，它会自动进行重启应用。当然，你也可以使用插件来实现，例如在maven中配置插件：

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <configuration>
                <fork>true</fork>
            </configuration>
        </plugin>
    </plugins>
</build>
```

```groovy
bootRun {
    addResources = true
}
```

DevTools在重启过程中依赖于application context的shutdown hook，如果设置`SpringApplication.setRegisterShutdownHook(false)`，则自动重启将不起作用。

#### 排除静态资源文件 ####
静态资源文件在改变之后有时候没必要触发应用程序重启，例如，Thymeleaf的模板会被变量替代。默认的，/META-INF/maven、/META-INF/resources、/resources、/static、/public或者/templates这些目录下的文件修改之后不会触发重启但是会触发LiveReload。可以通过`spring.devtools.restart.exclude`属性来修改默认值

如果你想保留默认配置，并添加一些额外的路径，可以使用`spring.devtools.restart.additional-exclude`属性

#### 观察额外的路径 ####
如果你想观察不在classpath中的路径的文件变化并触发重启，则可以配置 spring.devtools.restart.additional-paths 属性

#### 关闭自动重启 ####
设置 spring.devtools.restart.enabled 属性为false，可以关闭该特性。可以在application.properties中设置，也可以通过设置环境变量的方式

```java
public static void main(String[] args) {
    System.setProperty("spring.devtools.restart.enabled", "false");
    SpringApplication.run(MyApp.class, args);
}
```

#### 使用一个触发文件 ####
通过设置`spring.devtools.restart.trigger-file`属性指定一个文件，当该文件被修改时，则触发自动重启

#### 自定义自动重启类加载器 ####
Spring Boot自动重启使用的是两个类加载器，大多数情况下工作良好，有时候会出现问题。

默认的，IDE中打开的项目会使用 restart 类加载器进行加载，而任何其他的 .jar 文件会使用 base 类加载器进行加载。如果你使用的是多模块的项目，并且有些模块没有被导入到IDE，你需要创建并编辑`META-INF/spring-devtools.properties`文件来自定义一些配置

`spring-devtools.properties`文件包含有 `restart.exclude`. 和 `restart.include`. 前缀的属性。include属性文件的都会被加入到restart类加载器，exclude属性文件的都会被加入到base类加载器，他们的值是正则表达式，所有的属性值必须是唯一的

例如：

	restart.include.companycommonlibs=/mycorp-common-[\\w-]+\.jar
	restart.include.projectcommon=/mycorp-myproj-[\\w-]+\.jar

> classpath中的所有META-INF/spring-devtools.properties文件都会被加载。

#### 已知的限制 ####

如果对象是使用ObjectInputStream进行反序列化，则自动重启将不可用。如果你需要反序列化对象，则你需要使用spring的`ConfigurableObjectInputStream`并配合`Thread.currentThread().getContextClassLoader()`进行反序列化

#### 3、LiveReload ####

在浏览器方面，DevTools内置了一个LiveReload服务，可以自动刷新浏览器。如果你使用JRebel，则自动重启将会失效，取而代之的是使用动态加载类文件。当然，其他的DevTools（例如LiveReload和属性覆盖）特性还能使用。

该特性可以通过`spring.devtools.livereload.enabled`属性来设置是否开启

#### 4、全局设置 ####
可以在 $HOME 目录下创建一个`.spring-boot-devtools.properties`文件来设置全局的配置。

例如，设置一个触发文件类触发重启：

	spring.devtools.reload.trigger-file=.reloadtrigger

#### 5、远程应用 ####
DevTools不仅可以用于本地应用，也可以用于远程应用。通过设置 spring.devtools.remote.secret 属性可以开启远程应用。

远程DevTools支持包括两个部分，服务端应用和你IDE中运行的本地客户端应用。当设置spring.devtools.remote.secret属性之后，服务端应用自动开启DevTools特性，客户端程序需要手动启动。

#### 运行远程客户端应用 ####

远程客户端应用被设计来运行在你的IDE中。你需要使用和你连接的远程应用相同的classpath来运行org.springframework.boot.devtools.RemoteSpringApplication类，传递给该类的必选参数是你连接的应用的url。

例如，如果你在使用Eclipse或者STS，并且你有一个 my-app 应用部署在Cloud Foundry，你可以按照以下步骤操作：

* 从Run菜单运行Run Configurations…`
* 创建一个Java Application的启动配置
* 浏览my-app项目
* 使用org.springframework.boot.devtools.RemoteSpringApplication作为main类
* 添加https://myapp.cfapps.io到the Program arguments

一个运行中的远程客户端将会是这样子：

	 .   ____          _                                              __ _ _
	 /\\ / ___'_ __ _ _(_)_ __  __ _          ___               _      \ \ \ \
	( ( )\___ | '_ | '_| | '_ \/ _` |        | _ \___ _ __  ___| |_ ___ \ \ \ \
	 \\/  ___)| |_)| | | | | || (_| []::::::[]   / -_) '  \/ _ \  _/ -_) ) ) ) )
	  '  |____| .__|_| |_|_| |_\__, |        |_|_\___|_|_|_\___/\__\___|/ / / /
	 =========|_|==============|___/===================================/_/_/_/
	 :: Spring Boot Remote :: 1.3.2.RELEASE
	
	2015-06-10 18:25:06.632  INFO 14938 --- [           main] o.s.b.devtools.RemoteSpringApplication   : Starting RemoteSpringApplication on pwmbp with PID 14938 (/Users/pwebb/projects/spring-boot/code/spring-boot-devtools/target/classes started by pwebb in /Users/pwebb/projects/spring-boot/code/spring-boot-samples/spring-boot-sample-devtools)
	2015-06-10 18:25:06.671  INFO 14938 --- [           main] s.c.a.AnnotationConfigApplicationContext : Refreshing org.springframework.context.annotation.AnnotationConfigApplicationContext@2a17b7b6: startup date [Wed Jun 10 18:25:06 PDT 2015]; root of context hierarchy
	2015-06-10 18:25:07.043  WARN 14938 --- [           main] o.s.b.d.r.c.RemoteClientConfiguration    : The connection to http://localhost:8080 is insecure. You should use a URL starting with 'https://'.
	2015-06-10 18:25:07.074  INFO 14938 --- [           main] o.s.b.d.a.OptionalLiveReloadServer       : LiveReload server is running on port 35729
	2015-06-10 18:25:07.130  INFO 14938 --- [           main] o.s.b.devtools.RemoteSpringApplication   : Started RemoteSpringApplication in 0.74 seconds (JVM running for 1.105)

说明：

* 因为远程客户端和远程应用使用的是相同的classpath，所以远程客户端可以直接读取应用的配置文件。所以spring.devtools.remote.secret属性能够被读取到并传递给服务端进行校验。
* 远程url建议使用更加安全的https://协议
* 如果需要使用代理，则需要设置`spring.devtools.remote.proxy.host`和`spring.devtools.remote.proxy.port`属性。

#### 远程更新 ####

远程客户端会监控你的应用的classpath的改变和本地重启的方式一样。任何更新的资源将会被推送到远程应用并且（如果有必要）触发一个重启。这在你集成一个使用云服务的本地不存在的特性时会是非常有用的。通常远程的更新和重启比一个完整的重新编译和部署周期会快的多。

> 只有在远程客户端运行的过程中，文件才会被监控。如果你在启动远程客户端之前，修改一个文件，其将不会被推送到远程应用。

#### 远程调试 ####

远程调试默认使用的端口是8000，你可以通过spring.devtools.remote.debug.local-port来修改。

你可以通过查看JAVA_OPTS来看远程调试是否被启用，主要是观察是否有-Xdebug -Xrunjdwp:server=y,transport=dt_socket,suspend=n参数

