### Ajax跨域无法共享同session

解决方法：

> 在控制器上添加@CrossOrigin(allowCredentials = "true",allowedHeaders = "*")
>
> 前端使用jquery的Ajax时，配置项中添加xhrFields:{withCredentials:true}
>
> 使用VueResource的时候
>
> ​    *//全局启用emulateJSON选项*
>
> ​    Vue.http.options.emulateJSON=true;
>
> ​    Vue.http.options.withCredentials=true;
>
> 其他跨域问题可以参考：https://www.jianshu.com/p/d05303d34222