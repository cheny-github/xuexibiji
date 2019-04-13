# 常用API学习

### MD5加密

使用Spring的DigestUtils

```java
DigestUtils.md5DigestAsHex(password.getBytes())
```

jdk1.8的

```java
String newStr = Base64.getEncoder().encodeToString("123".getBytes());
```



