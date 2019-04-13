## SpringMVC返回Json

在控制器方法上添加@ResponseBody表示返回的是json字符串。

该注解主要完成以下两个功能：

- 将要返回的对象转换为Json字符串并设置MIME类型为application/json,charset=utf-8。转换时使用jackson类库，因此需要导入jackson的相关jar
- 在不添加该注解时，控制器返回的是一个资源的跳转。添加以后，直接返回。

若方法的返回值不满足key-value格式，则转化为text/html返回。

