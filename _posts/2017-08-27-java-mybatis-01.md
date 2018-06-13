---
layout: post
title:  "mybatis 3.x 基础01"
desc: "mybatis 3.x 基础01"
keywords: "mybatis 3.x 基础01,mybatis,kinglyjn"
date: 2017-08-27
categories: [java]
tags: [java]
icon: fa-coffee
---



### 设计理念

`将SQL交给开发人员，将预编译、参数设置、sql执行、封装结果等步骤交给框架。`

<br>



### 传统使用方式最简单的示例

1、必要的依赖包

```xml
<dependencies>
	<!-- mybatis -->
	<dependency>
		<groupId>org.mybatis</groupId>
		<artifactId>mybatis</artifactId>
		<version>3.4.1</version>
	</dependency>
	<!-- mysql -->
	<dependency>
		<groupId>mysql</groupId>
		<artifactId>mysql-connector-java</artifactId>
		<version>5.1.39</version>
	</dependency>
	<!-- logs 日志 -->
	<dependency>
		<groupId>org.slf4j</groupId>
		<artifactId>slf4j-api</artifactId>
		<version>1.7.24</version>
	</dependency>
	<dependency>
		<groupId>org.slf4j</groupId>
		<artifactId>slf4j-log4j12</artifactId>
		<version>1.7.24</version>
	</dependency>
	<dependency>
		<groupId>log4j</groupId>
		<artifactId>log4j</artifactId>
		<version>1.2.17</version>
	</dependency>
</dependencies>
```

2、代码结构 和 库表准备

```default
/*代码结构*/
test01.sql_session_factory
    |--Emp.java
    |--Test01.java
    |--mybatis-test01-config.xml
    |--mybatis-test01-mapper.xml
    

/*创建数据库*/
create database if not exists test;
/*创建员工表*/
create table if not exists test.t_emp(
	id int not null primary key auto_increment,
	last_name varchar(50),
	email varchar(50),
	gender varchar(10)
);
```

3、mybatis-test01-config.xml（全局配置文件）

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
  PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-config.dtd">
  
<configuration>
	<environments default="development">
		<environment id="development">
			<transactionManager type="JDBC" />
			<dataSource type="POOLED">
				<property name="driver" value="com.mysql.jdbc.Driver" />
				<property name="url" value="jdbc:mysql://192.168.1.96:3306/test" />
				<property name="username" value="xxx" />
				<property name="password" value="xxx" />
			</dataSource>
		</environment>
	</environments>
	<mappers>
		<mapper resource="test01/sql_session_factory/mybatis-test01-mapper.xml" />
	</mappers>
</configuration>
```

4、mybatis-test01-mapper.xml（SQL映射文件）

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
  PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!-- 
	namespace: 名称空间（此处暂时名字任意）
	resultType: 返回数据的类型
	#{xxx}: 传递过来的参数值
-->
<mapper namespace="test01.sql_session_factory">
	<select id="selectEmp" resultType="test01.sql_session_factory.Emp">
		select id, last_name as lastName, email, gender from t_emp where id = #{id}
	</select>
</mapper>
```

5、Emp.java

```java
public class Emp {
	private Integer id;
	private String lastName;
	private String email;
	private String gender;
    // geter/setter ...
}
```

6、Test01.java

```java
/*
 SqlSession对象代表和数据库的一次会话，用完之后必须关闭。
 SqlSession和connection一样都是非线程安全的，每次使用都应该获取新的sqlsession对象。
*/
@Test
public void createSqlSessionFactoryWithXML() throws IOException {
  // 创建 SqlSessionFactory
  String resource = "test01/sql_session_factory/mybatis-test01-config.xml";
  InputStream inputStream = Resources.getResourceAsStream(resource);
  SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
  System.err.println(sqlSessionFactory);

  SqlSession session = null;
  try {
    // 获取 SqlSession
    session = sqlSessionFactory.openSession();
    Emp emp = session.selectOne("selectEmp", 1);
    System.err.println(emp);
  } finally {
    session.close();
  }
}

@Test
public void createSqlSessionFactoryWithoutXML() {
  // 数据源
  Properties properties = new Properties();
  properties.setProperty("driver", "com.mysql.jdbc.Driver");
  properties.setProperty("url", "jdbc:mysql://192.168.1.96:3306/test");
  properties.setProperty("username", "xxx");
  properties.setProperty("password", "xxx");
  PooledDataSourceFactory pooledDataSourceFactory = new PooledDataSourceFactory();
  pooledDataSourceFactory.setProperties(properties);
  DataSource dataSource = pooledDataSourceFactory.getDataSource();
  // 事务
  TransactionFactory transactionFactory = new JdbcTransactionFactory();
  //
  Environment environment = new Environment("development", 
                                            transactionFactory, dataSource);
  Configuration configuration = new Configuration(environment);
  configuration.addMapper(Mapper.class);
  SqlSessionFactory sqlSessionFactory = 
    new SqlSessionFactoryBuilder().build(configuration);
  System.err.println(sqlSessionFactory);
}
```

