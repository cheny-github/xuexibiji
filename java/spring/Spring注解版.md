# Spring注解版

## 声明配置类

`@Configuration`

spring的注解驱动开发，不需要xml文件作为配置文件了。通过声明配置类来对容器进行配置。使用`@Configuration`在类上声明该类为Spring的配置类。

```java

@Configuration
public class MyConfig {

    @Bean
    public Person person(){
        return  new Person("张三",20);
    }
}

```

## 加载上下文环境

```java
AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(MyConfig.class);
```

## 包扫描

`@ComponentScan`

在配置类上添加`@ComponentScan`注解，声明要扫描的包

```java
@Configuration
@ComponentScan({"com.cy"})
public class MyConfig {

    @Bean
    public Person person(){
        return  new Person("张三",20);
    }
}
```

### 使用类型过滤器排除要扫描的项目

```java
@ComponentScan(value = {"com.cy"},excludeFilters = {
        @ComponentScan.Filter(type = FilterType.ANNOTATION,classes = {Controller.class, Service.class})
})
```

使用自定义类型的过滤器

```java
@ComponentScan(value = {"com.cy"},
        includeFilters =  @ComponentScan.Filter(type = FilterType.CUSTOM,classes = {MyTypeFilter.class}
        )
        ,useDefaultFilters = false
)
```

在使用`includeFilters`时需要配置useDefaultFilters为false

其中`MyTypeFilter.class`是TypeFilter接口的实现类。

​	接口中match方法的返回值为true这表示该类通过。

```java
public class MyTypeFilter implements TypeFilter {
    @Override
    public boolean match(MetadataReader metadataReader, MetadataReaderFactory metadataReaderFactory) throws IOException {

        ClassMetadata classMetadata = metadataReader.getClassMetadata();
        if (classMetadata.getClassName().contains("er")) {
            return true;
        }
        return false;
    }
}
```

## 在AppplicationContext中获取到系统的环境信息

```java
ConfigurableEnvironment environment = context.getEnvironment();
String property = environment.getProperty("os.name");
```

## 作用域

使用`@Scope`指定作用域，常用的取值"singleton","prototype"

- "singleton"表示bean为单例。

- "prototype"则是多例

```java
    @Bean
    @Scope("prototype")
    public Person person(){
        System.out.println("person创建");
        return  new Person("张三",20);
    }
```

## 懒加载

在使用单例作用域时，容器初始化的时候bean默认就会被加载到容器中。为组件添加上`@Lazy`注解后，当从容器中获取组件时，该组件才注入到容器中。

```java
@Bean
@Lazy
public Person person(){
    System.out.println("person创建");
    return  new Person("张三",20);
}
```

## @Conditional

根据条件决定是否将某个组件注入到容器中。

```java
public @interface Conditional {
    Class<? extends Condition>[] value();
}
```

其value是condition接口的实现类。

举例，

根据操作系统决定是否注入，bill和linus。

```java
@Bean("bill")
@Conditional({WindowsCondition.class})
public Person person01(){
    System.out.println("比尔创建");
    return new Person("比尔盖茨",66);
}

@Bean("linus")
@Conditional({LinuxCondition.class})
public Person person02(){
    System.out.println("林纳斯创建");
    return new Person("林纳斯",45);
}
```

其中`WindowsCondition`的实现如下

```java
public class WindowsCondition implements Condition {
    @Override
    public boolean matches(ConditionContext conditionContext, AnnotatedTypeMetadata annotatedTypeMetadata) {
        Environment environment = conditionContext.getEnvironment();
        String osName = environment.getProperty("os.name");
        if (osName.contains("Windows")) {
            return true;
        }
        return false;
    }
}
```

linuxCondition

```java
public class LinuxCondition  implements Condition {
    @Override
    public boolean matches(ConditionContext conditionContext, AnnotatedTypeMetadata annotatedTypeMetadata) {
        Environment environment = conditionContext.getEnvironment();
        String osName = environment.getProperty("os.name");
        if (osName.contains("Linux")) {
            return true;
        }

        return false;
    }
}
```

检查Person类型的bean

```java
   public static void main( String[] args )
    {
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(MyConfig.class);
        Map<String, Person> beans = context.getBeansOfType(Person.class);
        beans.entrySet().forEach((entry)->{
            System.out.println(entry.getKey());
            System.out.println(entry.getValue());
        });
    }
```



## @Import

​	导入组件到容器中的另外一种方式。

使用Import 将Yellow对象注入到容易

```java
package com.cy.pojo;

public class Yellow {
}
```

```java
@Configuration
@Import(Yellow.class)
public class SpringCfg {
}
```

