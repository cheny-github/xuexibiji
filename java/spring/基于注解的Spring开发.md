# 基于注解的Spring开发

### 创建配置类

```java
@Configuration
@ComponentScan(value = "com")
public class SpringCfg {

    @Bean("dog")
    Dog getDog(){
        Dog dog = new Dog();
        dog.setName("好狗");
        return dog;
    }
}
```

注解解释

- @Configuration

  声明该类为Spring的配置类

- @ComponentScan

  配置注解扫描

- @Bean

  Bean注解描述方法，该方法的返回值将会被注入到容器中。名字由bean的value指定。

  



### 获取上下文环境

```java
AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(SpringCfg.class);
```

