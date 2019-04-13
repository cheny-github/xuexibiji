

## 什么是面向切面编程

Aspect Oriented Programming,在程序原有纵向流程中，针对某一个或者某一些方法添加通知，形成横切面的过程，就叫切面编程。

## 常用概念

- 原有功能：切点,point cut
- 前置通知：在切点执行之前的功能，before advice
- 后置通知：在切点执行之后的功能，after advice
- 如果切点执行过程中出现异常 会触发异常通知 throws advice
- 所有功能的总称叫做切面
- 织入：把切面嵌入到原有功能的过程叫织入

##　环境搭建

- 4个核心包+1个commons-log
- spring-aop.jar
- spring-aspects.jar
- spring-tx.jar
- aopalliance.jar
- apectjweaver.jar

![1545894608690](C:\Users\陈勇\AppData\Roaming\Typora\typora-user-images\1545894608690.png)

##### 引入XSD

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:aop="http://www.springframework.org/schema/aop"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
         http://www.springframework.org/schema/aop
        http://www.springframework.org/schema/aop/spring-aop.xsd
        ">

</beans>
```



## 基于schema的aop（jdk动态代理）

通过实现`AfterReturningAdvice`和`MethodBeforeAdvice`接口来实现要织入的通知。

```java
public class Demo {
	public void demo1(String arg) {
		System.out.println("demo1");
	}
	public void demo2() {
		
		System.out.println("demo2");
		
	}
	public void demo3() {
		System.out.println("demo3");
	}
}

```

后置通知

```java
public class MyAfterAdvice implements AfterReturningAdvice{

	@Override
	public void afterReturning(Object arg0, Method arg1, Object[] arg2, Object arg3) throws Throwable {
	System.out.println("MyAfter Advice");
	}
}

```

前置通知

```java
public class MyBeforeAdvice implements MethodBeforeAdvice {

	@Override
	public void before(Method arg0, Object[] arg1, Object arg2) throws Throwable {
		System.out.println("method before");
	}



}
```

然后通过配置xml完成织入​

```xml
      <!-- 配置要织入的通知 -->
<bean id="mybefore" class="com.cy.test.MyBeforeAdvice"></bean>
<bean id="myafter" class="com.cy.test.MyAfterAdvice"></bean>

<aop:config>
	<!-- 配置切入点 -->
	<aop:pointcut expression="execution(* com.cy.test.Demo.demo2()) " id="myPointcut"/>
	
	<!-- 注入通知 -->
	<aop:advisor advice-ref="mybefore" pointcut-ref="myPointcut"/>
	<aop:advisor advice-ref="myafter" pointcut-ref="myPointcut"/>
</aop:config>

<!-- 用于测试 -->
<bean id="demo" class="com.cy.test.Demo"></bean>
```

基于schema的AOP主要是实现Advice接口 然后通过配置 `aop:config`里的`aop:pointcut`和`aop:advisor`来完成织入。

#### `aop:pointcut`中的`expression`

`execution`表示JoinPoint的类型，例如`call` 和 `execution`等，最常用的是`execution`

`call`的切入方式

```java
Call（Before）
Pointcut{
    Pointcut Method
}
Call（After）
```

`execution`的切入方式

```
Pointcut{
  execution（Before）
    Pointcut Method
  execution（After）
}
```



##### `*`可以匹配任意方法名，任意类名，任意一级包名。

第一个`*`表示返回值类型 任意

第二个`*`个表示方法名称任意

```xml
<aop:pointcut expression="execution(* com.cy.test.Demo.*()) " id="myPointcut"/>
```

输出结果如下:

```java
demo1
method before
demo2
MyAfter Advice
method before
demo3
MyAfter Advice

```





##### 如果需要匹配任意类型参数则 在括号里匹配`..`

```xml
	<aop:pointcut expression="execution(* com.cy.test.Demo.*(..)) " id="myPointcut"/>
```

输出结果

```java
method before
demo1
MyAfter Advice
method before
demo2
MyAfter Advice
method before
demo3
MyAfter Advice

