# SpringMVC参数传递

1. ##### 自动装配基本数据类型

   ```java
   	@RequestMapping("demo")
   	public String demo(String username,String password) throws UnsupportedEncodingException {
   		username = new String(username.getBytes("ISO-8859-1"), "UTF-8");
   		System.out.println(username);
   		System.out.println(password);
   		return "/main.jsp";
   	}
   ```

2. ##### 自动装配对象

   ```java
   	@RequestMapping("demo2")
   	public String demo2(User user) throws UnsupportedEncodingException {
   		System.out.println("进入demo2控制器");
   		user.setUsername(new String(user.getUsername().getBytes("ISO-8859-1"),"UTF-8"));
   		System.out.println(user.getUsername());
   		System.out.println(user.getPassword());
   		return "/main.jsp";
   	}
   ```

3. ##### 装配HttpServletRequest和HttpServletResponse

   ```java
   	@RequestMapping("demo3")
   	public void demo3(User user,HttpServletRequest res ,HttpServletResponse resp) throws IOException {
   		resp.sendRedirect("js/2.html");
   	}
   ```

4. ##### 使用`@RequestParm`

   配置参数映射

   ```java
   	@RequestMapping("demo4")
   	public String demo4(@RequestParam("name")String username,@RequestParam("pwd")String password) throws IOException {
   		username = new String(username.getBytes("ISO-8859-1"), "UTF-8");
   		System.out.println(username);
   		System.out.println(password);
   		return "/main.jsp";
   	}
   	
   ```

   `@RequestParam`中 defaultValue配置参数的默认值。

   `@RequestParam`中 required表示的配置参数必须存在。

##### 5.传递RESTful风格参数

```java
@RequestMapping("demo8/{id1}/{name}")
public String demo8(@PathVariable String  name,@PathVariable("id1") int age){
　　System.out.println(name +" "+age);
　　return "/main.jsp";
}
```

