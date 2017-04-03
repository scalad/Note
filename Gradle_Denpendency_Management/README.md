### Gradle系列之依赖管理
Gradle依赖管理看起来很容易，但是当出现依赖解析冲突时就会很棘手， 复杂的依赖关系可能导致构建中依赖一个库的多个版本。Gradle通过分析依赖树得到依赖报告，你将很容易找到一个指定的依赖的来源.

Gradle有自己的依赖管理实现，除了支持ant和Maven的特性外，Gradle关心的是性能、可靠性和复用性.

#### 简要概述依赖管理
几乎所有基于JVM的项目都会或多或少依赖其他库，假设你在开发一个基于web的项目，你很可能会依赖很受欢迎的开源框架比如Spring MVC来提高效率。Java的第三方库一般以JAR文件的形式存在，一般用库名加版本号来标识。随着开发的进行依赖的第三方库增多小的项目变的越来越大， 组织和管理你的JAR文件就很关键.

#### 不算完美的依赖管理技术
由于Java语言并没提供依赖管理的工具，所以你的团队需要自己开发一套存储和检索依赖的想法。你可能会采取以下几种常见的方法：

* 手动复制JAR文件到目标机器，这是最原始的很容易出错的方法

* 使用一个共享的存储介质来存储JAR文件(比如共享的网盘)，你可以加载网络硬盘或者通过FTP检索二进制文件。这种方法需要开发者事先建立好与仓库的连接，手动添加新的依赖到仓库中
 
* 把依赖的JAR文件同源代码都添加到版本控制系统中。这种方法不需要任何额外的步骤，你的同伴在拷贝仓库的时候就能检索依赖的改变。另一方面，这些JAR文件占用了不必要的空间，当你的项目存在相互之间依赖的时候你需要频繁的check-in的检查源代码是否发生了改变.

#### 自动管理依赖的重要性
尽管上面的方法都能用，但是这距离理想的解决方案差远了，因为他们没有提供一个标准化的方法来命名和管理JAR文件。至少你得需要开发库的准确版本和它依赖的库(传递依赖)，这个为什么这么重要？

#### 准确知道依赖的版本
如果在项目中你没有准确声明依赖的版本这将会是一个噩梦，如果没有文档你根本无法知道这个库支持哪些特性，是否升级一个库到新的版本就变成了一个猜谜游戏因为你不知道你的当前版本.

#### 管理传递依赖
在项目的早期开发阶段传递依赖就会是一个隐患，这些库是第一层的依赖需要的，比如一个比较常见的开发方案是将Spring和Hibernate结合起来这会引入超过20个其他的开发库，一个库需要很多其他库来正常工作。下图展示了Hibernate核心库的依赖图：

