# Spring注解版02

## @AutoWired

- 优先按照类型装配，如果容器中存在同类型的多个实例，则根据被注入对象的名字装配。
- 可以指定required值，这个值默认为true，表示容器中必须要有可以用来注入的实例对其进行注入。当改为false时，容器中没有适合的实例注入时程序也能启动。
- `@AutoWired`和`@Qualifier`同时使用时，由@Qualifier给出要注入的bean的名称，通过给定的名称在容器中查找Bean然后进行注入。
- `@Primary`标记的Bean在使用@AutoWired进行注入时，优先注入它。通常在容器中有多个同类型的bean时使用。
- 当@Primary和@Qualifier和@AutoWired同时使用时，通过@Qualifier指定的Bean名称进行注入。

- 构造器，方法，参数，属性都可以标记@AutoWired

  1. 放在方法上时，由容器调用方法并将自定义类型的参数在容器中寻找并注入。

  2. 标在构造器上时与标在方法上类似。如果只有一个有参构造器，可以不用写@AutoWired，容器也会自动调用并注入。

  3. 在@Configuration标记的类中添加了@Bean的方法所需的参数也是自动从容器中获取注入的，这种方法很常用。

     ```java
         @Bean
         Green green(Boss boss){
             Green green = new Green();
             green.setBoss(boss);
             return green;
         }
     ```

     

### @Resource

java提供的注入规范，优先通过名称进行注入，指定name属性从容器中获取实例，若给出name的值，则默认从容器中获取与字段名相同的bean。不支持@Primary。不能指定required。



### @Inject

也是java提供的注入规范。功能和@AutoWired一样。但是需要导包javax inject，不能指定required。



## @Profile

根据当前环境动态激活组件。

例如：我们有开发环境，测试环境，生产环境。根据环境的不同我们需要启用和禁用某些组件，通过在组件上添加@Profile注解指定该组件所存在的环境。

示例：

准备一个c3p0数据源组件，根据环境的不同，选择不同的Bean注入到容器。

为了复习之前的知识，数据源需要的4个属性通过以下三种方法注入

- username，通过@Value从db.properties中获取
- password, 在作为方法参数时使用@Value注入
- url和driver，使用StringValueResolver读取db.properties

下面代码中，每个组件都被标记了@Profile，当容器启动时，根据环境加载组件。

```java

@Configuration
@PropertySource("classpath:/db.properties")
public class SpringCfgProfile implements EmbeddedValueResolverAware {

    @Value("${jdbc.username}")
    private String username;
    private StringValueResolver stringValueResolver;
    private String url;
    private String driver;

    @Profile("test")
    @Bean("dataSourceTest")
    DataSource dataSourceTest(@Value("${jdbc.password}") String pwd) throws Exception {
        ComboPooledDataSource source = new ComboPooledDataSource();
        source.setUser(username);
        source.setPassword(pwd);
        source.setJdbcUrl(url);
        source.setDriverClass(driver);
        return source;
    }

    @Profile("dev")
    @Bean("dataSourceDev")
    DataSource dataSourceDev(@Value("${jdbc.password}") String pwd) throws Exception {
        ComboPooledDataSource source = new ComboPooledDataSource();
        source.setUser(username);
        source.setPassword(pwd);
        source.setJdbcUrl("jdbc:mysql://103.40.18.90:3306/studentInfoManagement");
        source.setDriverClass(driver);
        return source;
    }

    @Profile("prod")
    @Bean("dataSourceProd")
    DataSource dataSourceProd(@Value("${jdbc.password}") String pwd) throws Exception {
        ComboPooledDataSource source = new ComboPooledDataSource();
        source.setUser(username);
        source.setPassword(pwd);
        source.setJdbcUrl(url);
        source.setDriverClass("com.mysql.jdbc.Driver");
        return source;
    }

    @Override
    public void setEmbeddedValueResolver(StringValueResolver stringValueResolver) {
        this.stringValueResolver = stringValueResolver;
        driver = stringValueResolver.resolveStringValue("${jdbc.driver}");
        url = stringValueResolver.resolveStringValue("${jdbc.url}");
    }
}

```

AnnotationConfigApplicationContext的有参构造器如下,我们使用无参构造器时，也应该有以下2个步骤

1. register配置类
2. refresh容器

