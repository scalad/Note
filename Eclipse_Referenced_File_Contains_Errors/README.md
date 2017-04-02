### Eclipse Xml编译错误Referenced file contains errors - spring-beans-4.0.xsd

在eclipse中，有时候在xml文件中，特别是于spring相关的配置文件中，会出现一些不影响程序正常运行的编译错误，如：

Referenced file contains errors (http://www.springframework.org/schema/beans/spring-beans-4.0.xsd).

可通过如下步骤解决这个编译错误：

1. Spring的版本变更了，但是Eclipse的编译器还是使用之前缓存的spring-beans-**.xsd文件。其原因是

对于Eclipse编译器来说有个缓存会缓存这些配置文件，这样验证的时候会告诉你版本不统一。

解决办法是清空这些文件并强制eclipse重新加载这些文件。

> 1） Preferences -> General -> Network Connections -> Cache

选择响应的文件病点击删除或者直接点击删除全部。

> 2） 如果是Maven工程，右击工程，并选择Maven,选择Update Project.

> 3）如果以上两步都不行，则可关闭project并重新打开强制eclipse进行编译。


2 . 当前使用的spring版本和配置文件中配置的不相同，导致xsd等文件不会被正确加载，改成当前版本即可。

如果不成功则重复上面的2）3）两步即可。

3 . 在使用spring时，使用多个配置文件，那么头里面的配置一定要统一。

如果不成功则重复上面的2）3）两步即可。


从网上http://stackoverflow.com/questions/7267341/validation-error-of-spring-beans-schema-inside-application-context看到可能又另外一种情况：

就是你在spring的配置文件中混合使用了不同版本的xsd文件，你可以把这些xsd文件的版本设置为一样，然后重新更新下项目