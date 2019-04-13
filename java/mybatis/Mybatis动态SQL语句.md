#### Mybatis动态SQL语句。

---

​	什么是动态SQL语句：在Mapper中加入控制流语句来动态生成SQL语句。

在接口中添加：

```java
	List<People> selectDynamic(
			@Param(value = "id") Integer id 
			, @Param(value = "name") String name 
			,@Param(value = "age") Integer age);	
```

在Mapper中添加：

```xml
	<select id="selectDynamic" resultType="People">
		select * from people where 1=1
		
		<if test="name!=null and name!=''">
			and name=#{name}
		</if>
		
		<if test="id!=null and id!=''">
			and id=#{id}
		</if>
		
		<if test="age!=null and age!=''">
			and age=#{age}
		</if>
	</select>
```

结果根据调用时传入的参数确定。

```java
	 List<People> namedZhangsan = peopleMapper.selectDynamic(null, "张三", null);

	生成的SQL语句如下
 		 	==> Preparing: select * from people where 1=1 and name=?  
	 		==> Parameters: 张三(String) 

	 
```

1. `<where>`相对于自己手写where少写一个"1=1 and"，提升了一个1=1的性能(微乎其微)。

   where生成语句时，第一个and是多余的话，会被自动删除掉。

```xml
	<select id="selectDynamic" resultType="People">
		select * from people
		<where>
			<if test="name!=null and name!=''">
				and name=#{name}
			</if>

			<if test="id!=null and id!=''">
				and id=#{id}
			</if>

			<if test="age!=null and age!=''">
				and age=#{age}
			</if>
		</where>
	</select>
```



2. `<choose> <when> <otherwise>`,类似于switch语句

   ```java
   	List<People> selectDynamic(
   			@Param(value = "id") Integer id 
   			, @Param(value = "name") String name 
   			,@Param(value = "age") Integer age);
   ```

   ```xml
   	<select id="selectDynamic" resultType="People">
   		select * from people
   		<where>
   			<choose>
   			<when test="name!=null and name!=''">and name=#{name}</when>
   			<when test="id!=null and id!=''">and id=#{id}</when>
   			<when test="age!=null and age!=''">	and age=#{age}</when>
   			<otherwise>1=1</otherwise>
   			</choose>
   		</where>
   	</select>	
   ```

   ```java
   	 List<People> namedZhangsan = peopleMapper.selectDynamic(1, "张三", 21);
    		==>  Preparing: select * from people WHERE name=?  
           ==> Parameters: 张三(String) 
               
        List<People> allpeople = peopleMapper.selectDynamic(null, null, null);
   	 	==>  Preparing: select * from people WHERE 1=1 
           ==> Parameters:  
   ```




2. ` <set>` set标签中生成的语句会去掉最后一个逗号。为了避免生成update语句错误，set标签里的第一句用"id=#{id},".

   ```java
   int update(People people);
   ```

   ```xml
   	<update id="update" parameterType="People">
   		update people 
   		<set>
   			id=#{id},
   			<if test="name!=null and name!=''">
   				name = #{name},
   			</if>
   			<if test="age!=null and age!=''">
   				age=#{age},
   			</if>
   		</set>
   		<where>
   			id=#{id}
   		</where>
   	</update>
   ```

   ```java
   		People zhangsan = new People();
   		zhangsan.setAge(20);
   		zhangsan.setName("张三三");
   		zhangsan.setId(1);
   		peopleMapper.update(zhangsan);
   			==>  Preparing: update people SET id=?, name = ?, age=? WHERE id=?  
       		==>  Parameters: 1(Integer), 张三三(String), 20(Integer), 1(Integer) 
   ```

3.  `<bind>` 在使用属性生成SQL语句前通过bind标签可以修改属性的值。

   ```java
   List<People> selectByKeyWord(@Param(value = "keyword") String keyword);
   ```

   ```xml
   	<select id="selectByKeyWord" resultType="People">
   		<if test="keyword!=null and keyword!=''">
   			<bind name="keyword" value="'%'+keyword+'%'" />
   		</if>
   		select * from people
   		<where>
   			<if test="keyword!=null and keyword!=''">
   				 name like '${keyword}'
   			</if>
   		</where>
   	</select>
   ```

   ```java
   People keyWordPeople = peopleMapper.selectByKeyWord("张").get(0);
   	==>  Preparing: select * from people WHERE name like '%张%'
   	==>  Parameters:  
   ```

4. `<foreach>`  常用于in 查询中

   ```java
   List<People> whereIdIn(List<Integer> idList);
   ```

   ```xml
   <select id="whereIdIn" resultType="People" parameterType="list">
   		select * from people where id in
   		<trim suffixOverrides="," suffix=")">
   			<foreach collection="list" open="(" item="id">
   				#{id},
   			</foreach>
   		</trim>
   </select>
   ```

   ```java
   		List<Integer> list= new ArrayList<>();
   		list.add(1);
   		list.add(2);
   		List<People> people = peopleMapper.whereIdIn(list);
   			 ==> Preparing: select * from people where id in ( ?, ? )  
       		 ==> Parameters: 1(Integer), 2(Integer) 
   	
   ```

5. `<sql>` 定义可以复用的片段。常用于多表联合查询

   ```xml
   <sql id="people">
   	id , name,age 
   </sql>
   <select id="selectAll" resultType="People">
   	select <include refid="people"></include>
   	from people
   </select>
   ```

   ```java
   List<People> selectAll = peopleMapper.selectAll();
   	==> select id , name,age from people 
   	==> Parameters:  
   ```