```







## 基于AspectJ的AOP

1. ##### 准备通知

   基于AspectJ的AOP通知，不需要实现接口。

   其中`myAround`环绕通知要传入`ProceedingJoinPoint`接口对操作进行前后划分

   `Myadvice.java`中的方法的参数名应该与`aop`标签下的一致

   ```java
   public void myBefore() {
   	System.out.println("前置通知");
   }
   public void myAfter() {
   	System.out.println("after 后置通知");
   }
   public Object myAround(ProceedingJoinPoint p) throws Throwable {
   	System.out.println("环绕前置");
   	Object object = p.proceed();
   	System.out.println("环绕后置");
   	return object;
   }
   public void myAfterReturnning() {
   	System.out.println("afterReturnning 后置通知");
   }
   public void myExceptionHandler(ArithmeticException e) {
   	
   	System.out.println("异常通知："+e.getMessage());
   }
   ```

2. 准备切面

   ```java
   public void pointcut2() {
   	int wqe =1/0;
   	System.out.println("demo2");
   }
   public void pointcut1(String name) {
   	
   }
   
   public void pointcut1(String name ,int age) {
   	
   }
   ```

3. 配置`applicationContext`

   配置时踩过的坑：

   `aop:pointcut` 中expression切面有参数时的格式。

   `aop:aspect` 要配置一个ref属性，指向要引用的advice类的bean

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
   	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
   	xmlns:aop="http://www.springframework.org/schema/aop"
   	xsi:schemaLocation="http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans.xsd
                   http://www.springframework.org/schema/aop
           http://www.springframework.org/schema/aop/spring-aop.xsd
           ">
   	
   	<bean class="com.cy.test.MyPointcut" id="demo"></bean>	
   	<bean class="com.cy.test.MyAdvice" id="myadvisor"></bean>
   	<aop:config>
   		<aop:pointcut expression="execution(* com.cy.test.MyPointcut.pointcut2())" id="mypointcut1"/>
   		<aop:aspect ref="myadvisor">
   				<aop:before method="myBefore" pointcut-ref="mypointcut1" />
   				<aop:after method="myAfter" pointcut-ref="mypointcut1" />
   				<aop:after-returning method="myAfterReturnning" pointcut-ref="mypointcut1" />
   				<aop:after-throwing method="myExceptionHandler" pointcut-ref="mypointcut1"  throwing="e" />
   				<aop:around method="myAround" pointcut-ref="mypointcut1"/>
   		</aop:aspect>	
   	</aop:config>
   
   </beans>
   ```

4. 参数传递

   ```java
   
   public class MyAdvice {
   	public void myBefore(String arg1) {
   		System.out.println("前置通知"+" name="+arg1);
   	}
   	public void myAfter(String arg1,int arg2) {
   		System.out.println("after 后置通知"+"name="+arg1+"---age="+arg2);
   		
   	}	
   }
   ```

   ```java
   public class MyPointcut {
   
   	public void pointcut1(String name) {
   	}
   
   	public void pointcut1(String name ,int age) {
   		
   	}
   }
   ```

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
   	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
   	xmlns:aop="http://www.springframework.org/schema/aop"
   	xsi:schemaLocation="http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans.xsd
                   http://www.springframework.org/schema/aop
           http://www.springframework.org/schema/aop/spring-aop.xsd
           ">
   	
   	<bean class="com.cy.test.MyPointcut" id="demo"></bean>	
   	<bean class="com.cy.test.MyAdvice" id="myadvisor"></bean>
   	<aop:config>
   		<aop:pointcut expression="execution(* com.cy.test.MyPointcut.pointcut1(String)) and args(arg1)" id="mypointcut2"/>
   		<aop:pointcut expression="execution(* com.cy.test.MyPointcut.pointcut1(String, int)) and args(arg1,arg2)" id="mypointcut3"/>
   		<aop:aspect ref="myadvisor">
   				<!-- 通知中获取切面参数 -->
   				<aop:before method="myBefore" pointcut-ref="mypointcut2" arg-names="arg1"/>
   				<aop:around method="myAfter" pointcut-ref="mypointcut3" arg-names="arg1,arg2"/>
   		</aop:aspect>	
   	</aop:config>
   	
   
   </beans>
   ```






## 配置异常通知的步骤

1. 只有当切点报异常才能触发异常通知

   - 由于切面编程常用于service层，所以service层的异常一定要往外抛，这样spring框架才能帮助我们处理异常。

2. 在spring中AspectJ方式提供了异常通知的办法

   使用方法如下：

   明确要抛出异常的类，接收异常通知的类

   ```java
   	//Demo.java 由该方法抛出异常
   	public void demo1(String arg)  throws Exception {
   		throw new Exception("mother fucker exception");
   	}
   ```

   ```java
   //由该类接收异常通知
   public class MyThrowAdvice {
   	public void myException(Exception e) {
   		System.out.println(e.getMessage());
   	}
   }
   
   ```

   在配置文件中配置相应的关系

   ​	1.将接收通知的类添加到bean中,`aop:aspect` 中引用该类

   ​	2.指定切点,即异常发出的方法/类

   ​	3.`aop:after-throwing` 将发出异常的类作为切点，把异常绑定到接收通知的方法上

   ```xml
   <bean id="mythrow" class="com.cy.test.MyThrowAdvice"></bean>
   <aop:config>
   	<aop:aspect ref="mythrow">
   		<aop:pointcut expression="execution(* com.cy.test.Demo.demo1(..))" id="mypoint"/>
   		<aop:after-throwing method="myException" pointcut-ref="mypoint" throwing="e"/>
   	</aop:aspect>
   </aop:config>
   ```



## 基于注解的AOP

##### 配置xml

```xml
     <!-- Spring不会自动扫描注解，需要在context 中配置要被扫描的包  多个包用逗号分隔-->
	<context:component-scan base-package="com.cy.annotation_aop" >
	</context:component-scan>
		<!-- 自动代理 默认false 为jdk动态代理 true为cglib动态代理 注解都是基于cglib做的，所以必须设置为true -->
		<aop:aspectj-autoproxy proxy-target-class="true"></aop:aspectj-autoproxy>
