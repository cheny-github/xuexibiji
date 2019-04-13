# Spring中设置全局编码格式

在web.xml中进行如下配置

```xml
	<!-- 配置全局编码格式 -->
	<!-- 配置编码方式过滤器,注意一点:要配置在所有过滤器的前面 -->
	<filter>
		<filter-name>CharacterEncodingFilter</filter-name>
		<filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
		<init-param>
			<param-name>encoding</param-name>
			<param-value>utf-8</param-value>
		</init-param>
	</filter>
	<filter-mapping>
		<filter-name>CharacterEncodingFilter</filter-name>
		<url-pattern>/*</url-pattern>
	</filter-mapping>
```

此方式只对post请求有效。对于get请求使用

```java
		username = new String(username.getBytes("ISO-8859-1"), "UTF-8");
```

