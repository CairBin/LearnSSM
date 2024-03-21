# MyBatis关联映射

## 为什么要关联映射

实际开发中，对数据库操作常常会涉及多张表，所以在OOP中就涉及对象与对象的关联关系。针对多表操作，MyBatis提供关联映射。

## 关联关系概述

- 一对一：A类中定义B类的属性b，B类中定义A类属性a。
- 一对多：一个A类类型对应多个B类。
- 多对多：在A类中定义B的集合，B中定义A的集合。

## 嵌套查询与嵌套结果

嵌套查询是通过执行另外一条SQL映射语句来返回预期复杂类型：

- 嵌套查询是在查询SQL嵌套一个子查询
- 嵌套查询会执行多条SQL语句
- 嵌套查询SQL语句编写相对简单


嵌套结果是使用嵌套结果映射来处理的联合结果子集：

- 嵌套结果是一个嵌套的多表查询SQL
- 嵌套结果只会执行一条复杂的SQL语句
- 嵌套结果SQL语句写起来相对麻烦


**对于嵌套查询有一个问题，那就是执行多条SQL语句导致性能开销很大。**于是就有了MyBatis的**延迟加载**——fetchType。

## 实践

### 创建表

我们还是用之前的账户连接数据库，终端命令可以如下

```sql
mysql -u db_mybatis -p
```

输入密码敲击回车，然后切换数据库到`db_mybatis`

Windows用户可以用终端或者`sqlyog`执行下面语句

如果你没有这个数据库请回到前面的章节创建它。

```sql
USE db_mybatis;
```

我们要使用一个新的`customer`表，所以把之前项目留下的删除掉

```sql
DROP TABLE IF EXISTS customer;
```

重新创建它
```sql
CREATE TABLE customer(
  `id` int(32) NOT NULL AUTO_INCREMENT,
  `username` varchar(50) DEFAULT NULL,
  `jobs` varchar(50) DEFAULT NULL,
  `phone` varchar(16) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=3 DEFAULT CHARSET=utf8;
```

给这张表插入两条数据方便我们接下来写代码用

```sql
INSERT  INTO customer(`id`,`username`,`jobs`,`phone`) VALUES (1,'zhangsan','teacher','11111111111'),(2,'lisi','student','11111111112');
```

创建一张`idcard`表，并设置主键（主码）为`id`

如果之前存在请删除

```sql
DROP TABLE IF EXISTS `idcard`;
```

```sql
CREATE TABLE idcard(
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `code` varchar(50) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=3 DEFAULT CHARSET=utf8;
```

同样插入两条数据（身份证号是我瞎编的，为了防止巧合不一定符合真实格式，但长度与现实一样）
```sql
INSERT  INTO idcard(`id`,`code`) VALUES(1,'37010219800X051118'),(2,'370203199X09092222');
```

创建`persion`表，并插入数据

```sql
DROP TABLE IF EXISTS person;

CREATE TABLE person(
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(50) DEFAULT NULL,
  `age` int(11) DEFAULT NULL,
  `card_id` int(11) DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `FK_person` (`card_id`),
  CONSTRAINT `FK_person` FOREIGN KEY (`card_id`) REFERENCES `idcard` (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=3 DEFAULT CHARSET=utf8;

INSERT INTO person(`id`,`name`,`age`,`card_id`) values (1,'张三',18,1),(2,'李四',18,2);
```

创建`product`表并插入数据

```sql
DROP TABLE IF EXISTS product;

CREATE TABLE product(
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(32) DEFAULT NULL,
  `price` double DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=3 DEFAULT CHARSET=utf8;

INSERT INTO product(`id`,`name`,`price`) values (1,'笔记本电脑',8888.8),(2,'华为手机',6666.6);
```

创建`user`表并插入数据

```sql
DROP TABLE IF EXISTS user;

CREATE TABLE user(
  `id` int(11) NOT NULL,
  `name` varchar(20) DEFAULT NULL,
  `password` varchar(20) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

INSERT INTO user(`id`,`name`,`password`) values (1,'zhangsan','654321'),(2,'lisi','123456');
```

创建`orders`表

```sql
DROP TABLE IF EXISTS orders;
CREATE TABLE orders(
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `number` varchar(32) DEFAULT NULL,
  `user_id` int(11) DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `FK_orders` (`user_id`),
  CONSTRAINT `FK_orders` FOREIGN KEY (`user_id`) REFERENCES `user` (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=2 DEFAULT CHARSET=utf8;

INSERT INTO `orders`(`id`,`number`,`user_id`) VALUES (1,'201901',1);
```

创建`orderitem`表，还是一样，如果之前存在先删除。**这个表一定放后面，否则创建外键会报错。**


