#### Mybatis缓存

---

Mybatis的缓存执行流程：

​	调用查询语句时，先查看内存中是否缓存有数据。有则返回缓存的数据，没有则去数据库中查询。将数据库中查询的数据返回给应用程序，顺便缓存起来。



缓存分级：

​	Mybatis中缓存分两级。默认是level1(SqlSession级)。

1. SqlSession级：

   对同一个statement对象进行缓存，更细致一点来说就是对`<select>`标签进行缓存。

   ```java
   		InputStream is = Resources.getResourceAsStream("Mybatis.xml");
   		SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(is);
   		SqlSession session = factory.openSession();
   		PeopleMapper peopleMapper = session.getMapper(PeopleMapper.class);
   		//第一次查询，缓存中没有数据，生成了SQL语句。
   		List<People> selectAll = peopleMapper.selectAll();
   		==>  Preparing: select id , name,age from people  
   		==>  Parameters:  
   		<==      Total: 13 
   		//没有生成查询语句，直接返回了上次查询的结果
   		List<People> selectAll2 = peopleMapper.selectAll();
   ```

2. SqlSessionFactory级(二级缓存)：

   对同一个factory产出的SqlSession对象的查询数据进行缓存。

   缓存的时机：当SqlSession对象close/commit时。

   使用二级缓存需要在mapper中进行如下配置：

   ```xml
   <cache readOnly="true"></cache>
   ```

   ```java
   		InputStream is = Resources.getResourceAsStream("Mybatis.xml");
   		SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(is);
   		SqlSession session = factory.openSession();
   		SqlSession session1= factory.openSession();
   		PeopleMapper peopleMapper = session.getMapper(PeopleMapper.class);
   		PeopleMapper peopleMapper2 = session1.getMapper(PeopleMapper.class);
   		
   		List<People> selectAll = peopleMapper.selectAll();
   		//第一次查询缓存中没有数据，生成了sql语句。
   		:Cache Hit Ratio [com.cy.mapper.PeopleMapper]: 0.0 
             ==>  Preparing: select id , name,age from people  
             ==>  Parameters:  
   		  <==      Total: 13 
   		session.close();//flush session的查询结果
   		//第二次查询
   		List<People> selectAll2 = peopleMapper2.selectAll();
   	    :Cache Hit Ratio [com.cy.mapper.PeopleMapper]: 0.5 //缓存命中   
                 
   ```