```java
@Test
public void Test2(){
    AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(SpringCfg.class);
    String[] definitionNames = context.getBeanDefinitionNames();
    for (String name : definitionNames) {
        System.out.println(name);
    }
}
输出如下：
org.springframework.context.annotation.internalConfigurationAnnotationProcessor
org.springframework.context.annotation.internalAutowiredAnnotationProcessor
org.springframework.context.event.internalEventListenerProcessor
org.springframework.context.event.internalEventListenerFactory
springCfg
com.cy.pojo.Yellow
```



### ImportSelector接口

使用`@Import`导入组件时，可以给value赋值一个`ImportSelector`接口的实现类，该实现类中的`public String[] selectImports(AnnotationMetadata annotationMetadata) `方法返回要导入的组件的全类名。

```java
@Configuration
@Import({MyImportSelector.class})
public class SpringCfg {

}
```

```java
public class MyImportSelector implements ImportSelector {
    @Override
    public String[] selectImports(AnnotationMetadata annotationMetadata) {
        return  new String[]{"com.cy.pojo.Red","com.cy.pojo.Red"};
    }
}
```

```java
package com.cy.pojo;

public class Red {
}
```

```java
package com.cy.pojo;

public class Green {
}
```

```java
@Test
public void Test2(){
    AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(SpringCfg.class);
    String[] definitionNames = context.getBeanDefinitionNames();
    for (String name : definitionNames) {
        System.out.println(name);
    }
}

```

> 输出结果如下：
> org.springframework.context.annotation.internalConfigurationAnnotationProcessor
> org.springframework.context.annotation.internalAutowiredAnnotationProcessor
> org.springframework.context.event.internalEventListenerProcessor
> org.springframework.context.event.internalEventListenerFactory
> **springCfg**
> **com.cy.pojo.Red**
> **com.cy.pojo.Green**



### ImportBeanDefinitionRegistrar

在使用`@Import`时指定`ImportBeanDefinitionRegistrar`的实现类时，容器会调用该类中的`    registerBeanDefinitions(AnnotationMetadata annotationMetadata, BeanDefinitionRegistry beanDefinitionRegistry)`方法，通过`beanDefinitionRegistry`对象注入组件到容器中。

示例：

如果Dog和Girl在容器中，则把Green注入到容器中

```java
public class MyImportBeanDefinitionRegistrar implements ImportBeanDefinitionRegistrar {

    @Override
    public void registerBeanDefinitions(AnnotationMetadata annotationMetadata, BeanDefinitionRegistry beanDefinitionRegistry) {
        boolean b = beanDefinitionRegistry.containsBeanDefinition("com.cy.pojo.Dog");
        boolean b1 = beanDefinitionRegistry.containsBeanDefinition("com.cy.pojo.Girl");
        if (b&&b1){
            RootBeanDefinition beanDefinition = new RootBeanDefinition(Green.class);
            //传入bean的名字，和一个BeanDefinition对象
         beanDefinitionRegistry.registerBeanDefinition("green",beanDefinition);

        }
    }
}
```

```java
@Configuration
@Import({Dog.class, Girl.class, MyImportBeanDefinitionRegistrar.class})
public class SpringCfg {

}
```

```java
@Test
public void Test2(){
    var context = new AnnotationConfigApplicationContext(SpringCfg.class);
    var definitionNames = context.getBeanDefinitionNames();
    for (var name : definitionNames) {
        System.out.println(name);
    }
}
```

> 输出：
>
> org.springframework.context.annotation.internalConfigurationAnnotationProcessor
> org.springframework.context.annotation.internalAutowiredAnnotationProcessor
> org.springframework.context.event.internalEventListenerProcessor
> org.springframework.context.event.internalEventListenerFactory
> **springCfg**
> **com.cy.pojo.Dog**
> **com.cy.pojo.Girl**
> **green**



## FactoryBean接口

spring容器调用接口中的getObject方法创建对象并将其注入到容器中。

```java
package com.cy.pojo;

import org.springframework.beans.factory.FactoryBean;


public class DogFactoryBean implements FactoryBean<Dog>{
    @Override
    public Dog getObject() throws Exception {
        System.out.println("Dog is created");
        return  new Dog();
    }

    @Override
    public Class<?> getObjectType() {
        return Dog.class;
    }

    @Override
    public boolean isSingleton() {
        return true;
    }
}
```

将工厂Bean注入到容器中 

```java
@Configuration
public class SpringCfg {
    @Bean
    DogFactoryBean dogFactoryBean(){
        return  new DogFactoryBean();
    }
}
```