```java
public AnnotationConfigApplicationContext(Class... annotatedClasses) {
    this();
    this.register(annotatedClasses);
    this.refresh();
}
```

测试类如下

​	通过ConfigurableEnvironment.addActiveProfile方法来指定环境。

```java
@Test
public void profileTest(){
    AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext();
    ConfigurableEnvironment environment = context.getEnvironment();
    environment.addActiveProfile("test");
    context.register(SpringCfgProfile.class);
    context.refresh();
    String[] beanDefinitionNames = context.getBeanDefinitionNames();
    for (String name : beanDefinitionNames) {
        System.out.println(name);
    }
}
```

> 输出结果如下：
>
> Initializing c3p0-0.9.5.2 [built 08-December-2015 22:06:04 -0800; debug? true; trace: 10]
> org.springframework.context.annotation.internalConfigurationAnnotationProcessor
> org.springframework.context.annotation.internalAutowiredAnnotationProcessor
> org.springframework.context.annotation.internalCommonAnnotationProcessor
> org.springframework.context.event.internalEventListenerProcessor
> org.springframework.context.event.internalEventListenerFactory
> springCfgProfile
> **dataSourceTest**

​	也可以通过设置启动参数来指定环境



![./im](.\imgs\TIM截图20190324132002.jpg)

```java
AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(SpringCfgProfile.class);
for (String definitionName : context.getBeanDefinitionNames()) {
    System.out.println(definitionName);
}
```

> 输出：
>
> Initializing c3p0-0.9.5.2 [built 08-December-2015 22:06:04 -0800; debug? true; trace: 10]
> org.springframework.context.annotation.internalConfigurationAnnotationProcessor
> org.springframework.context.annotation.internalAutowiredAnnotationProcessor
> org.springframework.context.annotation.internalCommonAnnotationProcessor
> org.springframework.context.event.internalEventListenerProcessor
> org.springframework.context.event.internalEventListenerFactory
> springCfgProfile
> Initializing c3p0-0.9.5.2 [built 08-December-2015 22:06:04 -0800; debug? true; trace: 10]
> org.springframework.context.annotation.internalConfigurationAnnotationProcessor
> org.springframework.context.annotation.internalAutowiredAnnotationProcessor
> org.springframework.context.annotation.internalCommonAnnotationProcessor
> org.springframework.context.event.internalEventListenerProcessor
> org.springframework.context.event.internalEventListenerFactory
> springCfgProfile
> **dataSourceProd**



## AOP

### 切面编程的编写步骤

1. 导入aop模块：Spring Aop(spring-aspects)

   ```xml
   <dependency>
       <groupId>org.springframework</groupId>
       <artifactId>spring-aspects</artifactId>
       <version>5.1.4.RELEASE</version>
   </dependency>
   ```

2. 编写业务类

   ```java
   public class Calculator {
       public Integer div(int a,int b){
           System.out.println("a / b="+a/b);
           return  a/b;
       }
   }
   ```

3. 编写切面

   - 前置通知----Before

   - 后置通知----After，无论方法是否执行成功该通知都会执行

   - 方法返回通知----AfterReturnning ，只有当方法正确返回了该通知才会执行

   - 异常抛出通知----AfterThrowing

   - 环绕通知---Around

     **编写Around的方法时应注意，方法不写返回值 后面的AfterReturning就不能get到返回值。**

     

     ```java
     @Aspect
     public class CalculatorAspects {
         Logger logger =Logger.getLogger(CalculatorAspects.class);
     
         //抽取公共切点表达式，如果要在本类引用，直接写方法名。其他类引用写方法的全限定名
         @Pointcut("execution(* com.cy.aop.Calculator.div(int ,int))")
         public void pointCut(){
     
         }
     
         @Before("pointCut()")
         public void before(JoinPoint joinPoint){
             logger.info("已于"+joinPoint.getSignature().getName()+"执行之前切入");
         }
     
         @After(value = "pointCut()")
         public void after(JoinPoint joinPoint){
             logger.info("已于"+joinPoint.getSignature().getName()+"执行之后切入，无论方法是否成功返回！！！===>不能get返回结果，因为不确定是否能获得返回值");
         }
     
         @AfterReturning(value = "pointCut()",returning ="result")
         public void afterReturning(JoinPoint joinPoint,Object result){
             logger.info("已于"+joinPoint.getSignature().getName()+"执行之后切入，只有方法成功返回才切入===>执行结果为"+result);
         }
     
         @AfterThrowing(value = "pointCut()",throwing = "exception")
          public void  afterThrowing(Exception exception){
             logger.info("切获异常===>"+exception);
         }
     
     //
         @Around("pointCut()")
         public Object around(ProceedingJoinPoint joinPoint){
             Object result =null;
             System.out.println("环绕通知前.....");
             try {
                 result=  joinPoint.proceed();
             } catch (Throwable throwable) {
                 throwable.printStackTrace();
             }finally {
                 System.out.println("环绕通知后.....");
                 return result;
             }
     
         }
     
     }
     ```

