### Maven中把依赖的JAR包一起打包

这里所用到的MAVEN-PLUGIN是MAVNE-ASSEMBLY-PLUGIN
官方网站是:[http://maven.apache.org/plugins/maven-assembly-plugin/usage.html](http://maven.apache.org/plugins/maven-assembly-plugin/usage.html)

1. 添加此PLUGIN到项目的POM.XML中
```java
	<build>  
        <plugins>  
            <plugin>  
                <artifactId>maven-assembly-plugin</artifactId>  
                <configuration>  
                    <archive>  
                        <manifest>  
                          <mainClass>com.allen.capturewebdata.Main</mainClass>  
                        </manifest>  
                    </archive>  
                    <descriptorRefs>  
                        <descriptorRef>jar-with-dependencies</descriptorRef>  
                    </descriptorRefs>  
                </configuration>  
            </plugin>  
        </plugins>  
    </build>  
```

如果出现CLASS重名的情况,这时候就要把最新的版本号添加进去即可.

2, 在当前项目下执行mvn assembly:assembly, 执行成功后会在target文件夹下多出一个以-jar-with-dependencies结尾的JAR包. 这个JAR包就包含了项目所依赖的所有JAR的CLASS.
 
3.如果不希望依赖的JAR包变成CLASS的话,可以修改ASSEMBLY插件.
  
3.1 找到assembly在本地的地址,一般是c:/users/${your_login_name}/.m2/\org\apache\maven\plugins\maven-assembly-plugin\2.4

3.2 用WINZIP或解压工具打开此目录下的maven-assembly-plugin-2.4.jar, 找到assemblies\jar-with-dependencies.xml

3.3 把里面的UNPACK改成FALSE即可