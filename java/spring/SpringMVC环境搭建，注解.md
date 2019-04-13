# 使用注解搭建SpringMVC环境

1. 导包

   ![1546930827198](C:\Users\陈勇\AppData\Roaming\Typora\typora-user-images\1546930827198.png)

2. 在web.xml添加`DispatcherServlet`

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
   	xmlns="http://java.sun.com/xml/ns/javaee"
   	xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
   	id="WebApp_ID" version="3.0">
   	<servlet>
   		<servlet-name>springmvc</servlet-name>
   		<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
   		<init-param>
   			<!-- 修改SpringMVC配置文件的路径 -->
   			<param-name>contextConfigLocation</param-name>
   			<param-value>classpath:springmvc.xml</param-value>
   		</init-param>
   
   		<!-- 自启动 当tomcat启动的时候 该servlet类就会被加载 -->
   		<load-on-startup>1</load-on-startup>
   	</servlet>
   
   	<servlet-mapping>
   		<servlet-name>springmvc</servlet-name>
   		<url-pattern>/</url-pattern>
   	</servlet-mapping>
   
   
   	<!-- 配置全局编码格式  只对Post有效-->
   	<filter>
   	<filter-name>encoding</filter-name>
   	<filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
   	<init-param><param-name>encoding</param-name><param-value>utf-8</param-value></init-param>
   	</filter>
       
   	<filter-mapping>
   	<filter-name>encoding</filter-name>
   	<url-pattern>/*</url-pattern>
   	</filter-mapping>
       
       
   </web-app>        
   ```

3. 配置springmvc.xml

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
   	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
   	xmlns:context="http://www.springframework.org/schema/context"
   	xmlns:mvc="http://www.springframework.org/schema/mvc"
   	xsi:schemaLocation="http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans.xsd
                   http://www.springframework.org/schema/context
           http://www.springframework.org/schema/context/spring-context.xsd
                   http://www.springframework.org/schema/mvc
           http://www.springframework.org/schema/mvc/spring-mvc.xsd
           ">
           <!-- 扫描注解 -->
   	<context:component-scan
   		base-package="com.cy.controller">
   	</context:component-scan>
   	<!-- 注解驱动 相当于配置 HandlerMapping 和 HandlerAdapter -->
   	<mvc:annotation-driven></mvc:annotation-driven>
       
       <!-- 不让DispatcherServlet拦截静态资源 所谓的路由配置 -->
   	<!-- mapping中 *表示当前路径下的所有文件，**表示当前路径下的所有文件以及子文件 -->
   	<!-- mapping为请求URL -->
   	<!-- location为静态资源的地址 -->
   	<mvc:resources location="/js/" mapping="/js/**"></mvc:resources>
   	
   	
   </beans>
   ```

4. 创建Controller

   ```java
   @Controller
   public class DemoController {
   	@RequestMapping("demo")
       //在此处添加参数可以自动装配哦。
   	public String demo() {
   		System.out.println("执行demo");
   		return "/main.jsp";
   	}
   }
   
   ```



