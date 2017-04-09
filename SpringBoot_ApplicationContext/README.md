### SpringBoot 获取应上下文 ApplicationContent ###
#### 1、定义上下文工具类： ####
```java
package com.alimama.config;

import org.springframework.context.ApplicationContext;
/**
 * 上下文获取工具类
 * @author mengfeiyang
 *
 */
public class SpringContextUtil {
	  private static ApplicationContext applicationContext;

	  public static void setApplicationContext(ApplicationContext context) {
	    applicationContext = context;
	  }
	  
	   public static Object getBean(String beanId) {
	    return applicationContext.getBean(beanId);
	  }
}
```

#### 2、在启动入口类中注入applicationContext ####
```java
package com.alimama;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.context.embedded.ConfigurableEmbeddedServletContainer;
import org.springframework.boot.context.embedded.EmbeddedServletContainerCustomizer;
import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.ComponentScan;

import com.alimama.config.SbootConfig;
import com.alimama.config.SpringContextUtil;
import com.alimama.config.ZKConfig;
import com.alimama.quartz.InitTask;

/**
 * spring boot启动入口类
 * @author mengfeiyang
 *
 */
@ComponentScan
@SpringBootApplication
@EnableConfigurationProperties({ZKConfig.class,SbootConfig.class})
public class SbootApplication implements EmbeddedServletContainerCustomizer{

	public static void main(String[] args) {
		ApplicationContext applicationContext = SpringApplication.run(SbootApplication.class, args);
		SpringContextUtil.setApplicationContext(applicationContext);
	}

	@Override
	public void customize(ConfigurableEmbeddedServletContainer container) {
		
	}
}
```

#### 3、调用方法 ####
```java
package com.alimama.quartz;

import java.io.IOException;

import org.phoenix.api.action.IInterfaceAPI;
import org.phoenix.api.action.InterfaceAPI;
import org.quartz.Job;
import org.springframework.beans.factory.annotation.Autowired;

import com.alimama.config.SpringContextUtil;
import com.alimama.dto.TaskBean;
import com.alimama.service.IConfigService;
import com.alimama.service.impl.ConfigService;
/**
 * 任务执行者
 * @author mengfeiyang
 *
 */
public class TaskHandler implements Job{
	private ConfigService configService = (ConfigService) SpringContextUtil.getBean("configService");
	private IInterfaceAPI interf = new InterfaceAPI();
	@Override
	public void execute(JobExecutionContext arg0){
		String watchDogServer = configService.getwatchDogServer();
	    System.out.println(watchDogServer);
	}
}
```
