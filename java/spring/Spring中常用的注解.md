# Spring中常用的注解

注解要生效，首先要配置要被扫描的包

```xml
<context:component-scan base-package="packagename"></context:component-scan>
```



---

### bean注解

##### @Component

- 相当于`<bean>`

##### @Service

- 与@Component的功能相同，只是语义不同
- 写在ServiceImpl上

##### @Repository

- 与@Component的功能相同，只是语义不同
- 写在数据访问类上。(在ssm中用不上，因为mybatis不用写数据访问类的实体类)

##### @Controller

- 与@Component的功能相同，只是语义不同
- 写在控制器类上



##### 以上四个注解可以相互转换



---

### 注入属性注解

##### 使用注解注入时，不用需要get和set方法了



##### @Resource

- javax.annotation.Resource，JDK的注解
- 默认按照byName注入，如果没有名称对象则byType注入

##### @Autowired

- org.springframework.beans.factory.annotation.Autowired，Spring的注解
- 默认按照byType注入



##### @Value

- 读取属性文件中的内容进行注入

##### @ComponentScan

- 扫描注解

##### @Bean

- 配置Bean

---

### AOP注解

​                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   

##### @PointCut

- 定义切点

##### @Aspect

- 定义切面类

##### @Before

- 前置通知

##### @After

- Final通知

##### @AfterReturnning

- 后置通知

##### @AfterThrowing

- 异常通知

##### @Around

- 环绕通知