<br>



### 使用接口方式(推荐)的最简单的示例

1、在方式一的基础上增加 EmpMapper 接口

```java
// 接口式编程，将DAO层的定义和实现解耦
public interface EmpMapper {
	Emp getEmpById(Integer id);
}
```

2、mapper映射文件绑定 Mapper接口

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
  PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!--
1、命名空间namespace必须与Mapper接口全类名相同
2、sql语句id必须与Mapper接口的方法名相同
-->
<mapper namespace="test02.sql_session_factory.EmpMapper">
	<select id="getEmpById" resultType="test02.sql_session_factory.Emp">
		select id, last_name as lastName, email, gender from t_emp where id = #{id}
	</select>
</mapper>
```

3、测试类

```java
@Test
public void createSqlSessionFactoryWithXML() throws IOException {
  // 创建 SqlSessionFactory
  String resource = "test02/sql_session_factory/mybatis-test02-config.xml";
  InputStream inputStream = Resources.getResourceAsStream(resource);
  SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
  System.err.println(sqlSessionFactory);

  // 通过 session 获取 mapper
  SqlSession session = null;
  try {
    session = sqlSessionFactory.openSession();
    EmpMapper empMapper = session.getMapper(EmpMapper.class);
    Emp emp = empMapper.getEmpById(1);
    System.err.println(empMapper); // xxx.MapperProxy@87f383f
    System.err.println(emp);
  } finally {
    session.close();
  }
}
```

<br>



### 使用注解映射SQL（代码和SQL耦合，不推荐使用）

1、编写mapper接口

```java
public interface EmpAnnotationMapper {
	@Select("select * from t_emp where id=#{id}")
	Emp getEmpById(Integer id);
}
```

2、引入SQL映射接口

```xml
<mappers>
  <mapper class="test03.configuration.EmpAnnotationMapper"/>
