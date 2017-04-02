### Alibaba Druid数据源监控

在[阿里巴巴Druid](https://github.com/Alibaba/Druid)数据库连接池上写着这一句话,"为监控而生的数据库连接池"，确实，Druid在数据库连接池监控上做得非常完善。

你可以克隆下来然后安装到你的额本地仓库中

* git clone https://github.com/alibaba/druid.git
 
* cd druid && mvn install

关于Druid的中文文档，你可以在[这里](https://github.com/alibaba/druid/wiki/%E5%B8%B8%E8%A7%81%E9%97%AE%E9%A2%98)查看到。

下面主要完成项目中如何集成集成Druid

如果你使用的Maven，你需要加入Druid的依赖

	<!-- https://mvnrepository.com/artifact/com.alibaba/druid -->
	<dependency>
	    <groupId>com.alibaba</groupId>
	    <artifactId>druid</artifactId>
	    <version>1.0.26</version>
	</dependency>

如果你使用的是Gradle，你需要在denpendencies中加入

	compile 'com.alibaba:druid:1.0.26'

然后再spring的配置文件中配置数据源

	<bean name="dataSource" class="com.alibaba.druid.pool.DruidDataSource" init-method="init" destroy-method="close">
	    <property name="driverClassName" value="${driver}" />
	    <property name="url" value="${url}" />
	    <property name="username" value="${user}" />
	    <property name="password" value="${password}" />
	    <!-- 初始化连接大小 -->
	    <property name="initialSize" value="${initialSize}"></property>
	    <!-- 连接池最大数量 -->
	    <property name="maxActive" value="${maxActive}"></property>
		<!-- 连接池最小空闲 -->
	    <property name="minIdle" value="${minIdle}"></property>
	    <!-- 获取连接最大等待时间 -->
	    <property name="maxWait" value="${maxWait}"></property>
	    <!-- 配置间隔多久才进行一次检测，检测需要关闭的空闲连接，单位是毫秒 -->
      	<property name="timeBetweenEvictionRunsMillis" value="60000" />
      	<!-- 配置一个连接在池中最小生存的时间，单位是毫秒 -->
        <property name="minEvictableIdleTimeMillis" value="300000" />
	    <property name="validationQuery" value="SELECT 'x'" />
        <property name="testWhileIdle" value="true" />
        <property name="testOnBorrow" value="false" />
        <property name="testOnReturn" value="false" />
        <!-- 如果用Oracle，则把poolPreparedStatements配置为true，mysql可以配置为false。分库分表较多的数据库，建议配置为false。 -->
        <!-- 打开PSCache，并且指定每个连接上PSCache的大小 -->
        <property name="poolPreparedStatements" value="true" />
        <property name="maxPoolPreparedStatementPerConnectionSize" value="20" />
        <!-- 配置监控统计拦截的filters -->
        <property name="filters" value="stat" /> 
	</bean>

db.properties配置文件

	driver=com.mysql.jdbc.Driver
	url=jdbc:mysql://localhost:3306/ssm
	user=root
	password=root
	initialSize=5
	maxActive=20
	minIdle=1
	maxWait=60000

最后，我们还需要在web.xml加入

	<servlet>
      <servlet-name>DruidStatView</servlet-name>
      <servlet-class>com.alibaba.druid.support.http.StatViewServlet</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>DruidStatView</servlet-name>
        <url-pattern>/druid/*</url-pattern>
    </servlet-mapping>

启动服务器，访问`http://localhost:8080/ProjectName/druid

可以看到

![](https://github.com/silence940109/Java/blob/master/Alibaba_Druid/1.png)

![](https://github.com/silence940109/Java/blob/master/Alibaba_Druid/2.png)

具体项目看[我的项目](https://github.com/silence940109/SSM)