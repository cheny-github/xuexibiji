# SpringMVC作用域传值的几种方式

#### servlet作用域

- page

  当前页面不会被重新实例化

- request

  同一个请求下都是相同的对象

- session

  在一个会话内，根据Cookie的失效时间来决定session的存在时间

- application

  tomcat启动到关闭时的单例对象。