</mappers>
```

3、测试（略）

<br>



### 全局配置文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
  PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-config.dtd">
  
<configuration>
	<!-- 
	 properties: 
	 properties标签引入外部properties配置文件中的内容（一般的spring项目会把配置工作交给spring来做）
	 resource: 引入外部properties配置文件
	 url: 引入网络或磁盘路径下的properties配置文件
	 -->
	<properties resource="test03/configuration/jdbc.properties">
		<!-- peoperty ... -->
	</properties>


	<!-- 
	 settings:
	 包含了众多极为重要的配置项
	-->
	<settings>
		<!-- 参照: http://www.mybatis.org/mybatis-3/zh/configuration.html -->
		<setting name="mapUnderscoreToCamelCase" value="true"/>
	</settings>
	
	
	<!-- 
	 tyleAliases:
	 别名处理器，可以为java类型起别名
	 typeAlias: 为java类型起别名，type指定全类型，alias指定别名（如果不指定则默认为类名首字母小写）
	 package: 为某个包及其下的子包的所有类批量起别名（默认别名为类名首字母小写，实际上别名不区分大小写）
	 注意：如果一个包下的子包中也有相同别名的类，可以使用 @Alias 注解为类指定新的别名以避免冲突
	 建议：写全类名以方便开发
	 参考：常见的Java类型内建的相应的类型别名参照 
          http://www.mybatis.org/mybatis-3/zh/configuration.html
	-->
	<typeAliases>
		<!-- <typeAlias type="test03.configuration.Emp"/> -->
		<package name="test03.configuration"/>
	</typeAliases>
	
	
	<!-- 
	 typeHandlers:
	 无论是 MyBatis 在预处理语句（PreparedStatement）中设置一个参数时，还是从结果集中取出一个值时， 
	 都会用类型处理器将获取的值以合适的方式转换成 Java 类型。mybatis定义了一些默认的类型处理器。另外
	 你可以重写类型处理器或创建你自己的类型处理器来处理不支持的或非标准的类型。
	 具体做法为: 实现 org.apache.ibatis.type.TypeHandler 接口，或继承一个很便利的类 
	 org.apache.ibatis.type.BaseTypeHandler，然后可以选择性地将它映射到一个 JDBC 类型。
	 -->
	<typeHandlers>
		<!-- typeHandler ... -->
	</typeHandlers>
	

	<!-- 
	 plugins:
	 MyBatis允许你在已映射语句执行过程中的某一点进行拦截调用（实际还是代理）。
	 默认情况下，MyBatis允许使用插件来拦截的方法调用包括：
	 Executor (update, query, flushStatements, commit, rollback, 
               getTransaction, close, isClosed)
	 ParameterHandler (getParameterObject, setParameters)
	 ResultSetHandler (handleResultSets, handleOutputParameters)
	 StatementHandler (prepare, parameterize, batch, update, query)
	 使用插件是非常简单的，只需实现 Interceptor 接口，并指定想要拦截的方法签名即可。
     使用插件需要很小心，因为如果在试图修改或重写已有方法的行为的时候，你很可能会修改MyBatis的核心模块。
	 -->
	 <!-- 
	 <plugins>
	 	<plugin interceptor="org.mybatis.example.ExamplePlugin">
	 		<property name="someProperty" value="100"/>
	 	</plugin>
	 </plugins>
	 -->


	<!-- 
	 environments:
	 1、可以配置多个environment，但创建SessionFactory的时候只能选择其一。
	 2、每个environment标签下必须包含 transactionManager 和 dataSource 两个子标签。
	 3、事务管理器transactionManager的可选值有 JDBC|MANAGED。
	    JDBC: 这个配置就是直接使用了 JDBC 的提交和回滚设置，它依赖于从数据源得到的连接来管理事务作用域。
	    MANAGED: 这个配置几乎没做什么。它从来不提交或回滚一个连接，而是让容器来管理事务的整个生命周期（
	    比如 JEE 应用服务器的上下文）。默认情况下它会关闭连接，然而一些容器并不希望这样，因此需要使用其下
	    property子标签将closeConnection 属性设置为false来阻止它默认的关闭行为。
	    其实这两个可选值它们是类型别名（参照 org.apache.ibatis.session.Configuration），换句话说，
	    通过实现 TransactionFactory接口 可以自定义事务，
        并使用实现类的完全限定名或类型别名代替 JDBC|MANAGED。
	    注意：如果你正在使用 Spring + MyBatis，则没有必要配置事务管理器，
	         因为Spring模块会使用自带的管理器来覆盖前面的配置。
	 4、数据源dataSource的可选值有 UNPOOLED|POOLED|JNDI。
	    通过实现 DataSourceFactory接口 可以设置自定义的数据源（eg. c3p0、druid等）
	-->
	<environments default="development">
		<environment id="development">
			<transactionManager type="JDBC" />
			<dataSource type="POOLED">
				<property name="driver" value="${jdbc.driver}" />
				<property name="url" value="${jdbc.url}" />
				<property name="username" value="${jdbc.username}" />
				<property name="password" value="${jdbc.password}" />
			</dataSource>
		</environment>
	</environments>
	
	
	<!-- 
	databaseIdProvider:
	Mybatis对多数据库厂商的支持，方法如下：
	1、定义数据库厂商标识的别名：多数据库厂商支持，作用就是根据数据库厂商的标识。驱动包实现方法
	  DatabaseMetaData#getDatabaseProductName可以获取厂商标识。由于通常情况下这个字符串
	  都非常长而且相同产品的不同版本会返回不同的值，所以最好通过设置属性别名来使其变短。
	2、设置具体sql语句相关标签的databaseId属性为当前数据库厂商标识别名，如：
	  <select id="getEmpById" resultType="emp" databaseId="mysql">xxx<select>
	3、配置相应数据库的 environment，并设置为默认环境。
	-->
	<databaseIdProvider type="DB_VENDOR">
		<property name="MySQL" value="mysql"/>
	  	<property name="Oracle" value="oracle" />
		<property name="SQL Server" value="sqlserver"/>
	  	<property name="DB2" value="db2"/>    
	</databaseIdProvider> 
	
	
	<!-- 
	 mappers:
	 1、使用映射文件映射SQL(推荐)
	 将SQL映射注册到全局配置中。
	 resource：引用类路径下的sql映射文件
	 url：引用网络路径下的sql映射文件
	 package：用于批量注册，要求mapper映射文件(如果有)和mapper接口必须在同一个包下，并且名称相同。
	 class：a、引用mapper接口，如果使用class注册mapper接口，那么mapper
	          映射文件必须和mapper接口在同一个目录下，并且具有相同的名称。
	        b、class标签也可以直接注册使用注解映射SQL的mapper接口。
	 
	 2、使用注解映射SQL
	 在这种情况下，所有的SQL都是使用注解写在接口上，并且使用class标签将接口注册进来。
	-->
	<mappers>
		<package name="test03.configuration"/>
		<!-- 
		<mapper resource="test03/configuration/EmpMapper.xml" />
		<mapper class="test03.configuration.EmpAnnotationMapper"/>
		-->
	</mappers>
</configuration>
```