![](https://github.com/silence940109/Java/blob/master/Gradle_Denpendency_Management/image/20150512084408_658.png)

如果没有正确的管理依赖，你可以会遇到没想到过的编译期错误和运行期类加载问题。我们可以总结到我们需要一个更好的方式来管理依赖，一般来讲你想在 项目元数据中声明你的依赖和它的版本号。作为一个项目自动化的过程，这个版本的库会自动从中央仓库下载、安装到你的项目中，我们来看几个现有的开源解决方 案。

#### 使用自动化的依赖管理
在Java领域里支持声明的自动依赖管理的有两个项目：Apache Ivy(Ant项目用的比较多的依赖管理器)和Maven(在构建框架中包含一个依赖管理器)，我不再详细介绍这两个的细节而是解释自动依赖管理的概念和机制

Ivy和Maven是通过XML描述文件来表达依赖配置，配置包含两部分：依赖的标识加版本号和中央仓库的位置(可以是一个HTTP链接)，依赖管 理器根据这个信息自动定位到需要下载的仓库然后下载到你的机器中。库可以定义传递依赖，依赖管理器足够聪明分析这个信息然后解析下载传递依赖。如果出现了 依赖冲突比如上面的Hibernate core的例子，依赖管理器会试着解决。库一旦被下载就会存储在本地的缓存中，构建系统先检查本地缓存中是否存在需要的库然后再从远程仓库中下载。下图显 示了依赖管理的关键元素：

![](https://github.com/silence940109/Java/blob/master/Gradle_Denpendency_Management/image/20150512084409_161.png)

Gradle通过DSL来描述依赖配置，实现了上面描述的架构

#### 自动依赖管理面临的挑战
虽然依赖管理器简化了手工的操作，但有时也会遇到问题。你会发现你的依赖图中会依赖同个库的不同版本，使用日志框架经常会遇到这个问题，依赖管理器 基于一个特定的解决方案只选择其中一个版本来避免版本冲突。如果你想知道某个库引入了什么版本的传递依赖，Gradle提供了一个非常有用的依赖报告来回 答这个问题。下一节我会通过一个例子来讲解

#### 声明依赖
DSL配置block dependencies用来给配置添加一个或多个依赖，你的项目不仅可以添加外部依赖，下面这张表显示了Gradle支持的各种不同类型的依赖.

![](https://github.com/silence940109/Java/blob/master/Gradle_Denpendency_Management/image/20150512084409_541.png)

这一章直接扫外部模块依赖和文件依赖，我们来看看Gradle APi是怎么表示依赖的

#### 理解依赖的API表示

每个Gradle项目都有一个DependencyHandler的实例，你可以通过getDependencies()方法来获取依赖处理器的引 用，上表中每一种依赖类型在依赖处理器中都有一个相对应的方法。每一个依赖都是Dependency的一个实例，group, name, version, 和classifier这几个属性用来标识一个依赖，下图清晰的表示了项目(Project)、依赖处理器(DependencyHandler)和依赖 三者之间的关系：

![](https://github.com/silence940109/Java/blob/master/Gradle_Denpendency_Management/image/20150512084410_595.png)

#### 外部模块依赖
在Gradle的术语里，外部库通常是以JAR文件的形式存在，称之为外部模块依赖，代表项目层次外的一个模块，这种类型的依赖是通过属性来唯一的标识，接下来我们来介绍每个属性的作用

#### 依赖属性

当依赖管理器从仓库中查找依赖时，需要通过属性的结合来定位，最少需要提供一个name。

* group： 这个属性用来标识一个组织、公司或者项目，可以用点号分隔，Hibernate的group是org.hibernate。

* name： name属性唯一的描述了这个依赖，hibernate的核心库名称是hibernate-core。

* version： 一个库可以有很多个版本，通常会包含一个主版本号和次版本号，比如Hibernate核心库3.6.3-Final。

* classifier： 有时候需要另外一个属性来进一步的说明，比如说明运行时的环境，Hibernate核心库没有提供classifier。

#### 依赖的写法

你可以使用下面的语法在项目中声明依赖：

	dependencies {
		configurationName dependencyNotation1, 	dependencyNotation2, ...
	}

你先声明你要给哪个配置添加依赖，然后添加依赖列表，你可以用map的形式来注明，你也可以直接用冒号来分隔属性，比如这样的：

![](https://github.com/silence940109/Java/blob/master/Gradle_Denpendency_Management/image/20150512084410_977.png)

	//声明外部属性
	ext.cargoGroup = 'org.codehaus.cargo'
	ext.cargoVersion = '1.3.1'

	dependencies {
		//使用映射声明依赖
		compile group: cargoGroup, name: 'cargo-core-uberjar',version: cargoVersion
		//用快捷方式来声明，引用了前面定义的外部属性
		cargo "$cargoGroup:cargo-ant:$cargoVersion"
	}

如果你项目中依赖比较多，你把一些共同的依赖属性定义成外部属性可以简化build脚本.

Gradle没有给项目选择默认的仓库，当你没有配置仓库的时候运行deployTOLocalTomcat任务的时候回出现如下的错误：
	
	$ gradle deployToLocalTomcat
	:deployToLocalTomcat FAILED
	FAILURE: Build failed with an exception.
	
	Where: Build file '/Users/benjamin/gradle-in-action/code/chapter5/cargo-configuration/build.gradle' line: 10
	
	What went wrong:
	Execution failed for task ':deployToLocalTomcat'.
	> Could not resolve all dependencies for configuration ':cargo'.
		> Could not find group:org.codehaus.cargo, module:cargo-core-uberjar, version:1.3.1.
		Required by:
			:cargo-configuration:unspecified
	> Could not find group:org.codehaus.cargo, module:cargo-ant,version:1.3.1.
		Required by:
		:cargo-configuration:unspecified

到目前为止还没讲到怎么配置不同类型的仓库，比如你想使用MavenCentral仓库，添加下面的配置代码到你的build脚本中：
	
	repositories {
		mavenCentral()
	}

#### 检查依赖报告

当你运行dependencies任务时，这个依赖树会打印出来，依赖树显示了你build脚本声明的顶级依赖和它们的传递依赖：

![](https://github.com/silence940109/Java/blob/master/Gradle_Denpendency_Management/image/20150512084410_225.png)

仔细观察你会发现有些传递依赖标注了*号，表示这个依赖被忽略了，这是因为其他顶级依赖中也依赖了这个传递的依赖，Gradle会自动分析下载最合适的依赖.

#### 排除传递依赖

Gradle允许你完全控制传递依赖，你可以选择排除全部的传递依赖也可以排除指定的依赖，假设你不想使用UberJar传递的xml-api的版本而想声明一个不同版本，你可以使用exclude方法来排除它：

	dependencies {
		cargo('org.codehaus.cargo:cargo-ant:1.3.1') {
			exclude group: 'xml-apis', module: 'xml-apis'
		}
		cargo 'xml-apis:xml-apis:2.0.2'
	}

exclude属性值和正常的依赖声明不太一样，你只需要声明group和(或)module，Gradle不允许你只排除指定版本的依赖.

有时候仓库中找不到项目依赖的传递依赖，这会导致构建失败，Gradle允许你使用transitive属性来排除所有的传递依赖：

	dependencies {
		cargo('org.codehaus.cargo:cargo-ant:1.3.1') {
		transitive = false
		}
		// 选择性的声明一些需要的库
	}

#### 动态版本声明

如果你想使用一个依赖的最新版本，你可以使用latest.integration，比如声明 Cargo Ant tasks的最新版本，你可以这样写org.codehaus .cargo:cargo-ant:latest-integration，你也可以用一个+号来动态的声明：

	dependencies {
		//依赖最新的1.x版本
		cargo 'org.codehaus.cargo:cargo-ant:1.+'
	}

Gradle的dependencies任务可以清晰的看到选择了哪个版本，这里选择了1.3.1版本：

	$ gradle –q dependencies
	------------------------------------------------------------
	Root project
	------------------------------------------------------------
	Listing 5.4 Excluding a single dependency
	Listing 5.5 Excluding all transitive dependencies
	Listing 5.6 Declaring a dependency on the latest Cargo 1.x version
	Exclusions can be
	declared in a shortcut
	or map notation.
	120 CHAPTER 5 Dependency management
	cargo - Classpath for Cargo Ant tasks.
	\--- org.codehaus.cargo:cargo-ant:1.+ -> 1.3.1
	\--- ...

#### 文件依赖
如果你没有使用自动的依赖管理工具，你可能会把外部库作为源代码的一部分或者保存在本地文件系统中，当你想把项目迁移到Gradle的时候，你不想去重构，Gradle很简单就能配置文件依赖。下面这段代码复制从Maven中央仓库解析的依赖到libs/cargo目录。

	task copyDependenciesToLocalDir(type: Copy) {
		//Gradle提供的语法糖
		from configurations.cargo.asFileTree
		into "${System.properties['user.home']}/libs/cargo"
	}

运行这个任务之后你就可以在依赖中声明Cargo库了，下面这段代码展示了怎么给cargo配置添加JAR文件依赖：

	dependencies {
		cargo fileTree(dir: "${System.properties['user.home']}/libs/cargo",include: '*.jar')
	}

#### 配置远程仓库
Gradle支持下面三种不同类型的仓库：

![](https://github.com/silence940109/Java/blob/master/Gradle_Denpendency_Management/image/20150512084411_208.png)

下图是配置不同仓库对应的Gradle API：

![](https://github.com/silence940109/Java/blob/master/Gradle_Denpendency_Management/image/20150512084411_990.png)

下面以Maven仓库来介绍，Maven仓库是Java项目中使用最为广泛的一个仓库，库文件一般是以JAR文件的形式存在，用XML(POM文 件)来来描述库的元数据和它的传递依赖。所有的库文件都存储在仓库的指定位置，当你在构建脚本中声明了依赖时，这些属性用来找到库文件在仓库中的准确位 置。group属性标识了Maven仓库中的一个子目录，下图展示了Cargo依赖属性是怎么对应到仓库中的文件的：

![](https://github.com/silence940109/Java/blob/master/Gradle_Denpendency_Management/image/20150512084412_417.png)

RepositoryHandler接口提供了两个方法来定义Maven仓库，mavenCentral方法添加一个指向仓库列表的引用，mavenLocal方法引用你文件系统中的本地Maven仓库.

#### 添加Maven仓库

要使用Maven仓库你只需要调用mavenCentral方法，如下所示：

	repositories {
		mavenCentral()
	}

添加本地仓库

本地仓库默认在 /.m2/repository目录下，只需要添加如下脚本来引用它：
	
	repositories {
		mavenLocal()
	}

#### 添加自定义Maven仓库

如果指定的依赖不存在与Maven仓库或者你想通过建立自己的企业仓库来确保可靠性，你可以使用自定义的仓库。仓库管理器允许你使用Maven布局 来配置一个仓库，这意味着你要遵守artifact的存储模式。你也可以添加验证凭证来提供访问权限，Gradle的API提供两种方法配置自定义的仓 库：maven()和mavenRepo()。下面这段代码添加了一个自定义的仓库，如果Maven仓库中不存在相应的库会从自定义仓库中查找：
	
	repositories {
		mavenCentral()
		maven {
		name 'Custom Maven Repository',
		url 'http://repository.forge.cloudbees.com/release/')
		}
	}

来自：[http://coolshell.info/blog/2015/05/gradle-dependency-management.html](http://coolshell.info/blog/2015/05/gradle-dependency-management.html)