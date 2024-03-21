# 动态SQL

## 什么是动态SQL

动态SQL是MyBatis强大特性之一，MyBatis3采用了功能强大的基于OGNL的表达式来完成SQL。

## 动态SQL主要元素

常用的动态SQL主要元素如下：

![动态SQL主要元素](http://img.cairbin.top/img/202403211730019.png)

如果单独对以上元素解释理解起来还是比较费力的，接下来还是以代码的形式展现。

## 实践

### 基础部分

我们还是用之前的项目`top.cairbin.test4`，我们把`src/main/java`目录下的`top.cairbin.test4`包中的`CustomerMapper.xml`文件里的`<mapper namespace="top.cairbin.test4.CustomerMapper"></mapper>`中间的内容全部删除，最终效果看上去应该与下面一致：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
    "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!-- namespace表示命名空间 -->
<mapper namespace="top.cairbin.test4.CustomerMapper">

</mapper>
```

然后把`src/test/java`目录下的`top.cairbin.test4`中的测试类`MyBatisTest`里面的测试方法也删除掉

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

}
```

我们先来编写表与对象的映射关系，在`CustomerMapper.xml`的`<mapper>`元素之间填写如下内容（注意包名与你的一致）：

```xml
<!--对应的数据集-->
<resultMap id="BaseResultMap"
	type="top.cairbin.test4.Customer">
	<id column="id" jdbcType="INTEGER" property="id" />
	<result column="username" jdbcType="VARCHAR"
		property="username" />
	<result column="jobs" jdbcType="VARCHAR" property="jobs" />
	<result column="phone" jdbcType="VARCHAR" property="phone" />
</resultMap>
<sql id="Base_Column_List">
	id, username, jobs, phone
</sql>
```

看起来效果应该跟下面一样，不过在后文中这个文件会越写越长，由于篇幅有限，我们每次仅给出增添部分的代码，而不是每次修改都粘贴全部配置到这里来

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
    "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!-- namespace表示命名空间 -->
<mapper namespace="top.cairbin.test4.CustomerMapper">
	
	<!--对应的数据集-->
	<resultMap id="BaseResultMap"
		type="top.cairbin.test4.Customer">
		<id column="id" jdbcType="INTEGER" property="id" />
		<result column="username" jdbcType="VARCHAR"
			property="username" />
		<result column="jobs" jdbcType="VARCHAR" property="jobs" />
		<result column="phone" jdbcType="VARCHAR" property="phone" />
	</resultMap>
	<sql id="Base_Column_List">
		id, username, jobs, phone
	</sql>

</mapper>
```

为了方便操作`SqlSession`，我们在`src/main/java`的`top.cairbin.test4`包下写一个工具类，名称为`MyBatisHelper`

```java
package top.cairbin.test4;

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
			Reader reader = Resources.getResourceAsReader("mybatis-config.xml");
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

你可能对`static{...}`这段代码有所疑惑，但是你在学习Java语言的时候应该涉及过才对，这也算一种技巧吧，即然碰到了我们就讲一下。

还记不记得`static`关键字的特性？被它修饰的代码不依赖于任何实例化的对象，它们的内存是固定且唯一的，并且不会被Java的垃圾回收机制回收，直至程序结束。也就是说当它们的内存一旦被确认下来，在程序需运行的过程中，所指向的内存位置都是同一地址。

当你定义一个类`MyClass`，类内有个`public`静态成员`staticTest`，你并不用`(new MyClass()).staticTest`，而是直接以`MyClass.staticTest`的方式去调用。因为对于每一个`MyClass`它所指向的`staticTest`都是同一个。

那么跟我们要说的`static`修饰的代码块有什么关系呢？想一想你每次调用方法，都要连接一次数据库，操作完后被Java的回收机制释放掉，当调用次数过多的时候对于连接方和被连接方开销都是巨大的。那么这时候你就会想，能不能让这个过程只执行一次而且不会被Java的回收机制释放呢？这时候就可以利用我们的**静态代码块**了。无论你`new MyBatisHelper()`多少次，它只会加载一次且在整个程序运行时有效！

### 编写方法

回到正题，我们来编写第一个方法，插入Customer

我们在`CustomerMapper.xml`中`<mapper></mapper>`标签中间，在原来的基础上**追加**以下内容（这句话描述的很准确了）

```xml
<insert id="insertSelective"
parameterType="top.cairbin.test4.Customer">
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
```

然后在`MyBatisTest`测试类里添加测试方法（这里就用到了`MyBatisHelper`）：

```java
@Test
public void insertCustomerTest() {
	SqlSession session = MyBatisHelper.getSession();
	Customer cc = new Customer();
	cc.setUsername("cairbin");
	cc.setJobs("student");
	cc.setPhone("888888");
	int insert = session.insert("top.cairbin.test4.CustomerMapper.insertSelective", cc);
	session.commit();
	System.out.println(insert + "," + cc);
	session.close();
}
```

我们运行一下，看看结果

![](http://img.cairbin.top/img/202403212126943.png)

看样子是成功了。

来看看数据库里，操作还是那样，我这里仅给个结果，你可以跳过这一步，因为我们接下来要编写查询方法。

![](http://img.cairbin.top/img/202403212127006.png)

继续编写`CustomerMapper.xml`文件，我们这里用到了`<select>`标签，并且同样用`<if>`来对数据进行判断。这里应当提一句，看一下`<select>`的`resultMap`属性，是不是正好是`<resultMap>`标签的`id`，对应的Java类也就是`Customer`，同样对应数据库的表`customer`。

```xml
<select id="findCustomerByNameAndJobs"
parameterType="top.cairbin.test4.Customer" resultMap="BaseResultMap">
select * from customer where 1=1
<if test="username !=null and username !=''">
	and username like concat('%',#{username}, '%')
</if>
<if test="jobs !=null and jobs !=''">
	and jobs= #{jobs}
</if>
</select>
```

继续编写测试方法，结合XML，我们可知这个是测试根据`username`和`jobs`来查询数据的。

```java
@Test
public void findCustomerByNameAndJobsTest() {
	// 通过工具类生成SqlSession对象
	SqlSession session = MyBatisHelper.getSession();
	// 创建Customer对象，封装需要组合查询的条件
	Customer customer = new Customer();
	customer.setUsername("cairbin");
	customer.setJobs("student");
	// 执行SqlSession的查询方法，返回结果集
	List<Customer> customers = session
			.selectList("top.cairbin.test4" + ".CustomerMapper.findCustomerByNameAndJobs", customer);
	// 输出查询结果信息
	for (Customer customer2 : customers) {
		// 打印输出结果
		System.out.println(customer2);
	}
	// 关闭SqlSession
	session.close();
}
```

大致原理已经知道了，剩下的我们就不做演示了，自己尝试去写一写吧。

这里给出完全的`CustomerMapper.xml`和`MyBatis.java`

- `CustomerMapper.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
    "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!-- namespace表示命名空间 -->
<mapper namespace="top.cairbin.test4.CustomerMapper">
	
	<!--对应的数据集-->
	<resultMap id="BaseResultMap"
		type="top.cairbin.test4.Customer">
		<id column="id" jdbcType="INTEGER" property="id" />
		<result column="username" jdbcType="VARCHAR"
			property="username" />
		<result column="jobs" jdbcType="VARCHAR" property="jobs" />
		<result column="phone" jdbcType="VARCHAR" property="phone" />
	</resultMap>
	<sql id="Base_Column_List">
		id, username, jobs, phone
	</sql>
	
	<select id="selectByPrimaryKey"
		parameterType="java.lang.Integer" resultMap="BaseResultMap">
		select
		<include refid="Base_Column_List" />
		from customer
		where id = #{id,jdbcType=INTEGER}
	</select>
	<delete id="deleteByPrimaryKey"
		parameterType="java.lang.Integer">
		delete from customer
		where id = #{id,jdbcType=INTEGER}
	</delete>
	<insert id="insert" parameterType="top.cairbin.test4.Customer"
		useGeneratedKeys="true" keyProperty="id">
		insert into customer (id, username, jobs,
		phone)
		values (#{id,jdbcType=INTEGER}, #{username,jdbcType=VARCHAR},
		#{jobs,jdbcType=VARCHAR},
		#{phone,jdbcType=VARCHAR})
	</insert>
	<insert id="insertSelective"
		parameterType="top.cairbin.test4.Customer">
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
	<update id="updateByPrimaryKeySelective"
		parameterType="top.cairbin.test4.Customer">
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
	<update id="updateByPrimaryKey"
		parameterType="top.cairbin.test4.Customer">
		update customer
		set username = #{username,jdbcType=VARCHAR},
		jobs = #{jobs,jdbcType=VARCHAR},
		phone = #{phone,jdbcType=VARCHAR}
		where id = #{id,jdbcType=INTEGER}
	</update>
	<select id="findAllCustomer"
		resultType="top.cairbin.test4.Customer">
		select * from customer where 1=1
	</select>
	<select id="findCustomerByNameAndJobs"
		parameterType="top.cairbin.test4.Customer" resultMap="BaseResultMap">
		select * from customer where 1=1
		<if test="username !=null and username !=''">
			and username like concat('%',#{username}, '%')
		</if>
		<if test="jobs !=null and jobs !=''">
			and jobs= #{jobs}
		</if>
	</select>
	<select id="findCustomerByNameOrJobs"
		parameterType="top.cairbin.test4.Customer" resultMap="BaseResultMap">
		select * from customer where 1=1
		<choose>
			<when test="username !=null and username !=''">
				and username like concat('%',#{username}, '%')
			</when>
			<when test="jobs !=null and jobs !=''">
				and jobs= #{jobs}
			</when>
			<otherwise>
				and phone is not null
			</otherwise>
		</choose>
	</select>
	<select id="findCustomerByIds" parameterType="List"
		resultMap="BaseResultMap">
		select * from customer where id in
		<foreach item="id" index="index" collection="list" open="("
			separator="," close=")">
			#{id}
		</foreach>
	</select>
	<select id="findCustomerByName" parameterType="Integer"
		resultMap="BaseResultMap">
		<bind name="pattern_username"
			value="'%'+_parameter.getUsername()+'%'" />
		select * from customer
		where username like #{pattern_username}
	</select>

</mapper>
```

- `MyBatisTest.java`

```java
package top.cairbin.test4;

import java.io.InputStream;
import java.util.ArrayList;
import java.util.List;

import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
import org.junit.Test;
import top.cairbin.test4.*;

public class MyBatisTest {
	@Test
	public void insertCustomerTest() {

		SqlSession session = MyBatisHelper.getSession();
		Customer cc = new Customer();
		cc.setUsername("cairbin");
		cc.setJobs("student");
		cc.setPhone("888888");
		int insert = session.insert("top.cairbin.test4.CustomerMapper.insertSelective", cc);
		session.commit();
		System.out.println(insert + "," + cc);
		session.close();
	}

	@Test
	public void findCustomerByNameAndJobsTest() {
		// 通过工具类生成SqlSession对象
		SqlSession session = MyBatisHelper.getSession();
		// 创建Customer对象，封装需要组合查询的条件
		Customer customer = new Customer();
		customer.setUsername("cairbin");
		customer.setJobs("student");
		// 执行SqlSession的查询方法，返回结果集
		List<Customer> customers = session
				.selectList("top.cairbin.test4" + ".CustomerMapper.findCustomerByNameAndJobs", customer);
		// 输出查询结果信息
		for (Customer customer2 : customers) {
			// 打印输出结果
			System.out.println(customer2);
		}
		// 关闭SqlSession
		session.close();
	}

	/**
	 * 根据客户姓名或职业查询客户信息列表
	 */
	@Test
	public void findCustomerByNameOrJobsTest() {
		// 通过工具类生成SqlSession对象
		SqlSession session = MyBatisHelper.getSession();
		// 创建Customer对象，封装需要组合查询的条件
		Customer customer = new Customer();
		customer.setUsername("cairbin");
//	    customer.setJobs("teacher");
		// 执行SqlSession的查询方法，返回结果集
		List<Customer> customers = session.selectList("top.cairbin.test4" + ".CustomerMapper.findCustomerByNameOrJobs",
				customer);
		// 输出查询结果信息
		for (Customer customer2 : customers) {
			// 打印输出结果
			System.out.println(customer2);
		}
		// 关闭SqlSession
		session.close();
	}

	/**
	 * 更新客户
	 */
	@Test
	public void updateCustomerTest() {
		// 获取SqlSession
		SqlSession sqlSession = MyBatisHelper.getSession();
		// 创建Customer对象，并向对象中添加数据
		Customer customer = new Customer();
		customer.setId(1);
		customer.setPhone("13311111234");
		// 执行SqlSession的更新方法，返回的是SQL语句影响的行数
		int rows = sqlSession.update("top.cairbin.test4" + ".CustomerMapper.updateByPrimaryKeySelective", customer);
		// 通过返回结果判断更新操作是否执行成功
		if (rows > 0) {
			System.out.println("您成功修改了" + rows + "条数据！");
		} else {
			System.out.println("执行修改操作失败！！！");
		}
		// 提交事务
		sqlSession.commit();
		// 关闭SqlSession
		sqlSession.close();
	}

	/**
	 * 根据客户编号批量查询客户信息
	 */
	@Test
	public void findCustomerByIdsTest() {
		// 获取SqlSession
		SqlSession session = MyBatisHelper.getSession();
		// 创建List集合，封装查询id
		List<Integer> ids = new ArrayList<Integer>();
		ids.add(1);
		ids.add(2);
		// 执行SqlSession的查询方法，返回结果集
		List<Customer> customers = session.selectList("top.cairbin.test4" + ".CustomerMapper.findCustomerByIds", ids);
		// 输出查询结果信息
		for (Customer customer : customers) {
			// 打印输出结果
			System.out.println(customer);
		}
		// 关闭SqlSession
		session.close();
	}

	/**
	 * bind元素的使用：根据客户名模糊查询客户信息
	 */
	@Test
	public void findCustomerByNameTest() {
		// 通过工具类生成SqlSession对象
		SqlSession session = MyBatisHelper.getSession();
		// 创建Customer对象，封装查询的条件
		Customer customer = new Customer();
		customer.setUsername("cair");
		// 执行sqlSession的查询方法，返回结果集
		List<Customer> customers = session.selectList("top.cairbin.test4" + ".CustomerMapper.findCustomerByName",
				customer);
		// 输出查询结果信息
		for (Customer customer2 : customers) {
			// 打印输出结果
			System.out.println(customer2);
		}
		// 关闭SqlSession
		session.close();
	}

}
```