##### MyBatis中配置mapper的Sql语句中使用参数： 为语句设置ParameterType属性，表示参数的类型。

```xml
<select id="selectByID" resultMap="PeopleResultMap" parameterType="int">
	select * from people where id=#{0}
</select>
```



- **使用#{}，底层使用的是JDBC的preparedStatement，sql语句中使用?占位符**

1. 参数是基本数据类型，在语句中以#{0},#{1}.....引用参数
2. 参数是基本数据类型，在语句中以#{param1},#{param2}.....引用参数
3. 当只有一个参数时：#{}中的内容可以随意填写
4. 如果参数是对象，#{}中写对象的字段引用参数。
5. 如果参数是map，#{}中写key来引用参数。
6. 如果希望传递多个参数，可以使用对象或map

- **使用${},效果和#{}一样，底层直接拼接SQL语句，不加占位符**

1. 使用${}时，select语句设置ParameterType为Object，${}中写对象的字段，mybatis通过对应字段的getter来获取值，然后拼接sql语句。
2. ${}很少用，二者的区别在面试的时候会考