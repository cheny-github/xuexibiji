### Mybatis接口绑定。

---

1. 在Mybatis.xml中添加如下配置信息:

   ```xml
   <mappers>
   	<package name="com/cy/mapper"/>
   </mappers>
   ```

2.  在com.cy.mapper包下创建

   - PeopleMapper.java
   - PeopleMapper.xml

3. 在接口中添加

   ```java
   public interface PeopleMapper {
   	List<People> selectAll();
   	People selectByIdAndName(
           @Param(value = "id") int id,
           @Param(value = "name") String name);
   }
   ```

4. 在PeopleMapper.xml中添加：

```xml
<mapper namespace="com.cy.mapper.PeopleMapper">
	<select id="selectAll" resultMap="People">
		select * from people
	</select>

	<!-- 多个参数时不需要写参数类型，由接口来指定 -->
	<select id="selectByIdAndName" resultType="People">
		select * from people where id=#{id} and name=#{name}
	</select>
</mapper>
```



**注意：**

1. **`xml中mapper的namespace的值和接口的全类名一致。`**
2. **`xml中sql语句的id和接口的中方法名一致。`**
3. **`xml中sql语句的参数由在接口方法用注解@param标识。`**



