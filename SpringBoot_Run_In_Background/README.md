### Spring Boot应用的后台运行配置 ###

酱油一篇，整理一下关于Spring Boot后台运行的一些配置方式。在介绍后台运行配置之前，我们先回顾一下Spring Boot应用的几种运行方式：

* 运行Spring Boot的应用主类
* 使用Maven的Spring Boot插件mvn spring-boot:run来运行
* 打成jar包后，使用java -jar运行

我们在开发的时候，通常会使用前两种，而在部署的时候往往会使用第三种。但是，我们在使用java -jar来运行的时候，并非后台运行。下面我们分别针对Windows和Linux/Unix两种环境，整理一下如何配置后台运行的方法。

#### Windows ####
Windows下比较简单，我们可以直接使用这款软件：AlwaysUp。如下图所示，简单、暴力、好用。

![](https://github.com/scalad/Note/blob/master/SpringBoot_Run_In_Background/image/13213400_9s9B.png)

配置方式很简单，我们只需要把Spring Boot应用通过mvn install打成jar包，然后编写一个java -jar yourapp.jar的bat文件。再打开AlwaysUp，点击工具栏的第一个按钮，如下图所示，选择上面编写的bat文件，并填写服务名称。

![](https://github.com/scalad/Note/blob/master/SpringBoot_Run_In_Background/image/13213417_TI8z.png)

完成了创建之后，在列表中可以看到我们配置的服务，通过右键选择Start xxx就能在后台将该应用启动起来了

#### Linux/Unix ####
下面我们来说说服务器上该如何来配置。实际上，实现的方法有很多种，这里就列两种还比较好用的方式：

nohup和Shell

该方法主要通过使用nohup命令来实现，该命令的详细介绍如下：

nohup 命令

	用途：不挂断地运行命令。
	
	语法：nohup Command [ Arg … ][ & ]
	
	描述：nohup 命令运行由 Command 参数和任何相关的 Arg 参数指定的命令，忽略所有挂断（SIGHUP）信号。在注销后使用 nohup 命令运行后台中的程序。要运行后台中的 nohup 命令，添加 &到命令的尾部。

所以，我们只需要使用nohup java -jar yourapp.jar &命令，就能让yourapp.jar在后台运行了。但是，为了方便管理，我们还可以通过Shell来编写一些用于启动应用的脚本，比如下面几个：

*　关闭应用的脚本：stop.sh

```shell
#!/bin/bash
PID=$(ps -ef | grep yourapp.jar | grep -v grep | awk '{ print $2 }')
if [ -z "$PID" ]
then
    echo Application is already stopped
else
    echo kill $PID
    kill $PID
fi
```

*　启动应用的脚本：start.sh

```shell
#!/bin/bash
nohup java -jar yourapp.jar --server.port=8888 &
```

* 整合了关闭和启动的脚本：run.sh，由于会先执行关闭应用，然后再启动应用，这样不会引起端口冲突等问题，适合在持续集成系统中进行反复调用。

```shell
#!/bin/bash
echo stop application
source stop.sh
echo start application
source start.sh
```

#### 系统服务 ####
在Spring Boot的Maven插件中，还提供了构建完整可执行程序的功能，什么意思呢？就是说，我们可以不用java -jar，而是直接运行jar来执行程序。这样我们就可以方便的将其创建成系统服务在后台运行了。主要步骤如下：

* 在pom.xml中添加Spring Boot的插件，并注意设置executable配置

```xml
<build> 
  <plugins> 
    <plugin> 
      <groupId>org.springframework.boot</groupId>  
      <artifactId>spring-boot-maven-plugin</artifactId>  
      <configuration> 
        <executable>true</executable> 
      </configuration> 
    </plugin> 
  </plugins> 
</build>
```

* 在完成上述配置后，使用mvn install进行打包，构建一个可执行的jar包

* 创建软连接到/etc/init.d/目录下

	sudo ln -s /var/yourapp/yourapp.jar /etc/init.d/yourapp

* 在完成软连接创建之后，我们就可以通过如下命令对yourapp.jar应用来控制启动、停止、重启操作了

	/etc/init.d/yourapp start|stop|restart

原文地址：[http://blog.didispace.com/spring-boot-run-backend/](http://blog.didispace.com/spring-boot-run-backend/)