```java
@Test
public void showDog(){
    var context = new  AnnotationConfigApplicationContext(SpringCfg.class);
    //获取工厂生产的对象
    Object dog = context.getBean("dogFactoryBean");
    System.out.println(dog.getClass());
    
    //获取工厂对象
    Object dogFactory = context.getBean("&dogFactoryBean");
    System.out.println(dogFactory.getClass());
}
```



## 容器中对象的生命周期

所谓的生命周期就是一个钩子函数(钩子函数注册在某个事件下，当事件发生时，钩在事件下的所有钩子函数都会被调用)。

Spring中对象的生命周期分为以下三个阶段

1. 构造

   - 单实例构造：容器创建时调用
   - 多实例构造：从容器中获取对象时调用

   

   BeanPostProcessor.postProcessBeforeInitialization

2. 初始化

   待容器中对象构造，属性赋值完毕后调用

   BeanPostProcessor.postProcessAfterInitialization

   

3. 销毁

- 单实例时容器关闭时会调用。
- 多实例时请手动调用

示例：

### Singleton情况

```java
public class Car {
    public Car() {
        System.out.println("构造....");
    }

    public void init(){
        System.out.println("init...");
    }

    public void destroy(){
        System.out.println("destroy.....");
    }
}
```

在配置类中注册Bean,并指定初始化和销毁时的钩子函数

```java
@Configuration
public class SpringCfg {

    @Bean(destroyMethod = "destroy",initMethod = "init")
    Car car(){
        return  new Car();
    }
}
```

```java
@Test
public void lifeCycle(){
    AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(SpringCfg.class);
    context.close();
}
```

> 输出：
>
> 构造....
> init...
> destroy.....

### prototype情况

```java
@Configuration
public class SpringCfg {

    @Scope("prototype")
    @Bean(destroyMethod = "destroy",initMethod = "init")
    Car car(){
        return  new Car();
    }
}
```

```java
@Test
public void lifeCycle(){
    AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(SpringCfg.class);
    Car bean = context.getBean(Car.class);
    context.close();//容器关闭时也没有调用destroy方法
}
```

> 输出：
>
> 构造....
> init...

### InitializingBean,DisposableBean接口

实现这两个接口可以分别指定bean的初始化行为和销毁时的行为

```java
package com.cy.pojo;

import org.springframework.beans.factory.DisposableBean;
import org.springframework.beans.factory.InitializingBean;
import org.springframework.stereotype.Component;

@Component
public class Cat implements InitializingBean, DisposableBean {
    public Cat() {
        System.out.println("cat构造函数调用.....");
    }

    @Override
    public void destroy() throws Exception {
        System.out.println("cat销毁.....");
    }

    @Override
    public void afterPropertiesSet() throws Exception {
        System.out.println("cat初始化....");
    }
}
```

```java
@Configuration
@ComponentScan("com.cy.pojo")
public class SpringCfg {
    
}
```

```java
@Test
public void lifeCycle(){
    AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(SpringCfg.class);
    context.close();
}
```

> 输出：
>
> cat构造函数调用.....
> cat初始化....
> cat销毁.....



### @PostContruct,@PreDestroy

> post作为词根词缀时的意思是 在...之后
>
> postwar 战后

没有这两个注解的话要先导入javax.annotation-api依赖

```java
<dependency>
    <groupId>javax.annotation</groupId>
    <artifactId>javax.annotation-api</artifactId>
    <version>1.2</version>
</dependency>
```

```java

@Component
public class Phone {
    public Phone() {
        System.out.println("Phone's construct has been called ....");
    }



    @PostConstruct
    public void init(){
        System.out.println("Phone 初始化");
    }

    @PreDestroy
    public void destroy(){
        System.out.println("Phone 销毁");
    }
}
```



```java
@Test
public void lifeCycle(){
    AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(SpringCfg.class);
    context.close();
}
```

> 输出：
>
> Phone's construct has been called ....
> Phone 初始化
> Phone 销毁



### BeanPostProcessor接口

AKA,Bean后置处理器

`BeanPostProcessor`接口中的`postProcessBeforeInitialization`会在每个Bean的初始化之前执行(populateBean方法之后)，`postProcessAfterInitialization`会在每个Bean初始化完成后执行

![](F:\笔记\java\spring\imgs\绘图1.jpg)

```java
public class MyBeanPostProcessor implements BeanPostProcessor {
    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("postProcessBeforeInitialization......");
                //一旦方法返回null 后面的后置处理器都不会执行，具体请下断点在调用堆栈中查看源码。
       return bean;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("postProcessAfterInitialization......");
        return bean;
    }
}
```

下面是三个要往容器中注入的Bean

