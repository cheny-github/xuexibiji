#### Spring环境搭建

---

导入核心包：

1. core.jar
2. expression.jar
3. bean.jar
4. context.jar

配置applicationContext——基于schema约束,也就是xmlns(xml namespace)，相对于DTD扩展性更好。

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">
</beans>
```

使用ApplicationContext对象获取已托管的对象：

```java
	ApplicationContext ac = new 	 ClassPathXmlApplicationContext("applicationContext.xml");

		People people = ac.getBean("people",People.class);
```