```



##### 使用注解声明切面类

- 其中`@Component `声明bean，给出的value为bean的名字，不给的话默认获取bean时 类名小写 

- `@Aspect`声明切面类

- 通知(advice)通过以下注解声明

  `@Before`

  Declaring Advice  

  Advice is associated with a pointcut expression and runs before, after, or around method executions matched by the pointcut. The pointcut expression may be either a simple reference to a named pointcut or a pointcut expression declared in place.

  翻译：

  声明通知

  通知和一个切点表达式相关联，被表达式所匹配的方法将会执行对应的Before,After，Around方法。

  切点表达式可以是`对某个切点名字的简单的引用`，也可以是一个`声明到位的切点表达式`(in-place pointcut expression)

  ```java
  //官方例子
  import org.aspectj.lang.annotation.Aspect;
  import org.aspectj.lang.annotation.Before;
  
  @Aspect
  public class BeforeExample {
  		//对某个切点名字的简单的引用
      @Before("com.xyz.myapp.SystemArchitecture.dataAccessOperation()")
      public void doAccessCheck() {
          // ...
      }
  
  }
  ```

  ```java
  @Aspect
  @Component
  public class MyAspect {
  		//声明到位的切点表达式
  	@Before(value = "execution(* com.cy.annotation_aop.CarServiceImpl.drive(String)) and args(driverName)")
  	public void myBefore(String driverName) {
  		System.out.println(driverName+" has started the car");
  	}
  	
  
  }
  ```

  `@After`(Final)

  `@AfterReturning` 

  有时候需要获取到方法的返回值：通过指定`returning`属性来完成。

  ```java
  //官方例子
  import org.aspectj.lang.annotation.Aspect;
  import org.aspectj.lang.annotation.AfterReturning;
  
  @Aspect
  public class AfterReturningExample {
  
      @AfterReturning(
          pointcut="com.xyz.myapp.SystemArchitecture.dataAccessOperation()",
          returning="retVal")
      public void doAccessCheck(Object retVal) {
          // ...
      }
  
  }
  ```

  `	@AfterThrowing`

  触发异常通知，通过指定`throwing`参数来给出抛出的异常的名称，在通知的方法签名中限定接收异常的类型（如果不限定异常的类型，则可以使用Throwable接收参数）。

  ```java
  import org.aspectj.lang.annotation.Aspect;
  import org.aspectj.lang.annotation.AfterThrowing;
  
  @Aspect
  public class AfterThrowingExample {
  
      @AfterThrowing(
          pointcut="com.xyz.myapp.SystemArchitecture.dataAccessOperation()",
          throwing="ex")//此处的ex要与下面的ex相同
      public void doRecoveryActions(DataAccessException ex) {
          // ...
      }
  
  }
  ```

  `	@Around`

  通常用在要以一个线程安全的方式共享方法执行前后状态的情况下。

  Always use the least powerful form of advice that meets your requirements (that is, do not use around advice if before advice would do).翻译：能用单独before/after做的就别用around做。

  Around通知的第一个参数必须是ProceedingJoinPoint

  切点的参数通过`ProceedingJoinPoint`获取

  ```java
  	@Around(value = "execution(* com.cy.annotation_aop.CarServiceImpl.drive(String))"
  			)
  	public void MyAround(ProceedingJoinPoint p) throws Throwable {
  		String name = (String) p.getArgs()[0];
  		System.out.println(name+" before around");
  		p.proceed();
  		System.out.println(name+" after around");
  	}
  ```

  调用结果：

  ```java
  public class Test {
  	@SuppressWarnings("resource")
  	public static void main(String[] args) {
  		ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext1.xml");
  			String[] names = context.getBeanDefinitionNames();
  		CarService carServiceImpl = (CarService) context.getBean("carServiceImpl");
  		carServiceImpl.drive("rick");
  	}
  }
  rick before around
  rick has started the car
  rick is driving the car!
  rick after around
  
  ```

- 注解配置完毕后，通过ApplicationContext获取bean来执行业务。

  ```java
  public class Test {
  	@SuppressWarnings("resource")
  	public static void main(String[] args) {
  		ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext1.xml");
  			String[] names = context.getBeanDefinitionNames();
  		CarService carServiceImpl = (CarService) context.getBean("carServiceImpl");
  		carServiceImpl.drive("rick");
  	}
  }
  ```



​	

## 踩过的坑

### warning no match for this type name:Users[Xlint:invalidAbsoluteTypeName]

#### 切点参数不匹配。

应该是com.cy.pojo.Users



# 第二次学习的笔记

#### 不是被Spring容器管理的类是不能被织入的。



#### 切点表达式

- spring只支持aspectj的execution不支持call

- 表达式1：

  `"execution(void add())"` 

  匹配容器中所有返回值为void 名称为 add的空参方法

- 表达式2：

  `"execution(* *())"`

  匹配容器中任意返回值 任意名称的空参方法

- 表达式3：

  `"execution(* com.cy.service.*.*(..))"`

  匹配service包下的所有方法

- 表达式4：

  `"execution(* com.cy.service.*.*(java.lang.String,int))"`

  匹配service包下的所有参数列表为(String,int)的方法

  在表达式中，包装类和基本类型是严格区分的，不存在自动拆箱和装箱的说法

- 表达式5：

  ``"execution(* com..*.*.*(java.lang.String,int))"``

  `..`表示com下的任意包的任意层

#### 通过JoinPoint窥探切入点的信息

```java
    public void before(JoinPoint joinPoint){
        String kind = joinPoint.getKind();
        System.out.println(kind);
        Object[] args = joinPoint.getArgs();
        System.out.println("参数个数："+args.length);
        for (int i = 0; i < args.length; i++) {
            System.out.println("参数["+i+"]:"+args[i]);
        }
        Signature signature = joinPoint.getSignature();
        System.out.println("signature:" +signature);
        System.out.println(joinPoint.toLongString());
    }
