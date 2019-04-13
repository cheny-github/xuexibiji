## 使用Spring的异步

### 创建配置类

​	记得@EnableAsync

```java
@Configuration
@ComponentScan("com.cy")
@EnableAsync
public class Config {
    
}

```

### 将服务注入容器

​	在要执行的异步方法上添加@Async注解

```java

@Service
public class AsyncService {
    @Async
    public void  methodA(){
        while (true) {
            System.out.println("methodA");
        }
    }

    @Async
    public void methodB(){
        while (true) {
            System.out.println("methodB");
        }
    }
}

```

### 调用

```java
    public static void main(String[] args) {
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(Config.class);
        AsyncService service = context.getBean(AsyncService.class);
        service.methodA();
        service.methodB();
    }
```

