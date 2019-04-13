# Spring自动注入

##### 创建两个`bean` 

```java
package com.cy.autowire;

public class Student {
}

```

```java
public class School {
	Student student;

	public School(Student student) {
		super();
		this.student = student;
	}

	public School() {
		super();
		// TODO Auto-generated constructor stub
	}

	public Student getStudent() {
		return student;
	}

	public void setStudent(Student student) {
		this.student = student;
	}
	
}

```

##### 配置xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd"
        default-autowire="byType"
        >
	<bean class="com.cy.autowire.School" id="school213"> </bean>
	<bean class="com.cy.autowire.Student" id="student"></bean>
	
</beans>
```

在`beans`配置 `default-autowire` 来指定全局的默认自动装配方式

也可以在`bean`中配置`autowire`指定局部的自动装配方式

```xml
	<bean class="com.cy.autowire.School" id="school213" autowire="byType"> </bean>
```





##### default-autowire/autowire的取值

- byType

  容器中不能出现两个相同类型的bean，否则抛出异常

- byName

  在容器中找对应的id注入

- constructor

  根据构造方法注入，容器中的候选对象要足够匹配构造方法的参数

- default

  对于default-autowire，default的值是no；对于autowire，default的值是default-autowire的值

- no

  不自动装配

在使用constructor以外的注入时必须要提供默认构造函数。

