### Gretty插件实现Gradle Web项目热部署
在build.gradle配置文件中

	buildscript {
	repositories {
		jcenter()
	}
	dependencies {
		classpath 'org.akhikhl.gretty:gretty:+'
	}
	}
	
	apply plugin: 'org.akhikhl.gretty'
	或
	apply from: 'https://raw.github.com/akhikhl/gretty/master/pluginScripts/gretty.plugin'

常用命令

	gradle appRun

另有appRunWar、appRunDebug、appRunWarDebug

	gradle appStart

另有appStartWar、appStartDebug、appStartWarDebug

gradle jetty* / gradle tomcat*

	gretty {
		// 端口默认8080
		// serlvetContainer 支持 jetty7/8/9，tomcat7/8
		// contextPath 设置根路径，默认为项目名称
		port = 8080
		servletContainer = 'tomcat8'
		contextPath = '/'
	}

热部署属性

	scanInterval：监视周期，单位为秒，设置为0等于完全关闭热部署
	scanDir：需要监视的文件夹
	recompileOnSourceChange：监视源码变动，自动编译
	reloadOnClassChange：编译的类发生改变，自动加载
	reloadOnConfigChange：WEB-INF或META-INF发生改变
	reloadOnLibChange：依赖发生改变

Gretty默认如下

	scanInterval 设置为1，每秒扫描改动1次
	scanDir默认为下 ：
	
	${projectdir}/src/main/java
	${projectdir}/src/main/groovy
	${projectdir}/src/main/resources
	${projectdir}/build/classes/main
	${projectdir}/build/resources/main
	
	recompileOnSourceChange、reloadOnClassChange、reloadOnConfigChange 和 reloadOnLibChange默认为true

fastReload属性，默认为true，监听webapp/中的内容，文件发生改变，无需重启。

除了src/main/webapp外，可另外指定资源目录

	gretty{
		// …
		extraResourceBase 'dir1',
		extraResourceBases 'dir2','dir3'
		// …
	}

产品生成
	
gradle buildProduct

	生成安装文件
	生成目录位于 build/output/${project.name}
	结构如下
	–build/output/${project.name}
	|–conf/ => 配置文件
	|–runner/ => servlet container 所需库
	|–starter/
	|–webapps/ => java web 应用
	|–restart.bat/sh
	|–run.bat/sh
	|–start.bat/sh
	|–stop.bat/sh
	多应用，需在build.gradle中配置 product，例如
	product {
	webapp project // include this project
	webapp ':ProjectA'
	webapp ':ProjectB'
	}

项目启动，修改项目文件，自动编译部署

![](https://github.com/silence940109/Java/blob/master/Gradle_Gretty/image/1.png)
