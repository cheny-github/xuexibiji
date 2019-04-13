# HTTP协议

跨域请求

> 　在正式post之前，浏览器会先发出一个options请求（也叫preflight），同时header带上origin还有Access-Control-Request-*:**之类的头，服务器响应会返回相应的access-control-allow-origin，如果匹配，那么浏览器就会发送正式post，否则就会出现上述错误。这也解答了，跨域访问时，我们明明发送的post请求，失败的话，查看chrome network会发现是options方法的原因。

