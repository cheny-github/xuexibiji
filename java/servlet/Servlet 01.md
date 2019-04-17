# Servlet 01

#### 什么是Servlet?

Web服务器与应用程序通信的规范，这套规范是以接口的形式体现的。



#### Servlet生命周期

1. 实例化
2. 初始化
3. 服务
4. 销毁

![](F:\笔记\java\servlet\imgs\servlet生命周期.jpg)





#### servlet特征

1. 单例

   一个实例只会执行一次`构造函数`和`init()`方法，而且是在第一次访问时执行

   一个实例只会执行一次`destory()`方法，服务器正常停止时执行

2. 多线程访问service

   用户每提交一次请求时，就会访问一次service方法

   在servlet中定义属性时，要注意线程安全问题

3. 默认情况下，Web容器启动时是不会实例化servlet实例的

   在web.xml的`servlet标签下`中配置`<load-on-startup>1</load-on-startup>`标签来通知Web容器启动时，实例化该servlet。其中，`1`表示启动顺序的优先级，数字越大优先级越低（不要搞负数）。

#### Web容器中的两个Map

​	![](.\imgs\两个map.png)





> Servlet接口中的getServletInfo()
>
> 以字符串的形式返回该Servlet的描述信息，
>
> 例：
>
> public String getServletInfo(){
>
> ​	return "Servlet的版本，Servlet的作者，Servlet所在的应用"
>
> }
>
> 以上内容用处不大了解即可



