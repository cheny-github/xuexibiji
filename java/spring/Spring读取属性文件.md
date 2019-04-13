# Spring读取属性文件

属性文件的使用主要是方便于部署的环境下。

创建属性文件`sql.properties`

```json
jdbc.driver=com.mysql.cj.jdbc.Driver
jdbc.url =jdbc:mysql://localhost:3306/ssm?serverTimezone=UTC
jdbc.username=root
jdbc.password=Cy1269903329
```



引入`context`命名空间

配置属性文件位置`	<context:property-placeholder location="classpath:sql.properties"/>`

引用属性

```xml
	<bean id="dataSource"
		class="org.springframework.jdbc.datasource.DriverManagerDataSource">
		<property name="driverClassName"
			value="${jdbc.driver}"></property>
		<property name="url"
			value="${jdbc.url}"></property>
		<property name="username" value="${jdbc.username}"></property>
		<property name="password" value="${jdbc.password}"></property>
	</bean>
```

完整xml如下

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:aop="http://www.springframework.org/schema/aop"
	xmlns:context="http://www.springframework.org/schema/context"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/aop
        http://www.springframework.org/schema/aop/spring-aop.xsd
                http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd
        "
	default-autowire="byName">
	
	<context:property-placeholder location="classpath:sql.properties"/>
	
	
	<bean id="dataSource"
		class="org.springframework.jdbc.datasource.DriverManagerDataSource">
		<property name="driverClassName"
			value="${jdbc.driver}"></property>
		<property name="url"
			value="${jdbc.url}"></property>
		<property name="username" value="${jdbc.username}"></property>
		<property name="password" value="${jdbc.password}"></property>
	</bean>



		<bean id="factory"
		class="org.mybatis.spring.SqlSessionFactoryBean">
		<property name="dataSource" ref="dataSource"></property>
		<property name="typeAliasesPackage" value="com.cy.pojo"></property>
	</bean>

	<bean id="mapper"
		class="org.mybatis.spring.mapper.MapperScannerConfigurer">
		<property name="basePackage" value="com.cy.mapper"></property>
		<property name="sqlSessionFactoryBeanName" value="factory"></property>
	</bean>

	<aop:aspectj-autoproxy proxy-target-class="true"></aop:aspectj-autoproxy>
	<context:component-scan base-package="com.cy.advisor,com.cy.serviceImpl"></context:component-scan>

</beans>

```



### 注意！

当使用全局自动注入时，`SqlSessionFactoryBean`的id不能为sqlSessionFactory并且在`MapperScannerConfigurer`不能使用sqlSessionFactory属性进行注入，而是使用`sqlSessionFactoryBeanName`指定value属性进行注入。

原因如下：

​	当使用`default-autowire="byName"`进行自动注入时，`bean`创建后进行属性注入的优先级比读取属性文件的优先级高。所以，当mapper创建时，如果容器里有sqlSessionFactory时会自动注入到mapper中，而factory的创建则需要引用dataSource，此时属性文件还没有读取出来，对dataSource进行属性获取时就会报错。





##### 属性文件中的属性除了可以在xml中使用${}引用还可以通过@Value注解进行注入。（前提是这个类被Spring管理）

1.properties

```json
myString= HelloWorld
```

xml中声明属性文件

```xml
	<context:property-placeholder location="classpath:sql.properties,classpath:1.properties"/>
```

注入属性

```java
	@Value(value = "${myString}")
	String myString;
```

