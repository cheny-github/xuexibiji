# 模拟Spring的基于schame的AOP

创建标记接口

```java
//mark interface
public interface Advice {

}
```

```java
//这个也是标记接口
public interface BeforeAdvice extends Advice{

}

```

创建MethodBeforeAdvice

```java
public interface MethodBeforeAdvice extends BeforeAdvice{
	Object before() throws Throwable;
}

```



创建动态代理对象InvocationHandler

```java
public class AdviceHandler implements InvocationHandler{
	private Object target;
	private List<Advice> advices;
	public AdviceHandler(Object target, List<Advice> advices) {
		super();
		this.target = target;
		this.advices = advices;
	}
	@Override
	public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
		//重点
        for (Advice	 advice: advices) {
			if (advice instanceof BeforeAdvice) {
				((MethodBeforeAdvice)advice).before();
			}
		}
		
		Object invoke = method.invoke(target, args);
		
		return invoke;
	}
	
}

```

创建通知类

```java
public class PrivilegeAdvice implements MethodBeforeAdvice{

	private String role;
	
	
	public String getRole() {
		return role;
	}


	public void setRole(String role) {
		this.role = role;
	}


	@Override
	public Object before() throws Throwable{
		if (!role.equals("admin")) {
			throw new RuntimeException("没有权限");
		}
		
		return role;
	}

}
```

```java
public class LogAdvice implements MethodBeforeAdvice{
	SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd hh:mm:ss");
	@Override
	public Object before() throws Throwable {
		System.out.println(sdf.format(new Date()));
		return null;
	}

}

```



绑定切点和通知

```java
public class Test {
	public static void main(String[] args) {
		UserService userService = new UserServiceImpl();
		List<Advice> advices = new ArrayList<>();
		PrivilegeAdvice privilegeAdvice = new PrivilegeAdvice();
		privilegeAdvice.setRole("admin");
		advices.add(privilegeAdvice);
		advices.add(new LogAdvice());
		AdviceHandler userAdvisor = new AdviceHandler(userService, advices);
		UserService proxyInstance = (UserService)Proxy.newProxyInstance(ClassLoader.getSystemClassLoader(), UserServiceImpl.class.getInterfaces(), userAdvisor);
		proxyInstance.eat();
	}
}

输出结果：
2019-01-04 01:11:47
吃饭

如果privilegeAdvice.setRole("admin1");
则抛出异常：
Exception in thread "main" java.lang.RuntimeException: 没有权限
```

