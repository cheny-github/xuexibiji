#### spring创建对象的三种方式

---

1. 构造方法

   1. 无参构造

      ```xml
      <bean id="peopleNonArg" class="com.cy.bean.People"></bean>
      ```

   2. 有参构造

      ```xml
      	<bean id="peopleByIdName" class="com.cy.bean.People">
      		<constructor-arg index="0" name="name"
      			type="java.lang.String" value="张三">
      		</constructor-arg>
      		<constructor-arg index="1" name="id" type="int"
      			value="20">
      		</constructor-arg>
      	</bean>
      ```

      使用有参构造时，应注意构造函数重载的问题。

      - 之所以会出现 `index`,`name`,`type`三个属性，就是为了解决重载时对构造函数的定位。
      - 任选上述属性的组合来指定构造函数时，没有得到唯一的构造函数，则选择类中第一个构造函数作为该bean的构造函数。
      - 还需要注意的是：`int`和`java.lang.Integer`作为构造函数的参数时也有重载。（推广：基本类型和包装类型）
      - 当同时指定了index,name,type三个属性的时候可以唯一确定一个构造函数。

2. 实例工厂

   ​	需要指定 工厂类，然后 通过 ``factory-bean``  和`factory-method` 引用工厂

   ​	在`bean` 中指定`constructor-arg` 来给出工厂的参数

   ```xml
   	<bean class="com.cy.factory.PeopleFactory" id="PeopleFactory">
   	</bean>
   
   	<bean id="getPolice" factory-bean="PeopleFactory"
   		factory-method="newInstance">
   		<constructor-arg index="0" value="police"></constructor-arg>
   	</bean>
   ```

3. 静态工厂

   只需要配置一个`bean` 给出 `factory-method`即可

   ```xml
   	<bean class="com.cy.factory.PeopleFactory" factory-method="newInstance" id="getPolice">
   		<constructor-arg index="0" value="police"></constructor-arg>
   	</bean>
   ```
