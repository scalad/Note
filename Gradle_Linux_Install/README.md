### Gradle Linux安装
#### 使用SDKMAN安装

sdkman(The Software Development Kit Manager), 中文名为:软件开发工具管理器．这个工具的主要用途是用来解决在类unix操作系统(如mac, Linux等)中多种版本开发工具的切换, 安装和卸载的工作．对于windows系统的用户可以使用Powershell CLI来体验．

例如: 项目A使用Jdk7中某些特性在后续版本中被移除（尽管这是不好的设计），项目B使用Jdk8,我们在切换开发这两个项目的时候，需要不断的切换系统中的JAVA_PATH,这样很不方便，如果存在很多个类似的版本依赖问题，就会给工作带来很多不必要的麻烦． 
　　 
sdkman这个工具就可以很好的解决这类问题，它的工作原理是自己维护多个版本，当用户需要指定版本时，sdkman会查询自己所管理的多版本软件中对应的版本号，并将它所在的路径设置到系统PATH.

直接打开终端,执行如下命令:

	curl -s http://get.sdkman.io | bash

上面的命令的含义: 首先sdkman官网下载对应的安装shell script，然后调用bash解析器去执行．

![](https://github.com/silence940109/Java/blob/master/Gradle_Linux_Install/image/sdk1.jpg)
![](https://github.com/silence940109/Java/blob/master/Gradle_Linux_Install/image/sdk2.jpg)

可以通过输入sdk help或sdk version确认安装是否完成

	root@iZ286714gzoZ:~# sdk version
	SDKMAN 5.1.18+191

	sroot@iZ286714gzoZ:~# sdk help
	Usage: sdk <command> [candidate] [version]
	       sdk offline <enable|disable>
	   commands:
	       install   or i    <candidate> [version]
	       uninstall or rm   <candidate> <version>
	       list      or ls   [candidate]
	       use       or u    <candidate> [version]
	       default   or d    <candidate> [version]
	       current   or c    [candidate]
	       upgrade   or ug   [candidate]
	       version   or v
	       broadcast or b
	       help      or h
	       offline           [enable|disable]
	       selfupdate        [force]
	       flush             <candidates|broadcast|archives|temp>
	   candidate  :  the SDK to install: groovy, scala, grails, gradle, kotlin, etc.
	                 use list command for comprehensive list of candidates
	                 eg: $ sdk list
	   version    :  where optional, defaults to latest stable if not provided
	                 eg: $ sdk install groovy


#### 安装指定版本的gradle
打开一个新的终端执行以下命令安装指定版本的gradle
		
	$ sdk install gradle 3.3

![](https://github.com/silence940109/Java/blob/master/Gradle_Linux_Install/image/install.jpg)
#### 移除安装的gradle

	 sdk uninstall gradle
	 or
	 sdk rm gradle

#### 使用临时版本

	sdk use gradle 3.0

#### 设置默认版本
	
    sdk default gradle 3.0

#### 查看安装的sdk版本列表

	sdk current gradle

#### Grale其他地址

[Binary only distribution (no documentation or source code)](https://services.gradle.org/distributions/gradle-3.3-all.zip)

[Gradle source code (just the Gradle source code; not a usable Gradle installation)](https://services.gradle.org/distributions/gradle-3.3-src.zip)