### Spring MVC 中 HandlerInterceptorAdapter的使用

一般情况下，对来自浏览器的请求的拦截，是利用Filter实现的，这种方式可以实现Bean预处理、后处理。 
Spring MVC的拦截器不仅可实现Filter的所有功能，还可以更精确的控制拦截精度。 

spring为我们提供了org.springframework.web.servlet.handler.HandlerInterceptorAdapter这个适配器，继承此类，可以非常方便的实现自己的拦截器。他有三个方法：
	
	package org.springframework.web.servlet.handler;
	
	import javax.servlet.http.HttpServletRequest;
	import javax.servlet.http.HttpServletResponse;
	
	import org.springframework.web.servlet.AsyncHandlerInterceptor;
	import org.springframework.web.servlet.ModelAndView;
	
	public abstract class HandlerInterceptorAdapter implements AsyncHandlerInterceptor {

		public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)
			throws Exception {
			return true;
		}
	
		public void postHandle(
				HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView)
				throws Exception {
		}
	
		public void afterCompletion(
				HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex)
				throws Exception {
		}
	
		public void afterConcurrentHandlingStarted(
				HttpServletRequest request, HttpServletResponse response, Object handler)
				throws Exception {
		}

    }

其中afterConcurrentHandlingStarted方法是其父类的

这三个方法分别实现预处理、后处理（调用了Service并返回ModelAndView，但未进行页面渲染）、返回处理（已经渲染了页面） 
在preHandle中，可以进行编码、安全控制等处理； 在postHandle中，有机会修改ModelAndView； 在afterCompletion中，可以根据ex是否为null判断是否发生了异常，进行日志记录。 

如果基于xml配置使用Spring MVC，
可以利用SimpleUrlHandlerMapping、BeanNameUrlHandlerMapping进行Url映射（相当于struts的path映射）和拦截请求（注入interceptors），
如果基于注解使用Spring MVC，可以使用DefaultAnnotationHandlerMapping注入interceptors。
注意无论基于xml还是基于注解，HandlerMapping bean都是需要在xml中配置的。 

一个demo： 

在这个例子中，我们假设UserController中的注册操作只在9：00-12：00开放，那么就可以使用拦截器实现这个功能。 

	public class TimeBasedAccessInterceptor extends HandlerInterceptorAdapter {    
	    private int openingTime;    
	    private int closingTime;    
	    private String mappingURL;//利用正则映射到需要拦截的路径    
	    public void setOpeningTime(int openingTime) {    
	        this.openingTime = openingTime;    
	    }    
	    public void setClosingTime(int closingTime) {    
	        this.closingTime = closingTime;    
	    }    
	    public void setMappingURL(String mappingURL) {    
	        this.mappingURL = mappingURL;    
	    }    
	    @Override    
	    public boolean preHandle(HttpServletRequest request,    
	            HttpServletResponse response, Object handler) throws Exception {    
	        String url=request.getRequestURL().toString();    
	        if(mappingURL==null || url.matches(mappingURL)){    
	            Calendar c=Calendar.getInstance();    
	            c.setTime(new Date());    
	            int now=c.get(Calendar.HOUR_OF_DAY);    
	            if(now<openingTime || now>closingTime){    
	                request.setAttribute("msg", "注册开放时间：9：00-12：00");    
	                request.getRequestDispatcher("/msg.jsp").forward(request, response);    
	                return false;    
	            }    
	            return true;    
	        }    
	        return true;    
	    }    
	}    


xml配置： 

	<bean id="timeBasedAccessInterceptor" class="com.spring.handler.TimeBasedAccessInterceptor">    
	    <property name="openingTime" value="9" />    
	    <property name="closingTime" value="12" />    
	    <property name="mappingURL" value=".*/user\.do\?action=reg.*" />    
	</bean>    
	<bean class="org.springframework.web.servlet.mvc.annotation.DefaultAnnotationHandlerMapping">    
	    <property name="interceptors">    
	        <list>    
	            <ref bean="timeBasedAccessInterceptor"/>    
	        </list>    
	    </property>    
	</bean>    

这里我们定义了一个mappingURL属性，实现利用正则表达式对url进行匹配，从而更细粒度的进行拦截。当然如果不定义mappingURL，则默认拦截所有对Controller的请求。 

UserController： 

	@Controller    
	@RequestMapping("/user.do")    
	public class UserController{    
	    @Autowired    
	    private UserService userService;    
	    @RequestMapping(params="action=reg")    
	    public ModelAndView reg(Users user) throws Exception {    
	        userService.addUser(user);    
	        return new ModelAndView("profile","user",user);    
	    }    
	    // other option ...    
	}  

这个Controller相当于Struts的DispatchAction 

你也可以配置多个拦截器，每个拦截器进行不同的分工. 