```sql
DROP TABLE IF EXISTS orderitem;

CREATE TABLE orderitem(
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `orders_id` int(11) DEFAULT NULL,
  `product_id` int(11) DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `FK_orderitem` (`orders_id`),
  KEY `FK_orderitem_product` (`product_id`),
  CONSTRAINT `FK_orderitem_product` FOREIGN KEY (`product_id`) REFERENCES `product` (`id`),
  CONSTRAINT `FK_orderitem` FOREIGN KEY (`orders_id`) REFERENCES `orders` (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=3 DEFAULT CHARSET=utf8;

INSERT INTO orderitem(`id`,`orders_id`,`product_id`) VALUES (1,1,1),(2,1,2);
```

### 构建项目

我们新创建一个Maven项目`top.cairbin.test5`

`pom.xml`添加依赖包

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>top.cairbin</groupId>
  <artifactId>test5</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <packaging>jar</packaging>

  <name>test5</name>
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
	<!-- https://mvnrepository.com/artifact/org.mybatis/mybatis -->
		<dependency>
			<groupId>org.mybatis</groupId>
			<artifactId>mybatis</artifactId>
			<version>3.5.4</version>
		</dependency>
		<!-- https://mvnrepository.com/artifact/mysql/mysql-connector-java -->
		<dependency>
			<groupId>mysql</groupId>
			<artifactId>mysql-connector-java</artifactId>
			<version>8.0.33</version>
		</dependency>
		<!-- https://mvnrepository.com/artifact/org.mybatis.generator/mybatis-generator-core -->
		<dependency>
			<groupId>org.mybatis.generator</groupId>
			<artifactId>mybatis-generator-core</artifactId>
			<version>1.4.0</version>
		</dependency>
    
    	<dependency>
    <groupId>log4j</groupId>
    <artifactId>log4j</artifactId>
    <version>1.2.17</version>
</dependency>
    
  </dependencies>
</project>
```

创建与`src`平级的`resources`目录，并右键->`Build Path`->`Use as Source Folder`

在里面添加一个数据库配置文件`db.properties`

```properties
jdbc.driver=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/db_mybatis
jdbc.username=db_mybatis
jdbc.password=db_mybatis
```

配置MyBatis，在同目录下创建`mybatis-config.xml`，这个文件会使用`db.properties`里面的配置，以`${...}`的方式调用，在`<properties resource="db.properties" />`里声明位置。

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
    "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
	<properties resource="db.properties" />
	<settings>
		<!-- 延迟加载的开关 -->
		<setting name="lazyLoadingEnabled" value="false" />
		<!-- 设置积极加载（true）或按需加载（false） -->
		<setting name="aggressiveLazyLoading" value="true" />
		<setting name="mapUnderscoreToCamelCase" value="true" />
		<setting name="logImpl" value="STDOUT_LOGGING" />
	</settings>
	<typeAliases>
		<package name="top.cairbin.test5" />
	</typeAliases>
	<!--1.配置环境 ，默认的环境id为mysql -->
	<environments default="mysql">
		<!--1.2.配置id为mysql的数据库环境 -->
		<environment id="mysql">
			<!-- 使用JDBC的事务管理 -->
			<transactionManager type="JDBC" />
			<!--数据库连接池 -->
			<dataSource type="POOLED">
				<!-- 数据库驱动 -->
				<property name="driver" value="${jdbc.driver}" />
				<!-- 连接数据库的url -->
				<property name="url" value="${jdbc.url}" />
				<!-- 连接数据库的用户名 -->
				<property name="username" value="${jdbc.username}" />
				<!-- 连接数据库的密码 -->
				<property name="password" value="${jdbc.password}" />
			</dataSource>
		</environment>
	</environments>
	<!--2.配置Mapper的位置 -->
	<mappers>
		<mapper resource="top/cairbin/test5/mapper/CustomerMapper.xml" />
		<mapper resource="top/cairbin/test5/mapper/UserMapper.xml" />
		<mapper resource="top/cairbin/test5/mapper/IdCardMapper.xml" />
		<mapper resource="top/cairbin/test5/mapper/OrdersMapper.xml" />
		<mapper resource="top/cairbin/test5/mapper/PersonMapper.xml" />
		<mapper resource="top/cairbin/test5/mapper/ProductMapper.xml" />
	</mappers>
</configuration>
```

注意上方的`<mapper>`的目录，我们一会介绍。

然后添加log4j的配置文件`log4j.properties`

```properties
# Global logging configuration
log4j.rootLogger=ERROR, stdout
# MyBatis logging configuration...
log4j.logger.top.cairbin.test5=DEBUG
# Console output...
log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=%5p [%t] - %m%n
```

### Dao层

我们先把对应的类创建了再说，到`src/main/java`下`top.cairbin.test5`创建一个子包，名称为`top.cairbin.test5.dao`