```java
public class Cat implements InitializingBean, DisposableBean {
    public Cat() {
        System.out.println("cat构造函数调用.....");
    }

    @Override
    public void destroy() throws Exception {
        System.out.println("cat销毁.....");
    }

    @Override
    public void afterPropertiesSet() throws Exception {
        System.out.println("cat初始化....");
    }
}
```

```java
public class Car {
    public Car() {
        System.out.println("构造....");
    }

    public void init(){
        System.out.println("Car init...");
    }

    public void destroy(){
        System.out.println("Car destroy.....");
    }
}
```

```java
public class Phone {
    public Phone() {
        System.out.println("Phone's construct has been called ....");
    }



    @PostConstruct
    public void init(){
        System.out.println("Phone 初始化");
    }

    @PreDestroy
    public void destroy(){
        System.out.println("Phone 销毁");
    }
}
```

配置类中注册BeanPostProcessor

```java
@Configuration
public class SpringCfg {

    @Bean
    public MyBeanPostProcessor myBeanPostProcessor(){
        return  new MyBeanPostProcessor();
    }

    @Bean
    public Phone phone(){
        return  new Phone();
    }

    @Bean(initMethod = "init" ,destroyMethod = "destroy")
    public Car car(){
        return new Car();
    }

    @Bean
    public Cat cat(){
        return  new Cat();
    }
}
```

测试类

```java
@Test
public void lifeCycle(){
    AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(SpringCfg.class);
    context.close();
}
```

> 输出：
>
> Phone's construct has been called ....
> postProcessBeforeInitialization......
> Phone 初始化
> postProcessAfterInitialization......
> 构造....
> postProcessBeforeInitialization......
> Car init...
> postProcessAfterInitialization......
> cat构造函数调用.....
> postProcessBeforeInitialization......
> cat初始化....
> postProcessAfterInitialization......
> cat销毁.....
> Car destroy.....
> Phone 销毁

#### 几个Spring自带的BeanPostProcessor接口的实现类

- ApplicationContextAwareProcessor

  该实现类主要是重写了postProcessBeforeInitialization方法， 给实现ApplicationContextAware了接口的Bean注入ApplicationContext

  ```java
  public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
  ....
   this.invokeAwareInterfaces(bean); //通过此方法注入上下文环境
  ....
  }
  ```

  ```java
  private void invokeAwareInterfaces(Object bean) {
  ....
   if (bean instanceof ApplicationContextAware)       ((ApplicationContextAware)bean).setApplicationContext(this.applicationContext);
                                       
  }
  ```

  实现ApplicationContextAware接口的Bean：

  ```java
  @Component
  public class Dog implements ApplicationContextAware {
  
      private ApplicationContext applicationContext;
      
      @Override
      public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
          this.applicationContext=applicationContext;
      }
      public ApplicationContext getApplicationContext() {
          return applicationContext;
      }
      
  ```

  测试类：

  ```java
  @Test
  public void getContext(){
      var context = new AnnotationConfigApplicationContext(SpringCfg.class);
      Dog bean = context.getBean(Dog.class);
      System.out.println("Application Context is:");
      System.out.println(bean.getApplicationContext());
  }
  ```

  > 输出：
  >
  > Application Context is:
  > org.springframework.context.annotation.AnnotationConfigApplicationContext@11dc3715, started on Tue Mar 19 22:14:27 CST 2019

- AutowiredAnnotationBeanPostProcessor

  在Bean初始化完成后，检查@Autowired标注的字段给其注入依赖。

- BeanValidationPostProcessor

  给Bean进行赋值检查，数据校验

- InitDestroyAnnotationBeanPostProcessor

  对标记了@PostConstruct和@PreDestroy注解的方法进行Bean的生命周期注册

## @Value

使用该注解可以为Bean注入值类型数据，有三种使用方式

1. 传入值类型

2. SpringEL(使用#{})

3. 注入properties属性，环境变量里的值(使用${})，注入properties中的属性时需要在Configuration类上添加@PropertySource注解(相当于xml文件中的 context:property-placeholder标签)，注意是在Configuration类上添加！

   在Configuration类上添加！

   在Configuration类上添加！

   重要的事说三遍。

示例：

```properties
#bean.properties
person.nickName=小张
```

```java

public class Person {
    @Value("张三")
    private String name;
    @Value("${person.nickName}")
    private String nickName;
    @Value("#{20-2}")
    private Integer age;
}
```

```java
@PropertySource(value = {"classpath:/bean.properties"})
@Configuration
public class SpringCfg02 {
    @Bean
    public Person person(){
        return new Person();
    }
}
```

```java
@Test
public void setValue(){
    AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(SpringCfg02.class);
    Person person = context.getBean(Person.class);
    System.out.println(person);
}
```

> 输出：
>
> Person{name='张三', nickName='小张', age=18}

