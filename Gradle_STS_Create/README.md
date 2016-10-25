cd ##如何使用Gradle构建Web项目
关于Gradle这里不过多的描述

打开STS，打开Eclipse MarketPlace

![](https://github.com/silence940109/Java/blob/master/Gradle_STS_Create/image/1.png)

搜索gradle，选择

![](https://github.com/silence940109/Java/blob/master/Gradle_STS_Create/image/1.png)

点击install，它就会自己安装完成，然后重启,在新建项目中可以看到

![](https://github.com/silence940109/Java/blob/master/Gradle_STS_Create/image/2.png)

下一步

![](https://github.com/silence940109/Java/blob/master/Gradle_STS_Create/image/3.png)

输入项目名和项目位置

![](https://github.com/silence940109/Java/blob/master/Gradle_STS_Create/image/4.png)

选择gradle为本地，因为我在本地已经安装了gradle，如果从网上下载是相当慢的，还有JAVA_HOME位置，JVM配置信息，运行参数，这些都可以省略

![](https://github.com/silence940109/Java/blob/master/Gradle_STS_Create/image/5.png)

项目完成后，它不是一个web项目，我们可以右击项目->properties->Project Facts->选择Dynamic Web Module，可以修改WebRoot文件夹为src/main/webapp,并把web.xml文件添加上去

![](https://github.com/silence940109/Java/blob/master/Gradle_STS_Create/image/6.png)

该项目最后的结构为

![](https://github.com/silence940109/Java/blob/master/Gradle_STS_Create/image/7.png)

可以安装Gradle Edit插件高亮显示.gradle文件属性

![](https://github.com/silence940109/Java/blob/master/Gradle_STS_Create/image/gradleEdit.png)

代码高亮

![](https://github.com/silence940109/Java/blob/master/Gradle_STS_Create/image/gradleEdit1.png)

你可以从[下载](https://github.com/silence940109/SSM/releases/tag/1.0)下载该框架的源代码