4. 将切面和被切入的业务类加入容器

   ```java
   @Configuration
   @EnableAspectJAutoProxy
   public class SpringCfgAop {
   
       @Bean
       public Calculator calculator(){
           return  new Calculator();
       }
   
       @Bean
       public CalculatorAspects calculatorAspects(){
           return  new CalculatorAspects();
       }
   
   }
   ```

5. 必须告诉Spring哪个类是切面类（添加@Aspect注解）

6. 开启基于注解版的切面功能(在配置类上添加@EnableAspectJAutoProxy)

### 不抛异常时各Advice的执行情况

```java
@Test
public void  AopTest(){
    AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(SpringCfgAop.class);
    Calculator calculator = context.getBean(Calculator.class);
    calculator.div(1,1);
}
```

> 输出：
>
> 环绕通知前.....
> 已于div执行之前切入
> a / b=1
> 环绕通知后.....
> 已于div执行之后切入，无论方法是否成功返回！！！===>不能get返回结果，因为不确定是否能获得返回值
> 已于div执行之后切入，只有方法成功返回才切入===>执行结果为1

从输出可以看出各Advice的执行顺序：

1. Before
2. Around(Before)
3. After
4. Around(After)
5. AfterReturning



### 抛出异常时各Advice的执行情况

```java
@Test
public void  AopTest(){
    AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(SpringCfgAop.class);
    Calculator calculator = context.getBean(Calculator.class);
    try {
    calculator.div(1,0);
    }catch (Exception e){
        System.out.println("捕获到异常"+e);
    }
}
```

需要注意的是，在使用Around通知时，想要异常被AfterThrowing切获的话需要在Around的方法中把异常往外抛。

```java
@Around("pointCut()")
public Object around(ProceedingJoinPoint joinPoint) throws Throwable {
    Object result =null;
    System.out.println("环绕通知前.....");

        result=  joinPoint.proceed();

        System.out.println("环绕通知后.....");
        return result;


}
```

所以，一个没有坑的标准Around通知的方法必须包含以下两项

1. 返回值
2. throws 异常

> 输出：
>
> 环绕通知前.....
> 已于div执行之前切入
> div执行...
> 已于div执行之后切入，无论方法是否成功返回！！！===>不能get返回结果，因为不确定是否能获得返回值
> 切获异常===>java.lang.ArithmeticException: / by zero
> 捕获到异常java.lang.ArithmeticException: / by zero

执行顺序如下：

1. Around(Before)
2. Before
3. After
4. AfterThrowing





### @EnableAspectJAutoProxy实现细节

![](.\imgs\EnableAspectJAutoProxy.jpg)

点开源码后可以看见，导入了`AspectJAutoProxyRegistrar`这个组件





那么现在对这个组件进行分析：

![](.\imgs\AspectJAutoProxyRegistrar.jpg)

可以发现，这是一个ImportBeanDefinitionRegistrar。

那么ImportBeanDefinitionRegistrar又做了一些什么工作呢？



在接口的实现`registerBeanDefinitions`中，第一行是

```java
AopConfigUtils.registerAspectJAnnotationAutoProxyCreatorIfNecessary(registry);
```

该方法调用做了什么呢？

把**AnnotationAwareAspectJAutoProxyCreator**(命名为org.springframework.aop.config.internalAutoProxyCreator)注入到了容器中。



AnnotationAwareAspectJAutoProxyCreator的功能

继承关系如下

![](.\imgs\AnnotationAwareAspectJAutoProxyCreator.jpg)

可以看见，它实现了BeanFactoryAware和BeanPostProcessor。

**分析AbstractAutoProxyCreator**

