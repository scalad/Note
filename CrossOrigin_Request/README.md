[关于使用JS解决跨域问题](http://www.cnblogs.com/dojo-lzz/p/4265637.html)

服务器端解决跨域请求问题，拦截请求并重新设置响应头

服务器端拦截器

```Java

	package com.silence.util;
	
	import java.io.IOException;
	import java.text.SimpleDateFormat;
	import java.util.Date;
	
	import javax.servlet.Filter;
	import javax.servlet.FilterChain;
	import javax.servlet.FilterConfig;
	import javax.servlet.ServletException;
	import javax.servlet.ServletRequest;
	import javax.servlet.ServletResponse;
	import javax.servlet.http.HttpServletResponse;
	
	import org.slf4j.Logger;
	import org.slf4j.LoggerFactory;
	
	public class CrossOriginFilter implements Filter {
		
		private String allowDomain = "";
		private static final Logger logger = LoggerFactory.getLogger(CrossOriginFilter.class);
		
		public void init(FilterConfig filterConfig) throws ServletException {
			allowDomain = filterConfig.getInitParameter("domain");
		}

		public void doFilter(ServletRequest request, ServletResponse response,
				FilterChain chain) throws IOException, ServletException {
			logger.info("CrossOriginFilter 跨域请求拦截 " + new SimpleDateFormat("YYYY-DD-MM").format(new Date()));
			HttpServletResponse httpResponse = (HttpServletResponse) response;
			setAccessControl(httpResponse);
			chain.doFilter(request, response);
		}
	
		public void destroy() {
			
		}
		/**
		*在某域名下使用Ajax向另一个域名下的页面请求数据，会遇到跨域问题。另一个域名必须在response中添加 
		*Access-Control-Allow-Origin 的header，才能让前者成功拿到数据。
		*只有当目标页面的response中，包含了 Access-Control-Allow-Origin 这个header，并且它的值里有我们自己的域名时，
		*浏览器才允许我们拿到它页面的数据进行下一步处理。
		*如果它的值设为 * ，则表示谁都可以用
		*/
		private void setAccessControl(HttpServletResponse response) {
			response.setHeader("Access-Control-Allow-Origin", allowDomain);
			response.setHeader("Access-Control-Allow-Credentials", "true");
			String headers = "Origin, Accept-Language, Accept-Encoding,X-Forwarded-For, Connection, Accept, User-Agent, Host, Referer,Cookie, Content-Type, Cache-Control";
			response.setHeader("Access-Control-Allow-Headers", headers);
			response.setHeader("Access-Control-Request-Method", "GET,POST");
		}
	
	}
```

在web.xml中配置

	<filter>
		<filter-name>CrossOriginFilter</filter-name>
		<filter-class>com.silence.util.CrossOriginFilter</filter-class>
		<init-param>
			<param-name>domain</param-name>
			<param-value>*</param-value>
		</init-param>
	</filter>
	<filter-mapping>
		<filter-name>CrossOriginFilter</filter-name>
		<url-pattern>/*</url-pattern>
	</filter-mapping>

默认拦截所有的请求，允许来自任何链接的请求