<br>



### 简单的增查改删

1、定义mapper接口

```java
public interface EmpMapper {
	void addEmp(Emp emp);
  	void addEmp2(Emp emp);
  	void addEmp3(Emp emp);
	Emp getEmpById(Integer id);
	void updateEmp(Emp emp);
	void deleteEmp(Integer id);
  	itn deleteEmp2(Integer id);
}
```

2、编写SQL映射文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
  PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="test04.mapper.crud.EmpMapper">
  	<!-- mybatis底层调用Statement#getGeneratedKeys 获取生成的主键并赋值给封装对象的某个属性 -->
	<insert id="addEmp" databaseId="mysql" useGeneratedKeys="true" keyProperty="id">
		insert into t_emp(last_name,email,gender) values(#{lastName},#{email},#{gender})
	</insert>
    <!-- 使用 selectKey 标签在sql执行前后查询主键并封装到目标对象的某个属性 -->
	<insert id="addEmp2" databaseId="oracle">
		<selectKey keyProperty="id" resultType="integer" order="BEFORE">
			select emp_seq.nextval from dual
		</selectKey>
		insert into t_emp(id,last_name,email,gender) 
      		values(#{id},#{lastName},#{email},#{gender})
	</insert>
	<insert id="addEmp3" databaseId="oracle">
		<selectKey keyProperty="id" resultType="integer" order="AFTER">
			select emp_seq.currentval from dual
		</selectKey>
		insert into t_emp(id,last_name,email,gender) 
      		values(emp_seq.nextval,#{lastName},#{email},#{gender})
	</insert>
	<select id="getEmpById" resultType="emp" databaseId="mysql">
		select * from t_emp where id = #{id}
	</select>
	<update id="updateEmp">
		update t_emp set last_name=#{lastName},email=#{email},gender=#{gender} 
        where id=#{id}
	</update>
	<delete id="deleteEmp">
		delete from t_emp where id=#{id}
	</delete>
  	<delete id="deleteEmp2">
		delete from t_emp where id=#{id}
	</delete>
</mapper>
```

3、在全局配置文件中批量注册mapper

```xml
<mappers>
  <package name="test04.mapper.crud"/>
</mappers>
```

4、测试

```java
public SqlSessionFactory getSqlSessionFactory() throws IOException {
  String resource = "test04/mapper/crud/mybatis-test04-config.xml";
  InputStream inputStream = Resources.getResourceAsStream(resource);
  return new SqlSessionFactoryBuilder().build(inputStream);
}

@Test
public void addEmp() {// addEmp2 addEmp3 同理
  SqlSession session = null;
  try {
    SqlSessionFactory sf = getSqlSessionFactory();
    session = sf.openSession(); // autoCommit=false
    EmpMapper mapper = session.getMapper(EmpMapper.class);
    Emp emp = new Emp("wangwu", "wangwu@keyllo.com", "1");
    mapper.addEmp(emp);
    System.out.println("插入后的员工对象（含有生成的主键）: " + emp);
  } catch (Exception e) {
    e.printStackTrace();
  } finally {
    session.commit();
    session.close();
  }
}

@Test
public void getEmp() {
  SqlSession session = null;
  try {
    SqlSessionFactory sf = getSqlSessionFactory();
    session = sf.openSession();
    EmpMapper mapper = session.getMapper(EmpMapper.class);
    Emp emp = mapper.getEmpById(1);
    System.out.println(emp);
  } catch (Exception e) {
    e.printStackTrace();
  } finally {
    session.close();
  }
}

@Test
public void updateEmp() {
  SqlSession session = null;
  try {
    session = getSqlSessionFactory().openSession();
    EmpMapper mapper = session.getMapper(EmpMapper.class);
    Emp emp = new Emp("wangwu2", "wangwu2@keyllo.com", "1");
    emp.setId(4);
    mapper.updateEmp(emp);
  } catch (IOException e) {
    e.printStackTrace();
  } finally {
    session.commit();
    session.close();
  }
}

@Test
public void deleteEmp() { // deleteEmp2 同理
  SqlSession session = null;
  try {
    session = getSqlSessionFactory().openSession();
    EmpMapper mapper = session.getMapper(EmpMapper.class);
    mapper.deleteEmp(4);
  } catch (IOException e) {
    e.printStackTrace();
  } finally {
    session.commit();
    session.close();
  }
}
```

<br>



### Mybatis对映射参数的处理(ParamNameResolver)

```default
1、单个参数
名称任意，直接 #{xxx}

2、多个参数
多个参数会被封装成一个map，map的key为 param1..paramN，map的value为传入的参数值，
所以在映射sql取值的时候，应当使用 #{param1},..,#{paramN} 或者 #{0},..,#{N-1} 的方式

3、多个参数使用命名参数(推荐)
首先定义mapper接口的时候，使用 @Param 注解定义参数的名称，然后就可以在mapper映射文件中使用自定义的参数名
a. Emp findEmpByIdAndName(@Param("id") Integer id, @Param("lastName") String lastName);
b. select * from t_emp where id=#{id} and last_name=#{lastName}

4、多个参数是POJO的属性时可以使用POJO传递参数，取值时采用 #{属性名} 方式
有时为方便会专门封装TO（Transfer Object）对象用于传递参数(eg. Page)
a. void updateEmp(Emp emp);
b. update t_emp set last_name=#{lastName},email=#{email},gender=#{gender} where id=#{id}

5、多个参数没有对应POJO时也可以使用map进行参数传递，取值时采用 #{map的key} 方式
a. Emp findEmpByIdAndName(Map map);
b. select * from t_emp where id=#{id} and last_name=#{lastName}

6、如果参数是集合或数组，Mybatis也会把集合或数组封装在map中，map的key即为集合或者数组对象。
如果是List类型的参数还可以使用 "list" 作为取值的键，如果是数组则还可以使用 "array" 作为取值的键
List<Emp> findEmpByIds1(Set<Integer> ids);
<select id="findEmpByIds1" resultType="emp">
  select * from t_emp where id in
  <foreach collection="collection" item="id" open="(" close=")" separator=",">
      #{id}
  </foreach>
</select>

Emp findEmpByIds2(List<Integer> ids);
<select id="findEmpByIds2" resultType="emp">
	select * from t_emp where id=#{list[0]}
</select>

Emp findEmpByIds3(Integer[] ids);
<select id="findEmpByIds3" resultType="emp">
	select * from t_emp where id=#{array[0]}
</select>


一些示例：
-------
Emp getEmp(@Param("id")Integer id, String lastName);
取值：id=#{id/param1}  last_name=#{param2}

Emp getEmp(Integer id, @Param("emp")Emp emp);
取值：id=#{param1}  last_name=#{param2.lastName/emp.lastName}
```

<br>