------



- 实现了setBeanFactory方法。

  ```java
  public void setBeanFactory(BeanFactory beanFactory) {
      this.beanFactory = beanFactory;
  }
  ```

- 有后置处理器的处理逻辑

  ```java
   public Object postProcessBeforeInstantiation(Class<?> beanClass, String beanName) {
          Object cacheKey = this.getCacheKey(beanClass, beanName);
          if (!StringUtils.hasLength(beanName) || !this.targetSourcedBeans.contains(beanName)) {
              if (this.advisedBeans.containsKey(cacheKey)) {
                  return null;
              }
  
              if (this.isInfrastructureClass(beanClass) || this.shouldSkip(beanClass, beanName)) {
                  this.advisedBeans.put(cacheKey, Boolean.FALSE);
                  return null;
              }
          }
  ```

  ```java
      public Object postProcessAfterInitialization(@Nullable Object bean, String beanName) {
          if (bean != null) {
              Object cacheKey = this.getCacheKey(bean.getClass(), beanName);
              if (!this.earlyProxyReferences.contains(cacheKey)) {
                  return this.wrapIfNecessary(bean, beanName, cacheKey);
              }
          }
  
          return bean;
      }
  ```

  

  

  **分析AbstractAdvisorAutoProxyCreator**

  ------

  

  - 重写了**setBeanFactory**

    主要是判断这个factory是不是ConfigurableListableBeanFactory**并调用**

    **`initBeanFactory`方法**

    ```java
    public void setBeanFactory(BeanFactory beanFactory) {
        super.setBeanFactory(beanFactory);
        if (!(beanFactory instanceof ConfigurableListableBeanFactory)) {
            throw new IllegalArgumentException("AdvisorAutoProxyCreator requires a ConfigurableListableBeanFactory: " + beanFactory);
        } else {
            this.initBeanFactory((ConfigurableListableBeanFactory)beanFactory);
        }
    }
    ```

  

  

  **分析AnnotationAwareAspectJAutoProxyCreator**

------

​	重写了`initBeanFactory`方法，在父类调用setBeanFactory的时候调该方法咯。

- ```java
  protected void initBeanFactory(ConfigurableListableBeanFactory beanFactory) {
      super.initBeanFactory(beanFactory);
      if (this.aspectJAdvisorFactory == null) {
          this.aspectJAdvisorFactory = new ReflectiveAspectJAdvisorFactory(beanFactory);
      }
  
      this.aspectJAdvisorsBuilder = new AnnotationAwareAspectJAutoProxyCreator.BeanFactoryAspectJAdvisorsBuilderAdapter(beanFactory, this.aspectJAdvisorFactory);
  }
  ```



**总流程**

1.**创建BeanPostProcessor(AnnotationAwareAspectJAutoProxyCreator)**

------

1. `new AnnotationConfigApplicationContext(SpringCfgAop.class);`

   主要做了如下操作，调用无参构造方法，注册配置类，刷新容器

   ```java
   this();
   this.register(annotatedClasses);
   this.refresh();
   ```

2. `this.refresh();`

   注册后置处理器

   ```java
   public void refresh() throws BeansException, IllegalStateException {
   		...
           this.registerBeanPostProcessors(beanFactory);
           ...
           }
   ```

   **registerBeanPostProcessors**干了什么呢？

   1. 获取IOC容器中已定义的后置处理器
   2. 添加第三方后置处理器
   3. 注册实现了PriorityOrdered接口的后置处理器
   4. 注册实现了Ordered接口的后置处理器，根据getOrder()方法获取后置处理器注入顺序
   5. 注册没实现优先级接口的后置处理器

   

   3.在`registerBeanPostProcessors`方法中创建注册**internalAutoProxyCreator**

   - 创建Bean实例(AnnotationAwareAspectJAutoProxyCreator)
   - populateBean，给Bean的各种属性赋值
   - initializeBean
     1. invokeAwareMethods().处理Aware接口的方法回调
     2. 调用postProcessBeforeInitialization
     3. 调用invokeInitMethod
     4. 调用postProcessAfterInitialization

   4. BeanPostProcessor创建成功(AnnotationAwareAspectJAutoProxyCreator)并添加到BeanFactory中



2.`AnnotationAwareAspectJAutoProxyCreator`在拦截Bean的创建

---

