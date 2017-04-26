## Gradle排除依赖关系

在IDE中发现了C3P0的依赖，但是在build.gradle并没有手动导入，所以说某个jar包依赖了，在STS中没有像Maven可以直接查看依赖的窗口

可以在命令行下查看整个项目的依赖关系

	gradle dependencies

![](https://github.com/silence940109/Java/blob/master/Gradle_Denpendencies/image/1.png)

然后我们就可以在build.gradle去除该依赖

    compile ('org.quartz-scheduler:quartz:2.2.1'){
        //排除c3p0的依赖
    	exclude group:'c3p0',module:"c3p0"
    }

重新Gradle Refresh Gradle Project即可

