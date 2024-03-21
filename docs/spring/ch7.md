# Spring事务管理

## 概念

**事务（Transaction）**是访问并可能操作各种数据项的一个数据库操作序列，这些操作**要么全部执行，要么都不执行，是一个不可分割的工作单元**。

事务有如下特性：

- 原子性
- 隔离性
- 一致性
- 持久性

如果多个事务同时操作同一批数据，则会引发并发异常，设置不同的隔离级别可以解决这些问题。事务的隔离级别如下
![](http://img.cairbin.top/img/202403200602616.png)

隔离界别从小到大，安全性越高，但效率就越低。

事务的传播行为是指在同一个方法中，不同操作前后所使用的事务。传播行为可以控制是否需要创建事务以及如何创建事务，Spring默认传播行为是REQUIRED。
![](http://img.cairbin.top/img/202403200614154.png)

事务的管理方式主要分为两种：

- 编程式事务管理：通过编写代码实现的事务管理，包括定义事务的开始、正常执行后的事务提交和异常时的事务回滚
- 声明式事务管理：通过AOP技术实现的事务管理，主要思想是将事务作为一个“切面”代码单独编写，然后通过AOP技术将事务管理的“切面”植入到业务目标类中

为了解耦，我们一般用后者。


## 声明式事务管理

### 基于XML

Spring的声明式事务管理可以通过两种方式来实现，一种是基于XML的方式，另一种是基于注解的方式。

基于XML方式的声明式事务是在配置文件中通过`<tx:advice>`元素配置事务规则来实现的。当配置了事务的增强处理后，就可以通过编写的AOP配置，让Spring自动对目标生成代理。其属性如下。
![](http://img.cairbin.top/img/202403200619213.png)


### 基于注解

我们更关心基于注解的方式，下面用代码来了解下

我们用之前的`top.cairbin.test3`项目

`app.xml`文件内容如下

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
  xmlns:context="http://www.springframework.org/schema/context"
  xmlns:tx="http://www.springframework.org/schema/tx"
  xsi:schemaLocation="http://www.springframework.org/schema/beans 
  http://www.springframework.org/schema/beans/spring-beans.xsd
  http://www.springframework.org/schema/tx
  http://www.springframework.org/schema/tx/spring-tx.xsd
  http://www.springframework.org/schema/context 
  http://www.springframework.org/schema/context/spring-context.xsd">

	<!-- 指定需要扫描的包，使注解生效 -->
	<context:component-scan
		base-package="top.cairbin.test3" />
	<!-- 配置dataSource -->
	<bean id="dataSource"
		class="org.springframework.jdbc.datasource.DriverManagerDataSource">
		<!--数据库驱动 -->
		<property name="driverClassName"
			value="com.mysql.jdbc.Driver" />
		<!--连接数据库的url -->
		<property name="url"
			value="jdbc:mysql://localhost:3306/db_javaee" />
		<!--连接数据库的用户名 -->
		<property name="username" value="db_javaee" />
		<!--连接数据库的密码 -->
		<property name="password" value="dbjavaeepassword" />
	</bean>
	<!-- 2配置JDBC模板 -->
	<bean id="jdbcTemplate"
		class="org.springframework.jdbc.core.JdbcTemplate">
		<!-- 默认必须使用数据源 -->
		<property name="dataSource" ref="dataSource" />
	</bean>
	
	
	<bean id="transactionManager" class=
     "org.springframework.jdbc.datasource.DataSourceTransactionManager">
		<property name="dataSource" ref="dataSource" />
	</bean>	
    <!-- 注册事务管理器驱动 -->
	<tx:annotation-driven transaction-manager="transactionManager"/>
</beans>
```

在`IAccountDao`中声明方法

```java
public void transfer(String outUser,String inUser,Double money);
```

在`AccountDao`类中添加方法及注解
```java
@Transactional(propagation = Propagation.REQUIRED, isolation = Isolation.DEFAULT, readOnly = false)
public void transfer(String outUser, String inUser, Double money) {
    // 收款时，收款用户的余额=现有余额+所汇金额
    this.jdbcTemplate.update("update account set balance = balance +? "
            + "where username = ?",money, inUser);
    // 模拟系统运行时的突发性问题
    int i = 1/0;
    // 汇款时，汇款用户的余额=现有余额-所汇金额
    this.jdbcTemplate.update("update account set balance = balance-? "
            + "where username = ?",money, outUser);
}
```

在`JdbcTemplateTest`测试类中添加测试方法
```java
@Test
public void transTest(){		
    accountDao.transfer("lisi", "zhangsan", 100.0);
    // 输出提示信息
    System.out.println("转账成功！");
}
```

运行程序前向数据库中插入两条记录，我们使用终端或者`sqlyog`连接数据库，我这里以Mac的终端为例

```sql
mysql -u db_javaee -p
```

注意这里，我们用的不再是`root`账户，而是之前创建的`db_javaee`

输入密码`dbjavaeepassword`，如果你之前设置的与我不一样这里换成你的密码

输入密码的时候是不显示密码内容的，敲击回车，如果界面跟我下面差不多就说明成功了

![](http://img.cairbin.top/img/202403211157300.png)

后面来写SQL语句，首先把数据库切换到`db_javaee`下面
```sql
USE db_javaee;
```

如果出现`Database changed`提示则表示成功。

我们先来查看下之前创建的`Account`表

```sql
SELECT * FROM account;
```
这是我的表里面的内容，你的应该与我不太一样，不过不影响接下来的操作。

![](http://img.cairbin.top/img/202403211203501.png)

我们使用一下语句来插入两条数据
```SQL
INSERT INTO account(id,username,balance) VALUES(17,'zhangsan',100);
INSERT INTO account(id,username,balance) VALUES(18,'lisi',200);
```

成功的话会有提示
![](http://img.cairbin.top/img/202403211208001.png)

来看下目前表里的数据

```sql
SELECT * FROM account;
```

![](http://img.cairbin.top/img/202403211209163.png)

接下来我们回到**Eclipse IDE**，运行我们的代码
![](http://img.cairbin.top/img/202403211210424.png)

运行结果如下
![](http://img.cairbin.top/img/202403211211898.png)

不出意外的话该出意外了，注意看左边的`transTest()`方法竟然报错，那是正常的，因为我们`public void transfer(String outUser, String inUser, Double money);`代码写了`int i = 1/0;`这种语句来模拟报错。

我们到数据库中查看数据
```sql
SELECT * FROM account;
```
![](http://img.cairbin.top/img/202403211213178.png)

很显然数据没有啥变化，我们将`transfer`方法中的`int i = 1/0;`删掉试试。注意方法名称，别搞错了。
![](http://img.cairbin.top/img/202403211216318.png)

```java
@Transactional(propagation = Propagation.REQUIRED, isolation = Isolation.DEFAULT, readOnly = false)
public void transfer(String outUser, String inUser, Double money) {
    // 收款时，收款用户的余额=现有余额+所汇金额
    this.jdbcTemplate.update("update account set balance = balance +? "
            + "where username = ?",money, inUser);
    // 模拟系统运行时的突发性问题
    // int i = 1/0;
    // 汇款时，汇款用户的余额=现有余额-所汇金额
    this.jdbcTemplate.update("update account set balance = balance-? "
            + "where username = ?",money, outUser);
}
```

运行一下，发现控制台输出`转账成功！`。

![](http://img.cairbin.top/img/202403211218529.png)

回到数据库查看下
```sql
SELECT * FROM account;
```

下图是操作前后两次的对比，上面是转账成功之前的，后面是转账成功之后的。

![](http://img.cairbin.top/img/202403211219210.png)

数据显然是发生变化了！

## 问题

到了这里，你可能会有疑问，敲了这么多的代码，我们还不断地进行测试、查看数据库，到底是想干什么。

```java
@Transactional(propagation = Propagation.REQUIRED, isolation = Isolation.DEFAULT, readOnly = false)
public void transfer(String outUser, String inUser, Double money) {
    // 收款时，收款用户的余额=现有余额+所汇金额
    this.jdbcTemplate.update("update account set balance = balance +? "
            + "where username = ?",money, inUser);
    // 模拟系统运行时的突发性问题
    int i = 1/0;
    // 汇款时，汇款用户的余额=现有余额-所汇金额
    this.jdbcTemplate.update("update account set balance = balance-? "
            + "where username = ?",money, outUser);
}
```

我们看看上方的代码。**在前面提到过事务的特性是原子的，也就是不可分割的。**倘若我们没有事务，当`this.jdbcTemplate.update(...);`这条语句执行的时候，数据库就已经更新了，而`int i = 1/0;`却出现了一个突发性问题，按理来说我们转账`transfer`这一方法不应该成立才对，而数据库中的记录的确发生了变动。

如果你是一个恶意用户，然后你发现了这一漏洞，向银行存款，每次存款失败代码异常银行都退回你钱，然后你不断重复这个操作就可以使你的账户余额不断增加，然而你实际上并没有存一分钱进去，这显然是不合理的！

然而我们有了事务，就不会发生这种情况了，因为对于`transfer`这一操作是不可分割成小操作的，也就是说对于方法里面的东西要么全成功，要么全失败回滚，不可能出现部分成功部分失败这种情况。

## 结束

**Spring初步**这一部分就到此为止了，你会发现对于数据库操作在代码里嵌入SQL语句是很麻烦而且不可靠的，如果不做好过滤，用户提交一段包含SQL语句的字符串就有可能利用你的权限操作数据库，还会有安全问题，所以接下来我们将学习**MyBatis**这个框架。