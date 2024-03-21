# 了解MyBatis

## 什么是MyBatis

MyBatis 是一款优秀的持久层框架，它支持自定义 SQL、存储过程以及高级映射。MyBatis 免除了几乎所有的 JDBC 代码以及设置参数和获取结果集的工作。MyBatis 可以通过简单的 XML 或注解来配置和映射原始类型、接口和 Java POJO（Plain Old Java Objects，普通老式 Java 对象）为数据库中的记录。

[MyBatis中文文档](https://mybatis.net.cn/) 可参考 [https://mybatis.net.cn/](https://mybatis.net.cn/)。

## MyBatis是不是ORM？

什么是ORM？即Object-Relationl Mapping，它的作用是在关系型数据库和对象之间作一个映射，这样，我们在具体的操作数据库的时候，就不需要再去写复杂的SQL语句。

JPA是orm框架标准，MyBatis没有实现JPA，和orm框架的设计思路完全不一样。MyBatis是拥抱sql，而orm则更靠近面向对象。Mybatis是sql mapping框架而不是orm框架，当然orm和Mybatis都是持久层框架。

## MyBatis工作原理

![](http://img.cairbin.top/img/202403211303961.png)

## 使用MyBatis

废话不多说，我们直接上手用下。

### 创建数据库与表

我们来创建一个名称为`db_mybatis`的数据库

还是老样子，我这里以终端为例，你可以用`sqlyog`等工具。
```shell
mysql -uroot
```

有密码的话

```shell
mysql -u root -p
```

执行SQL语句创建数据库

```sql
CREATE DATABASE db_mybatis;
```

创建管理该数据库的用户`db_mybatis`，密码为`db_mybatis`，访问主机为本机可访问`localhost`，如果远程访问请改为`%`。

```sql
CREATE USER 'db_mybatis'@'localhost' IDENTIFIED BY 'db_mybatis';
```

给它管理`db_mybatis`的所有权限

```sql
GRANT ALL ON db_mybatis.* TO 'db_mybatis'@'localhost';
```

刷新权限

```sql
flush privileges;
```

如果你用的MySQL8.0及以上版本，客户端版本较老，由于加密算法问题需要追加以下语句：

```sql
ALERT USER 'db_mybatis'@'localhost' IDENTIFIED WITH mysql_native_password by 'db_mybatis';
```

退出MySQL控制台，返回PC终端
```sql
exit;
```

我们用刚创建的用户连接下mysql

```sql
mysql -u db_mybatis -p
```

然后出现提示，输入密码`db_mybatis`敲击回车，如果密码跟我不一样，这里与你上面的设置一致。

切换数据库到`db_mybatis`
```sql
USE db_mybatis;
```

如果表名称已经存在，会影响我们创建，如果你之前不太重要的话就直接删了，如果想要保留那么新建的表就要修改名字

```sql
DROP TABLE IF EXISTS `customer`;
```

我们创建一张表，名称为`customer`，主键为`id`

如果用标准的SQL语句来写的话，我这里版本为MySQL5.7，可能会报错

如果报错，就替换成以下语句
```sql
CREATE TABLE customer
(
    id int(32) AUTO_INCREMENT,
    username varchar(50) DEFAULT NULL,
    jobs varchar(50) DEFAULT NULL,
    phone varchar(16) DEFAULT NULL,
    PRIMARY KEY(`id`)
);
```

我们插入条数据（这里的电话号码是我瞎编的）

```sql
INSERT INTO customer(id,username,jobs,phone) VALUES(1,'joy','doctor','15544433322');
```

通过`SELECT`语句查看下，看来是成功的
```sql
SELECT * FROM customer;
```

![](http://img.cairbin.top/img/202403211342924.png)


### 创建项目

我们创建一个**Maven项目**，不会的请回Spring初步部分看看。

项目名称为`top.cairbin.test4`，然后修改`pom.xml`添加依赖项

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>top.cairbin</groupId>
  <artifactId>test4</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <packaging>jar</packaging>

  <name>test4</name>
  <url>http://maven.apache.org</url>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
  </properties>

  <dependencies>
    <dependency>
			<groupId>junit</groupId>
			<artifactId>junit</artifactId>
			<version>4.12</version>
			<scope>test</scope>
		</dependency>
		<dependency>
			<groupId>org.mybatis</groupId>
			<artifactId>mybatis</artifactId>
			<version>3.5.5</version>
		</dependency>
		<dependency>
			<groupId>mysql</groupId>
			<artifactId>mysql-connector-java</artifactId>
			<version>8.0.33</version>
		</dependency>
		<dependency>
			<groupId>org.mybatis.generator</groupId>
			<artifactId>mybatis-generator-core</artifactId>
			<version>1.4.0</version>
		</dependency>
  </dependencies>
</project>
```

我们创建一个`resources`文件夹，并`use as source folder`

在里面创建一个`log4j.properties`，可以让我们在控制台看到SQL语句


![](http://img.cairbin.top/img/202403211412967.png)

![](http://img.cairbin.top/img/202403211415900.png)

```properties
# Global logging configuration
log4j.rootLogger=ERROR, stdout
# MyBatis logging configuration...
log4j.logger.top.cairbin.test4=DEBUG
# Console output...
log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=%5p [%t] - %m%n
```

在`src/main/java`下的`top.cairbin.test4`包下创建一个持久化类`Customer`

```java
package top.cairbin.test4;

public class Customer {
	private Integer id;       // 主键id
	private String username; // 客户名称
	private String jobs;      // 职业
	private String phone;     // 电话
	
	public void setId(Integer id) {
		this.id = id;
	}
	
	public Integer getId() {
		return this.id;
	}
	
	public void setUsername(String username) {
		this.username = username;
	}
	
	public String getUsername() {
		return username;
	}
	
	public void setJobs(String jobs) {
		this.jobs = jobs;
	}
	
	public String getJobs() {
		return this.jobs;
	}
	
	public void setPhone(String phone) {
		this.phone = phone;
	}
	
	public String getPhone() {
		return this.phone;
	}
	
	//为了方便测试时输出结果，建议生成toString()方法。
	@Override
	public String toString() {
	     return "Customer [id=" + id + ", username=" + username + 
			          ", jobs=" + jobs + ", phone=" + phone + "]";
	}
}
```

在`top.cairbin.test4`下创建一个映射文件`CustomerMapper.xml`，注意`<mapper>`标签的`namespace`属性。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
    "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!-- namespace表示命名空间 -->
<mapper namespace="top.cairbin.test4.CustomerMapper">
    <!--根据客户编号获取客户信息 -->
	<select id="findCustomerById" parameterType="Integer"
		resultType="top.cairbin.test4.Customer">
		select * from customer where id = #{id}
	</select>
</mapper>
```

在`resources`文件夹里新建一个mybatis核心配置文件`mybatis-config.xml`
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
    "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
	<settings>
		<setting name="mapUnderscoreToCamelCase" value="true" />
		<setting name="logImpl" value="STDOUT_LOGGING" />
	</settings>
	<!--1.配置环境 ，默认的环境id为mysql -->
	<environments default="mysql">
		<!--1.2.配置id为mysql的数据库环境 -->
		<environment id="mysql">
			<!-- 使用JDBC的事务管理 -->
			<transactionManager type="JDBC" />
			<!--数据库连接池 -->
			<dataSource type="POOLED">
				<property name="driver" value="com.mysql.jdbc.Driver" />
				<property name="url"
					value="jdbc:mysql://localhost:3306/db_mybatis" />
				<property name="username" value="db_mybatis" />
				<property name="password" value="db_mybatis" />
			</dataSource>
		</environment>
	</environments>
	<!--2.配置Mapper的位置 -->
	<mappers>
		<mapper resource="top/cairbin/test4/CustomerMapper.xml" />
	</mappers>

</configuration>
```

注意这里的位置
```xml
<mappers>
	<mapper resource="top/cairbin/test4/CustomerMapper.xml" />
</mappers>
```

我们在`src/test/java`目录下创建测试类`MyBatisTest`

```java
package top.cairbin.test4;

import java.io.InputStream;
import java.util.List;

import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
import org.junit.Test;
import top.cairbin.test4.*;

public class MyBatisTest {
	/**
	 * 根据客户编号查询客户信息
	 */
	@Test
	public void findCustomerByIdTest() throws Exception {
		// 1、读取配置文件
		String resource = "mybatis-config.xml";
		InputStream inputStream = 
                     Resources.getResourceAsStream(resource);
		// 2、根据配置文件构建SqlSessionFactory
		SqlSessionFactory sqlSessionFactory = 
                     new SqlSessionFactoryBuilder().build(inputStream);
		// 3、通过SqlSessionFactory创建SqlSession
		SqlSession sqlSession = sqlSessionFactory.openSession();
		// 4、SqlSession执行映射文件中定义的SQL，并返回映射结果
		Customer customer = sqlSession.selectOne("top.cairbin.test4"  + ".CustomerMapper. findCustomerById", 1);
		// 打印输出结果，Customer类中记得生成toString()方法。
		System.out.println(customer.toString());
		// 5、关闭SqlSession
		sqlSession.close();
	}
}
```

注意这条语句

```java
Customer customer = sqlSession.selectOne("top.cairbin.test4"  + ".CustomerMapper. findCustomerById", 1);
```

我们正是在调用刚才在`CustomerMapper.xml`里编写的方法，对应`<select>`标签的`id`属性。

点击运行看到输出结果

![](http://img.cairbin.top/img/202403211446583.png)


在`CustomerMapper.xml`映射文件中添加模糊查询,添加,更新和删除的配置,在测试类中添加对应的测试方法,然后进行测试。

**注意每一个方法的**`parameterType`**和**`resultType`**属性对应的包**。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
    "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!-- namespace表示命名空间 -->
<mapper namespace="top.cairbin.test4.CustomerMapper">
    <!--根据客户编号获取客户信息（如果使用的生成类MBGTest.java生成的映射文件，可以不用添加如下内容，直接使用里面的selectByPrimaryKey方法。） -->
	<select id=" findCustomerById" parameterType="Integer"
		resultType="top.cairbin.test4.Customer">
		select * from customer where id = #{id}
	</select>
	
	<!--根据客户名模糊查询客户信息列表-->
	<select id="findCustomerByName" parameterType="String"
	    resultType="top.cairbin.test4.Customer">
	    <!-- select * from customer where username like '%${value}%' -->
	    select * from customer where username like concat('%',#{value},'%')
	</select>
	
	<insert id="addCustomer" parameterType="top.cairbin.test4.Customer">
	    insert into customer(username,jobs,phone)
	    values(#{username},#{jobs},#{phone})
	</insert>
	
	<update id="updateCustomer" parameterType="top.cairbin.test4.Customer">
	    update customer set
	    username=#{username},jobs=#{jobs},phone=#{phone}
	    where id=#{id}
	</update>
	
	<delete id="deleteCustomer" parameterType="Integer">
	    delete from customer where id=#{id}
	</delete>
</mapper>
```

同样的测试类修改为（请注意代码中的包名）
```java
package top.cairbin.test4;

import java.io.InputStream;
import java.util.List;

import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
import org.junit.Test;
import top.cairbin.test4.*;

public class MyBatisTest {
	/**
	 * 根据客户编号查询客户信息
	 */
	@Test
	public void findCustomerByIdTest() throws Exception {
		// 1、读取配置文件
		String resource = "mybatis-config.xml";
		InputStream inputStream = 
                     Resources.getResourceAsStream(resource);
		// 2、根据配置文件构建SqlSessionFactory
		SqlSessionFactory sqlSessionFactory = 
                     new SqlSessionFactoryBuilder().build(inputStream);
		// 3、通过SqlSessionFactory创建SqlSession
		SqlSession sqlSession = sqlSessionFactory.openSession();
		// 4、SqlSession执行映射文件中定义的SQL，并返回映射结果
		Customer customer = sqlSession.selectOne("top.cairbin.test4"  + ".CustomerMapper. findCustomerById", 1);
		// 打印输出结果，Customer类中记得生成toString()方法。
		System.out.println(customer.toString());
		// 5、关闭SqlSession
		sqlSession.close();
	}
	
	/**
	 * 根据用户名称来模糊查询用户信息列表
	 */
	@Test
	public void findCustomerByNameTest() throws Exception{	
	    // 1、读取配置文件
	    String resource = "mybatis-config.xml";
	    InputStream inputStream = Resources.getResourceAsStream(resource);
	    // 2、根据配置文件构建SqlSessionFactory
	    SqlSessionFactory sqlSessionFactory = 
	new SqlSessionFactoryBuilder().build(inputStream);
	    // 3、通过SqlSessionFactory创建SqlSession
	    SqlSession sqlSession = sqlSessionFactory.openSession();
	    // 4、SqlSession执行映射文件中定义的SQL，并返回映射结果
	    List<Customer> customers = sqlSession.selectList("top.cairbin.test4"
					+ ".CustomerMapper.findCustomerByName", "j");
	    for (Customer customer : customers) {
	        //打印输出结果集
	        System.out.println(customer);
	    }
	    // 5、关闭SqlSession
	    sqlSession.close();
	}
	
	/**
	 * 添加客户
	 */
	@Test
	public void addCustomerTest() throws Exception{		
	    // 1、读取配置文件
	    String resource = "mybatis-config.xml";
	    InputStream inputStream = Resources.getResourceAsStream(resource);
	    // 2、根据配置文件构建SqlSessionFactory
	    SqlSessionFactory sqlSessionFactory = 
	    		new SqlSessionFactoryBuilder().build(inputStream);
	    // 3、通过SqlSessionFactory创建SqlSession
	    SqlSession sqlSession = sqlSessionFactory.openSession();
	    // 4、SqlSession执行添加操作
	    // 4.1创建Customer对象，并向对象中添加数据
	    Customer customer = new Customer();
	    customer.setUsername("rose");
	    customer.setJobs("student");
	    customer.setPhone("13333533092");
	    // 4.2执行SqlSession的插入方法，返回的是SQL语句影响的行数
		int rows = sqlSession.insert("top.cairbin.test4"
					+ ".CustomerMapper.addCustomer", customer);
	    // 4.3通过返回结果判断插入操作是否执行成功
	    if(rows > 0){
	        System.out.println("您成功插入了"+rows+"条数据！");
	    }else{
	        System.out.println("执行插入操作失败！！！");
	    }
	    // 4.4提交事务
	    sqlSession.commit();
	    // 5、关闭SqlSession
	    sqlSession.close();
	}

	/**
	 * 更新客户
	 */
	@Test
	public void updateCustomerTest() throws Exception{		
	    // 1、读取配置文件
	    String resource = "mybatis-config.xml";
	    InputStream inputStream = Resources.getResourceAsStream(resource);
	    // 2、根据配置文件构建SqlSessionFactory
	    SqlSessionFactory sqlSessionFactory = 
	    		new SqlSessionFactoryBuilder().build(inputStream);
	    // 3、通过SqlSessionFactory创建SqlSession
	    SqlSession sqlSession = sqlSessionFactory.openSession();
	    // 4、SqlSession执行更新操作
	    // 4.1创建Customer对象，对对象中的数据进行模拟更新
	    Customer customer = new Customer();
	    customer.setId(2); //注意该id必须是你数据库中已有的id.
	    customer.setUsername("rose");
	    customer.setJobs("programmer");
	    customer.setPhone("13311111111");
	    // 4.2执行SqlSession的更新方法，返回的是SQL语句影响的行数
	    int rows = sqlSession.update("top.cairbin.test4"
	            + ".CustomerMapper.updateCustomer", customer);
	    // 4.3通过返回结果判断更新操作是否执行成功
	    if(rows > 0){
	        System.out.println("您成功修改了"+rows+"条数据！");
	    }else{
	        System.out.println("执行修改操作失败！！！");
	    }
	    // 4.4提交事务
	    sqlSession.commit();
	    // 5、关闭SqlSession
	    sqlSession.close();
	}

	/**
	 * 删除客户
	 */
	@Test
	public void deleteCustomerTest() throws Exception{		
	    // 1、读取配置文件
	    String resource = "mybatis-config.xml";
	    InputStream inputStream = Resources.getResourceAsStream(resource);
	    // 2、根据配置文件构建SqlSessionFactory
	    SqlSessionFactory sqlSessionFactory = 
	            new SqlSessionFactoryBuilder().build(inputStream);
	    // 3、通过SqlSessionFactory创建SqlSession
	    SqlSession sqlSession = sqlSessionFactory.openSession();
	    // 4、SqlSession执行删除操作
	    // 4.1执行SqlSession的删除方法，返回的是SQL语句影响的行数
	    int rows = sqlSession.delete("top.cairbin.test4"
	            + ".CustomerMapper.deleteCustomer", 2); //注意该id必须是你数据库中已有的id.
	    // 4.2通过返回结果判断删除操作是否执行成功
	    if(rows > 0){
	        System.out.println("您成功删除了"+rows+"条数据！");
	    }else{
	        System.out.println("执行删除操作失败！！！");
	    }
	    // 4.3提交事务
	    sqlSession.commit();
	    // 5、关闭SqlSession
	    sqlSession.close();
	}
	
}
```

运行，然后测试即可。

![](http://img.cairbin.top/img/202403211456060.png)