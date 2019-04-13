#### Spring整合Mybatis框架

---

1. 导包：

   - Mybatis lib下的所有的包
   - mybatis-spring.jar
   - jdbc
   - Spring核心包+commons-logging(四加一)
   - spring-tx.jar,spring-jdbc.jar,spring-aop.jar

2. 创建pojo包

   ```java
   package com.cy.pojo;
   
   public class People {
   	private int id;
   	private String name;
   	private int age;
   	public int getId() {
   		return id;
   	}
   	public void setId(int id) {
   		this.id = id;
   	}
   
   	public String getName() {
   		return name;
   	}
   	public void setName(String name) {
   		this.name = name;
   	}
   	public int getAge() {
   		return age;
   	}
   	public void setAge(int age) {
   		this.age = age;
   	}
   	@Override
   	public String toString() {
   		return "People [id=" + id + ", name=" + name + ", age=" + age + "]";
   	}
   }
   
   ```

3. 创建mapper包

   ```java
   package com.cy.mapper;
   
   import java.util.List;
   
   import org.apache.ibatis.annotations.Select;
   
   import com.cy.pojo.People;
   
   public interface PeopleMapper {
   	
   	@Select("select * from people")
   	List<People> selectAll();
   	
   }
   
   ```

4. 在service中添加建Mapper接口对象声明。配置applicationContext.xml并在xml中注入Mapper。

   ```java
   package com.cy.service;
   
   import java.util.List;
   
   import com.cy.mapper.PeopleMapper;
   import com.cy.pojo.People;
   
   public class PeopleServiceImpl implements PeopleService{
   
   	PeopleMapper peopleMapper;
   	
   	public PeopleMapper getPeopleMapper() {
   		return peopleMapper;
   	}
   
   	public void setPeopleMapper(PeopleMapper peopleMapper) {
   		this.peopleMapper = peopleMapper;
   	}
   
   	@Override
   	public List<People> showAll() {
   		return peopleMapper.selectAll();
   	}
   
   }
   ```

   applicationContext.xml中配了四个`bean`。

   - `dataSoucre` 通过把数据库连接信息注入到`org.springframework.jdbc.datasource.DriverManagerDataSource`

     从而整合了Mybatis.xml中的environment

   - `factory` 简化了SqlSessionFactory的创建

   - `mapperScanner` 整合了Mybatis.xml中的mapper

     需要注入的属性有：

     - basePackage

       指定了mapper所在的包，当applicationContext加在bean时，会把对应package下的所有接口都添加为bean。

     - sqlSessionFactory

   - `peopleService` 配置为bean,并为其注入相应的Mapper

   ```xml
   <!-- 干掉environment -->
   	<!-- 配置数据源封装类，位于 Spring.jdbc.jar中 -->
   	<bean id="dataSource"
   		class="org.springframework.jdbc.datasource.DriverManagerDataSource">
   
   		<property name="driverClassName">
   			<value>com.mysql.cj.jdbc.Driver</value>
   		</property>
   
   		<property name="url">
   			<value>jdbc:mysql://localhost:3306/ssm?serverTimezone=UTC</value>
   		</property>
   
   		<property name="username">
   			<value>root</value>
   		</property>
   
   		<property name="password">
   			<value>Cy1269903329</value>
   		</property>
   	</bean>
   
   	<!-- 干掉 创建 SqlsessionFactory对象 , 需要用到environment中的dataSource -->
   	<bean id="factory"
   		class="org.mybatis.spring.SqlSessionFactoryBean">
   		<property name="dataSource" ref="dataSource"></property>
   	</bean>
   	<!-- 干掉mapper -->
   	<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer"
   		id="mapperScanner">
   		<!-- mapper所在包 配置以后，package下的所有接口都会被实例化一个bean -->
   		<property name="basePackage" value="com.cy.mapper"></property>
   		<!-- 所用工厂。 -->
   		<property name="sqlSessionFactory" ref="factory"></property>
   	</bean>
   
   	<!-- 注入Mapper到Service ，由于在 mapperScanner中配置了 basePackage 所以peopleMapper的bean被自动加载-->
   	<bean id="peopleService" class="com.cy.service.PeopleServiceImpl">
   		<!-- 把名字为peopleMapper的bean注入到peopleService的peopleMapper中 -->
   		<property name="peopleMapper" ref="peopleMapper"></property>
   	</bean>
   
   ```





##### 在Servlet使用Spring

---

1. 导包

   ​	在上面的基础上导入spring-web.jar

2. 配置web.xml(放在WEB-INFO下)

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
   	xmlns="http://java.sun.com/xml/ns/javaee"
   	xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
   	id="WebApp_ID" version="3.0">
   	<context-param>
   	<param-name>contextConfigLocation</param-name>
   	<param-value>classpath:applicationContext.xml</param-value>
   	</context-param>
   	<listener>
   		<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
   	</listener>
   </web-app>
   ```

3. 编写servlet

   在`init`中使用 `WebApplicationContextUtils`获取ApplicationContext

   ```java
   @WebServlet("/show")
   public class PeopleServlet extends HttpServlet{
   	@Override
   	public void init() throws ServletException {
   		ApplicationContext applicationContext = WebApplicationContextUtils.getRequiredWebApplicationContext(getServletContext());
   		ps = applicationContext.getBean("peopleService", PeopleService.class);
   	
   	}
   	private PeopleService ps;
   	@Override
   	protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
   		req.setAttribute("list", ps.showAll());
   		req.getRequestDispatcher("index.jsp").forward(req, resp);
   	}
   }
   ```



##### 配置别名

---

```xml
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean" >
	<property name="dataSource" ref="dataSource"></property>
	<property name="typeAliasesPackage" value="com.cy.pojo"></property>
</bean>	
```
注入typeAliasesPackage属性