![](http://img.cairbin.top/img/202403220056991.png)

![](http://img.cairbin.top/img/202403220056749.png)

在我们新创建的包下面编写若干个类

首先是`Person`类，此时我们的`IdCard`还没创建，IDE提示错误很正常

```java
package top.cairbin.test5.dao;

public class Person {
	private Integer id;
	private String name;
	private Integer age;
	private Integer cardId;
	private IdCard card;

	public void setId(Integer id) {
		this.id = id;
	}
	
	public Integer getId() {
		return this.id;
	}
	
	public void setName(String name) {
		this.name = name;
	}
	
	public String getName() {
		return this.name;
	}
	
	public void setAge(Integer age) {
		this.age = age;
	}
	
	public Integer getAge() {
		return this.age;
	}
	
	public Integer getCardId() {
        return cardId;
    }

    public void setCardId(Integer cardId) {
        this.cardId = cardId;
    }
	
	public void setCard(IdCard card) {
		this.card = card;
	}
	
	public IdCard getCard() {
		return this.card;
	}	
}
```

接下来创建`IdCard`

```java
package top.cairbin.test5.dao;

public class IdCard {
    private Integer id;
    private String code;

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getCode() {
        return code;
    }

    public void setCode(String code) {
        this.code = code == null ? null : code.trim();
    }
}
```

然后是`Customer`类
```java
package top.cairbin.test5.dao;

public class Customer {
    private Integer id;
    private String username;
    private String jobs;
    private String phone;

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username == null ? null : username.trim();
    }

    public String getJobs() {
        return jobs;
    }

    public void setJobs(String jobs) {
        this.jobs = jobs == null ? null : jobs.trim();
    }

    public String getPhone() {
        return phone;
    }

    public void setPhone(String phone) {
        this.phone = phone == null ? null : phone.trim();
    }
}
```

接着是`Orders`

```java
package top.cairbin.test5.dao;

package top.cairbin.test5.dao;

import java.util.List;

public class Orders {
    private Integer id;
    private String number;
    private Integer userId;
    
    private List<Product> productList;

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getNumber() {
        return number;
    }

    public void setNumber(String number) {
        this.number = number == null ? null : number.trim();
    }

    public Integer getUserId() {
        return userId;
    }

    public void setUserId(Integer userId) {
        this.userId = userId;
    }
    
    public void setProductList(List<Product> productList) {
    	this.productList = productList;
    }

    public List<Product> getProductList(){
    	return this.productList;
    }   
}
```

`Product`类

```java
package top.cairbin.test5.dao;

public class Product {
    private Integer id;
    private String name;
    private Double price;

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name == null ? null : name.trim();
    }

    public Double getPrice() {
        return price;
    }

    public void setPrice(Double price) {
        this.price = price;
    }
}
```

`User`类

```java
package top.cairbin.test5.dao;

import java.util.List;

public class User {
    private Integer id;
    private String name;
    private String password;

    private List<Orders> ordersList;
    
    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name == null ? null : name.trim();
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password == null ? null : password.trim();
    }
    
    public void setOrdersList(List<Orders> ordersList) {
    	this.ordersList = ordersList;
    }
    
    public List<Orders> getOrdersList(){
    	return this.ordersList;
    }   
}
```

### Util层

这一层我们来放置一些工具类。创建包`top.cairbin.test5.util`

然后在下面创建`MyBatisHelper`类

```java
package top.cairbin.test5.util;

import java.io.Reader;
import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
public class MyBatisHelper {
	private static SqlSessionFactory sqlSessionFactory = null;
	// 初始化SqlSessionFactory对象
	static {
		try {
			// 使用MyBatis提供的Resources类加载MyBatis的配置文件
			Reader reader = 
					Resources.getResourceAsReader("mybatis-config.xml");
			// 构建SqlSessionFactory工厂
			sqlSessionFactory = 
					new SqlSessionFactoryBuilder().build(reader);
		} catch (Exception e) {
			e.printStackTrace();
		}
	}
	// 获取SqlSession对象的静态方法
	public static SqlSession getSession() {
		return sqlSessionFactory.openSession();
	}
}
```


### Mapper层

接下来我们再为`top.cairbin.test5`创建一个子包`top.cairbin.test5.mapper`，该层主要实现数据持久化。

与上面不一样的是，我们这里定义的是接口

首先是`CustomerMapper`接口

```java
package top.cairbin.test5.mapper;

import top.cairbin.test5.dao.Customer;

public interface CustomerMapper {
    int deleteByPrimaryKey(Integer id);

    int insert(Customer record);

    int insertSelective(Customer record);

    Customer selectByPrimaryKey(Integer id);

    int updateByPrimaryKeySelective(Customer record);

    int updateByPrimaryKey(Customer record);
}
```

到这里你可能会问，谁去实现它们呢，还记得`mybatis-config.xml`里那些`<mapper>`吗，它们每一个都指定`resource`到Mapper层，也就是说我们要在这里写XML文件让MyBatis实现这些方法。

第一个是`CustomerMapper.xml`，注意此处的`namespace`与`<resultMap>`的`id`，下文同理。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="top.cairbin.test5.mapper.CustomerMapper">
  <resultMap id="BaseResultMap" type="top.cairbin.test5.dao.Customer">
    <id column="id" jdbcType="INTEGER" property="id" />
    <result column="username" jdbcType="VARCHAR" property="username" />
    <result column="jobs" jdbcType="VARCHAR" property="jobs" />
    <result column="phone" jdbcType="VARCHAR" property="phone" />
  </resultMap>
  <sql id="Base_Column_List">
    id, username, jobs, phone
  </sql>
  <select id="selectByPrimaryKey" parameterType="java.lang.Integer" resultMap="BaseResultMap">
    select 
    <include refid="Base_Column_List" />
    from customer
    where id = #{id,jdbcType=INTEGER}
  </select>
  <delete id="deleteByPrimaryKey" parameterType="java.lang.Integer">
    delete from customer
    where id = #{id,jdbcType=INTEGER}
  </delete>
  <insert id="insert" parameterType="top.cairbin.test5.dao.Customer">
    insert into customer (id, username, jobs, 
      phone)
    values (#{id,jdbcType=INTEGER}, #{username,jdbcType=VARCHAR}, #{jobs,jdbcType=VARCHAR}, 
      #{phone,jdbcType=VARCHAR})
  </insert>
  <insert id="insertSelective" parameterType="top.cairbin.test5.dao.Customer">
    insert into customer
    <trim prefix="(" suffix=")" suffixOverrides=",">
      <if test="id != null">
        id,
      </if>
      <if test="username != null">
        username,
      </if>
      <if test="jobs != null">
        jobs,
      </if>
      <if test="phone != null">
        phone,
      </if>
    </trim>
    <trim prefix="values (" suffix=")" suffixOverrides=",">
      <if test="id != null">
        #{id,jdbcType=INTEGER},
      </if>
      <if test="username != null">
        #{username,jdbcType=VARCHAR},
      </if>
      <if test="jobs != null">
        #{jobs,jdbcType=VARCHAR},
      </if>
      <if test="phone != null">
        #{phone,jdbcType=VARCHAR},
      </if>
    </trim>
  </insert>
  <update id="updateByPrimaryKeySelective" parameterType="top.cairbin.test5.dao.Customer">
    update customer
    <set>
      <if test="username != null">
        username = #{username,jdbcType=VARCHAR},
      </if>
      <if test="jobs != null">
        jobs = #{jobs,jdbcType=VARCHAR},
      </if>
      <if test="phone != null">
        phone = #{phone,jdbcType=VARCHAR},
      </if>
    </set>
    where id = #{id,jdbcType=INTEGER}
  </update>
  <update id="updateByPrimaryKey" parameterType="top.cairbin.test5.dao.Customer">
    update customer
    set username = #{username,jdbcType=VARCHAR},
      jobs = #{jobs,jdbcType=VARCHAR},
      phone = #{phone,jdbcType=VARCHAR}
    where id = #{id,jdbcType=INTEGER}
  </update>
</mapper>
```

**对了，请检查你XML中的方法的`parameterType`属性以及`resultType`对应类型的包是否正确！！！**

然后是`IdCardMapper`接口

```java
package top.cairbin.test5.mapper;

import top.cairbin.test5.dao.IdCard;

public interface IdCardMapper {
    int deleteByPrimaryKey(Integer id);

    int insert(IdCard record);

    int insertSelective(IdCard record);

    IdCard selectByPrimaryKey(Integer id);

    int updateByPrimaryKeySelective(IdCard record);

    int updateByPrimaryKey(IdCard record);
}
```

同样在`IdCardMapper.xml`里实现这些方法

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="top.cairbin.test5.mapper.IdCardMapper">
  <resultMap id="BaseResultMap" type="top.cairbin.test5.dao.IdCard">
    <id column="id" jdbcType="INTEGER" property="id" />
    <result column="code" jdbcType="VARCHAR" property="code" />
  </resultMap>
  <sql id="Base_Column_List">
    id, code
  </sql>
  <select id="selectByPrimaryKey" parameterType="java.lang.Integer" resultMap="BaseResultMap">
    select 
    <include refid="Base_Column_List" />
    from idcard
    where id = #{id,jdbcType=INTEGER}
  </select>
  <delete id="deleteByPrimaryKey" parameterType="java.lang.Integer">
    delete from idcard
    where id = #{id,jdbcType=INTEGER}
  </delete>
  <insert id="insert" parameterType="top.cairbin.test5.dao.IdCard">
    insert into idcard (id, code)
    values (#{id,jdbcType=INTEGER}, #{code,jdbcType=VARCHAR})
  </insert>
  <insert id="insertSelective" parameterType="top.cairbin.test5.dao.IdCard">
    insert into idcard
    <trim prefix="(" suffix=")" suffixOverrides=",">
      <if test="id != null">
        id,
      </if>
      <if test="code != null">
        code,
      </if>
    </trim>
    <trim prefix="values (" suffix=")" suffixOverrides=",">
      <if test="id != null">
        #{id,jdbcType=INTEGER},
      </if>
      <if test="code != null">
        #{code,jdbcType=VARCHAR},
      </if>
    </trim>
  </insert>
  <update id="updateByPrimaryKeySelective" parameterType="top.cairbin.test5.dao.IdCard">
    update idcard
    <set>
      <if test="code != null">
        code = #{code,jdbcType=VARCHAR},
      </if>
    </set>
    where id = #{id,jdbcType=INTEGER}
  </update>
  <update id="updateByPrimaryKey" parameterType="top.cairbin.test5.dao.IdCard">
    update idcard
    set code = #{code,jdbcType=VARCHAR}
    where id = #{id,jdbcType=INTEGER}
  </update>
</mapper>
```

然后是`ProductMapper`接口以及`ProductMapper.xml`

```java
package top.cairbin.test5.mapper;
import top.cairbin.test5.dao.Product;

public interface ProductMapper {
    int deleteByPrimaryKey(Integer id);

    int insert(Product record);

    int insertSelective(Product record);

    Product selectByPrimaryKey(Integer id);

    int updateByPrimaryKeySelective(Product record);

    int updateByPrimaryKey(Product record);
}
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="top.cairbin.test5.mapper.ProductMapper">
  <resultMap id="BaseResultMap" type="top.cairbin.test5.dao.Product">
    <id column="id" jdbcType="INTEGER" property="id" />
    <result column="name" jdbcType="VARCHAR" property="name" />
    <result column="price" jdbcType="DOUBLE" property="price" />
  </resultMap>
  <sql id="Base_Column_List">
    id, name, price
  </sql>
  
  <select id="findProductByOrderId" parameterType="Integer"   resultType="Product">
		SELECT * from product where id IN(
		   SELECT product_id FROM orderitem  WHERE orders_id = #{id}
		)
	</select>

  
  <select id="selectByPrimaryKey" parameterType="java.lang.Integer" resultMap="BaseResultMap">
    select 
    <include refid="Base_Column_List" />
    from product
    where id = #{id,jdbcType=INTEGER}
  </select>
  <delete id="deleteByPrimaryKey" parameterType="java.lang.Integer">
    delete from product
    where id = #{id,jdbcType=INTEGER}
  </delete>
  <insert id="insert" parameterType="top.cairbin.test5.dao.Product">
    insert into product (id, name, price
      )
    values (#{id,jdbcType=INTEGER}, #{name,jdbcType=VARCHAR}, #{price,jdbcType=DOUBLE}
      )
  </insert>
  <insert id="insertSelective" parameterType="top.cairbin.test5.dao.Product">
    insert into product
    <trim prefix="(" suffix=")" suffixOverrides=",">
      <if test="id != null">
        id,
      </if>
      <if test="name != null">
        name,
      </if>
      <if test="price != null">
        price,
      </if>
    </trim>
    <trim prefix="values (" suffix=")" suffixOverrides=",">
      <if test="id != null">
        #{id,jdbcType=INTEGER},
      </if>
      <if test="name != null">
        #{name,jdbcType=VARCHAR},
      </if>
      <if test="price != null">
        #{price,jdbcType=DOUBLE},
      </if>
    </trim>
  </insert>
  <update id="updateByPrimaryKeySelective" parameterType="top.cairbin.test5.dao.Product">
    update product
    <set>
      <if test="name != null">
        name = #{name,jdbcType=VARCHAR},
      </if>
      <if test="price != null">
        price = #{price,jdbcType=DOUBLE},
      </if>
    </set>
    where id = #{id,jdbcType=INTEGER}
  </update>
  <update id="updateByPrimaryKey" parameterType="top.cairbin.test5.dao.Product">
    update product
    set name = #{name,jdbcType=VARCHAR},
      price = #{price,jdbcType=DOUBLE}
    where id = #{id,jdbcType=INTEGER}
  </update>
</mapper>
```

**对于剩下来的几个Dao层类对应的Mapper需要特别注意下，因为它们的类里都包含复合类型，这也是我们这一章节真正要学习的东西——关联映射。**

`PersonMapper`接口

```java
package top.cairbin.test5.mapper;

import top.cairbin.test5.dao.Person;

public interface PersonMapper {
    int deleteByPrimaryKey(Integer id);

    int insert(Person record);

    int insertSelective(Person record);

    Person selectByPrimaryKey(Integer id);

    int updateByPrimaryKeySelective(Person record);

    int updateByPrimaryKey(Person record);
}
```

`PersonMapper.xml`文件，请重点关注`findPersonById`和`findPersonById2`这两个地方。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="top.cairbin.test5.mapper.PersonMapper">

	<resultMap id="BaseResultMap" type="top.cairbin.test5.dao.Person">
    	<id column="id" jdbcType="INTEGER" property="id" />
    	<result column="name" jdbcType="VARCHAR" property="name" />
    	<result column="age" jdbcType="INTEGER" property="age" />
    	<result column="card_id" jdbcType="INTEGER" property="cardId" />
  </resultMap>
  <sql id="Base_Column_List">
    id, name, age, card_id
  </sql>
  
  <!-- 嵌套查询：通过执行另外一条SQL映射语句来返回预期的特殊类型 -->
	<select id="findPersonById" parameterType="Integer" 
    resultMap="IdCardWithPersonResult">
		SELECT * from person where id=#{id}
	</select>
	<resultMap type="Person" id="IdCardWithPersonResult">
		<id property="id" column="id" />
		<result property="name" column="name" />
		<result property="age" column="age" />
		<!-- 一对一：association使用select属性引入另外一条SQL语句 -->
		<association property="card" column="card_id" javaType="top.cairbin.test5.dao.IdCard"
			select="top.cairbin.test5.mapper.IdCardMapper.selectByPrimaryKey" />
	</resultMap>

  
  	<!-- 嵌套结果：使用嵌套结果映射来处理重复的联合结果的子集 -->
  	<select id="findPersonById2" parameterType="Integer" 
	                                   resultMap="IdCardWithPersonResult2">
	    SELECT p.*,idcard.code
	    from person p,idcard idcard
	    where p.card_id=idcard.id 
	    and p.id= #{id}
	</select>
	<resultMap type="Person" id="IdCardWithPersonResult2">
	    <id property="id" column="id" />
	    <result property="name" column="name" />
	    <result property="age" column="age" />
	    <association property="card" javaType="IdCard">
	        <id property="id" column="id" />
	        <result property="code" column="code" />
	    </association>
	</resultMap>

  
  
  
  <select id="selectByPrimaryKey" parameterType="java.lang.Integer" resultMap="BaseResultMap">
    select 
    <include refid="Base_Column_List" />
    from person
    where id = #{id,jdbcType=INTEGER}
  </select>
  <delete id="deleteByPrimaryKey" parameterType="java.lang.Integer">
    delete from person
    where id = #{id,jdbcType=INTEGER}
  </delete>
  <insert id="insert" parameterType="top.cairbin.test5.dao.Person">
    insert into person (id, name, age, 
      card_id)
    values (#{id,jdbcType=INTEGER}, #{name,jdbcType=VARCHAR}, #{age,jdbcType=INTEGER}, 
      #{cardId,jdbcType=INTEGER})
  </insert>
  <insert id="insertSelective" parameterType="top.cairbin.test5.dao.Person">
    insert into person
    <trim prefix="(" suffix=")" suffixOverrides=",">
      <if test="id != null">
        id,
      </if>
      <if test="name != null">
        name,
      </if>
      <if test="age != null">
        age,
      </if>
      <if test="cardId != null">
        card_id,
      </if>
    </trim>
    <trim prefix="values (" suffix=")" suffixOverrides=",">
      <if test="id != null">
        #{id,jdbcType=INTEGER},
      </if>
      <if test="name != null">
        #{name,jdbcType=VARCHAR},
      </if>
      <if test="age != null">
        #{age,jdbcType=INTEGER},
      </if>
      <if test="cardId != null">
        #{cardId,jdbcType=INTEGER},
      </if>
    </trim>
  </insert>
  <update id="updateByPrimaryKeySelective" parameterType="top.cairbin.test5.dao.Person">
    update person
    <set>
      <if test="name != null">
        name = #{name,jdbcType=VARCHAR},
      </if>
      <if test="age != null">
        age = #{age,jdbcType=INTEGER},
      </if>
      <if test="cardId != null">
        card_id = #{cardId,jdbcType=INTEGER},
      </if>
    </set>
    where id = #{id,jdbcType=INTEGER}
  </update>
  <update id="updateByPrimaryKey" parameterType="top.cairbin.test5.dao.Person">
    update person
    set name = #{name,jdbcType=VARCHAR},
      age = #{age,jdbcType=INTEGER},
      card_id = #{cardId,jdbcType=INTEGER}
    where id = #{id,jdbcType=INTEGER}
  </update>
</mapper>
```

对于上面这么长的文件，最重要的地方是这里
```xml
<!-- 一对一：association使用select属性引入另外一条SQL语句 -->
<association property="card" column="card_id"   javaType="top.cairbin.test5.dao.IdCard"
select="top.cairbin.test5.mapper.IdCardMapper.selectByPrimaryKey" />
```

为了看到效果，我们在`mybatis-config.xml`里将**延迟加载**关掉。

```xml
<settings>
	<!-- 延迟加载的开关 -->
	<!-- 为了让大家看到关联关系的效果，我们在这里关闭了延迟加载 -->  
	<setting name="lazyLoadingEnabled" value="false" />  
	<setting name="aggressiveLazyLoading" value="true"/>  
</settings>
```

![](http://img.cairbin.top/img/202403220218139.png)

创建`UserMapper`接口和`UserMapper.xml`

```java
package top.cairbin.test5.mapper;

import top.cairbin.test5.dao.User;

public interface UserMapper {
    int deleteByPrimaryKey(Integer id);

    int insert(User record);

    int insertSelective(User record);

    User selectByPrimaryKey(Integer id);

    int updateByPrimaryKeySelective(User record);

    int updateByPrimaryKey(User record);
}
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="top.cairbin.test5.mapper.UserMapper">
  <resultMap id="BaseResultMap" type="top.cairbin.test5.dao.User">
    <id column="id" jdbcType="INTEGER" property="id" />
    <result column="name" jdbcType="VARCHAR" property="name" />
    <result column="password" jdbcType="VARCHAR" property="password" />
    
    <collection property="ordersList" column="id" ofType="Orders" 
		     select="top.cairbin.test5.mapper.OrdersMapper.findOrdersWithUser">
	</collection>

    
  </resultMap>
  <sql id="Base_Column_List">
    id, name, password
  </sql>
  <select id="selectByPrimaryKey" parameterType="java.lang.Integer" resultMap="BaseResultMap">
    select 
    <include refid="Base_Column_List" />
    from user
    where id = #{id,jdbcType=INTEGER}
  </select>
  <delete id="deleteByPrimaryKey" parameterType="java.lang.Integer">
    delete from user
    where id = #{id,jdbcType=INTEGER}
  </delete>
  <insert id="insert" parameterType="top.cairbin.test5.dao.User">
    insert into user (id, name, password
      )
    values (#{id,jdbcType=INTEGER}, #{name,jdbcType=VARCHAR}, #{password,jdbcType=VARCHAR}
      )
  </insert>
  <insert id="insertSelective" parameterType="top.cairbin.test5.dao.User">
    insert into user
    <trim prefix="(" suffix=")" suffixOverrides=",">
      <if test="id != null">
        id,
      </if>
      <if test="name != null">
        name,
      </if>
      <if test="password != null">
        password,
      </if>
    </trim>
    <trim prefix="values (" suffix=")" suffixOverrides=",">
      <if test="id != null">
        #{id,jdbcType=INTEGER},
      </if>
      <if test="name != null">
        #{name,jdbcType=VARCHAR},
      </if>
      <if test="password != null">
        #{password,jdbcType=VARCHAR},
      </if>
    </trim>
  </insert>
  <update id="updateByPrimaryKeySelective" parameterType="top.cairbin.test5.dao.User">
    update user
    <set>
      <if test="name != null">
        name = #{name,jdbcType=VARCHAR},
      </if>
      <if test="password != null">
        password = #{password,jdbcType=VARCHAR},
      </if>
    </set>
    where id = #{id,jdbcType=INTEGER}
  </update>
  <update id="updateByPrimaryKey" parameterType="top.cairbin.test5.dao.User">
    update user
    set name = #{name,jdbcType=VARCHAR},
      password = #{password,jdbcType=VARCHAR}
    where id = #{id,jdbcType=INTEGER}
  </update>
</mapper>
```


最后`OrdersMapper`和`OrdersMapper.xml`

```java
package top.cairbin.test5.mapper;

import top.cairbin.test5.dao.Orders;

public interface OrdersMapper {
    int deleteByPrimaryKey(Integer id);

    int insert(Orders record);

    int insertSelective(Orders record);

    Orders selectByPrimaryKey(Integer id);

    int updateByPrimaryKeySelective(Orders record);

    int updateByPrimaryKey(Orders record);
}
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="top.cairbin.test5.mapper.OrdersMapper">
  <resultMap id="BaseResultMap" type="top.cairbin.test5.dao.Orders">
    <id column="id" jdbcType="INTEGER" property="id" />
    <result column="number" jdbcType="VARCHAR" property="number" />
    <result column="user_id" jdbcType="INTEGER" property="userId" />
  </resultMap>
  <sql id="Base_Column_List">
    id, number, user_id
  </sql>
  
  <select id="findOrdersWithUser" parameterType="Integer" 
              resultMap="BaseResultMap">
		select * from orders WHERE user_id=#{id}	
	</select>

  
  
  <select id="selectByPrimaryKey" parameterType="java.lang.Integer" resultMap="BaseResultMap">
    select 
    <include refid="Base_Column_List" />
    from orders
    where id = #{id,jdbcType=INTEGER}
  </select>
  <delete id="deleteByPrimaryKey" parameterType="java.lang.Integer">
    delete from orders
    where id = #{id,jdbcType=INTEGER}
  </delete>
  <insert id="insert" parameterType="top.cairbin.test5.dao.Orders">
    insert into orders (id, number, user_id
      )
    values (#{id,jdbcType=INTEGER}, #{number,jdbcType=VARCHAR}, #{userId,jdbcType=INTEGER}
      )
  </insert>
  <insert id="insertSelective" parameterType="top.cairbin.test5.dao.Orders">
    insert into orders
    <trim prefix="(" suffix=")" suffixOverrides=",">
      <if test="id != null">
        id,
      </if>
      <if test="number != null">
        number,
      </if>
      <if test="userId != null">
        user_id,
      </if>
    </trim>
    <trim prefix="values (" suffix=")" suffixOverrides=",">
      <if test="id != null">
        #{id,jdbcType=INTEGER},
      </if>
      <if test="number != null">
        #{number,jdbcType=VARCHAR},
      </if>
      <if test="userId != null">
        #{userId,jdbcType=INTEGER},
      </if>
    </trim>
  </insert>
  <update id="updateByPrimaryKeySelective" parameterType="top.cairbin.test5.dao.Orders">
    update orders
    <set>
      <if test="number != null">
        number = #{number,jdbcType=VARCHAR},
      </if>
      <if test="userId != null">
        user_id = #{userId,jdbcType=INTEGER},
      </if>
    </set>
    where id = #{id,jdbcType=INTEGER}
  </update>
  <update id="updateByPrimaryKey" parameterType="top.cairbin.test5.dao.Orders">
    update orders
    set number = #{number,jdbcType=VARCHAR},
      user_id = #{userId,jdbcType=INTEGER}
    where id = #{id,jdbcType=INTEGER}
  </update>
</mapper>
```

我们编写测试类，在`src/test/java`下的`top.cairbin.test5`里


`PersonTest`类来测试一对一的关系
```java
package top.cairbin.test5;

import org.apache.ibatis.session.SqlSession;
import org.junit.Test;
import top.cairbin.test5.dao.Person;
import top.cairbin.test5.util.*;

public class PersonTest {
	@Test
    public void findPersonByIdTest() {
        // 1、通过工具类生成SqlSession对象
        SqlSession session = MyBatisHelper.getSession();
        // 2.使用MyBatis嵌套查询的方式查询id为1的人的信息
        Person person = session.selectOne("top.cairbin.test5.mapper." 
                                   + "PersonMapper.selectByPrimaryKey", 1);
        // 3、输出查询结果信息
        System.out.println(person);
        // 4、关闭SqlSession
        session.close();
    }

	//测试一对一（嵌套结果）：
	@Test
    public void findPersonByIdTest2() {
        // 1、通过工具类生成SqlSession对象
        SqlSession session = MyBatisHelper.getSession();
        // 2.使用MyBatis嵌套查询的方式查询id为1的人的信息
        Person person = session.selectOne("top.cairbin.test5.mapper." 
                                   + "PersonMapper.findPersonById2", 1);
        // 3、输出查询结果信息
        System.out.println(person);
        // 4、关闭SqlSession
        session.close();
    }
}
```
点击运行，两个方法所用时间差距不小。
![](http://img.cairbin.top/img/202403220338937.png)

我们再编写测试类`UserTest`来测试一对多

```java
package top.cairbin.test5;

import org.apache.ibatis.session.SqlSession;
import org.junit.Test;
import top.cairbin.test5.dao.User;
import top.cairbin.test5.util.MyBatisHelper;

public class UserTest {
	@Test
    public void findUserTest() {
        // 1、通过工具类生成SqlSession对象
        SqlSession session = MyBatisHelper.getSession();
        // 2、查询id为1的用户信息，注意包名为mapper层的
        User user = session.selectOne("top.cairbin.test5.mapper."
                                + "UserMapper.selectByPrimaryKey", 1);
        // 3、输出查询结果信息
        System.out.println(user);
        // 4、关闭SqlSession
        session.close();
    }

}
```

![](http://img.cairbin.top/img/202403220339629.png)

最后编写`OrdersTest`来测试多对多

```java
package top.cairbin.test5;

import org.apache.ibatis.session.SqlSession;
import org.junit.Test;

import top.cairbin.test5.dao.Orders;
import top.cairbin.test5.util.MyBatisHelper;

public class OrdersTest {
	@Test
    public void findOrdersTest(){
        // 1、通过工具类生成SqlSession对象
        SqlSession session = MyBatisHelper.getSession();
        // 2、查询id为1的订单中的商品信息
        Orders orders = session.selectOne("top.cairbin.test5.mapper."
                               + "OrdersMapper.selectByPrimaryKey", 1);
        // 3、输出查询结果信息
        System.out.println(orders);
        // 4、关闭SqlSession
        session.close();
    }
}
```

![](http://img.cairbin.top/img/202403220342141.png)

## 总结

在本章节，我们学习了MyBatis的关联映射。此外，你会发现我们写代码不再像以前一样只放在一个包里，而是创建好几个子包分层放置不同的代码，这有利于代码逻辑清晰并且方便维护，这种分层设计已经有一个正式项目的雏形了，当你写完这些东西的时候项目目录应该与我差不多。

![](http://img.cairbin.top/img/202403220342671.png)