```

#### AfterReturnning通知中获取切点的返回值

```xml
<aop:after-returning method="afterReturnning" pointcut="execution(* com.cy.service.*.*(..))" returning="result"/>
```

```java
public class AfterReturnningAdvice {
    public void afterReturnning(JoinPoint joinPoint, String result){
        System.out.pri ntln(joinPoint.getSignature().getName()+ "的返回值："+result);
    }
}

```

##### After (Finally) Advice

​	在抛出异常后还有执行的建议

#### 使用注解

1. 将切点注入容器

   ```java
   @Component
   public class TestService {
   
       public void saySomething(String words){
           System.out.println(words);
       }
   
       public  String getName(){
           return  "东尼大木";
       }
   }
   ```

2. 注册切面

   ```java
   @Component
   @Aspect
   public class BeforeAdvice2 {
   
   }
   ```

3. 织入

   ```java
   @Component
   @Aspect
   public class BeforeAdvice2 {
       @Before("execution(* com..*(..))")
       public void before2(){
           System.out.println("before2执行");
       }
   
   }
   ```

##### 使用注解时切面的执行顺序

一个切面中最好只包含一种类型的通知，这样就可以通过`@Order`来指定同一类型的通知的执行顺序。

`@Before`的顺序随着Order的值的增大而减小

`@After`的顺序则是随着Order的值的减小而增大

```java
@Order(2)
@Component
@Aspect
public class BeforeAdvice {
    @Before("execution(* com..*(..))")
    void before1(){
        System.out.println("方法before1调用");
    }

}
```

```java
@Order(1)
@Component
@Aspect
public class BeforeAdvice2 {
    @Before("execution(* com..*(..))")
    public void before2(){
        System.out.println("before2执行");
    }

}
```

`before2`先于`before1`执行。

