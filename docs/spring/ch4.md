# Spring AOP与AspectJ

## 概念

**AOP**的全称为**Aspect-Oriented Programming**，即面向切面编程。

想象你是汉堡店的厨师，每一份汉堡都有好几层，这每一层都可以视作一个切面。现在有一位顾客想要品尝到不同风味肉馅的汉堡，如果按照传统的方式，你需要做多个汉堡，每个汉堡只有肉馅是不一样的，但是你每做一个汉堡都要重新制作面包。而聪明的厨师只需做一个汉堡，仅将肉饼那一层分成不同口味的几个区域，这样你就不需要再重复制作面包了。

对于程序员也是一样的，有多少个接口就要写或复制多少代码那一定是无法忍受的，我们只想关心不同的那部分。

尽管想通俗来讲，但是还是要去熟悉专业的概念：

- `Aspect`：切面，类似于Java类声明，里面会有`Pointcut`和`Advice`
- `Joint point`：连接点，在程序执行过程中某个阶段点
- `Pointcut`：切入点，切面与程序流的交叉点，往往此处需要处理
- `Advice`：通知或增强，在切入点处所要执行的代码。可以理解为切面类中的方法。
- `Target object`：目标对象，指所有被通知的对象。
- `Proxy`：代理，将通知应用到目标对象后，被动态创建的对象。
- `Weaving`：织入，将切面代码插到目标对象上，从而生成代理对象的过程。

别担心，我们之后会通过代码来慢慢理解。

## AOP的实现

AOP的实现主要分为**静态代理**和**动态代理**，在本教程中静态代理我们用`AspectJ`，而动态代理用`Spring AOP`。

静态代理在编译期就确定了代理类，而动态代理需要靠反射机制动态生成代理类。

Spring AOP动态代理有两种实现方式：一种是**JDK动态代理**，这种方式需要接口；另一种是**CGLib动态代理**，这种方式不依赖接口。

有了以上的知识，我们开始写代码，首先新创建一个**Maven项目**`top.cairbin.test2`，如果你不会请回去看之前的章节。

然后在`pom.xml`的`<dependencies></dependencies>`之间添加依赖包
```xml
<!-- https://mvnrepository.com/artifact/org.springframework/spring-context -->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>5.3.16</version>
</dependency>		
<!-- https://mvnrepository.com/artifact/cglib/cglib -->
<dependency>
    <groupId>cglib</groupId>
    <artifactId>cglib</artifactId>
    <version>3.3.0</version>
</dependency>		
<!-- https://mvnrepository.com/artifact/org.aspectj/aspectjweaver -->
<dependency>
    <groupId>org.aspectj</groupId>
    <artifactId>aspectjweaver</artifactId>
    <version>1.9.4</version>
</dependency>
```

我们去实现一个`IUser`接口，要求接口内有两个方法`void addUser()`和`void deleteUser()`
```java
package top.cairbin.test2;

public interface IUser {
	void addUser();
	void deleteUser();
}
```

接下来定义一个实现该接口的`User`类
```java
package top.cairbin.test2;

public class User implements IUser{
	@Override
	public void addUser() {
		System.out.println("进行增加用户操作！");
	}
	
	@Override
	public void deleteUser() {
		System.out.println("进行删除用户操作!");
	}
}
```

我们定义一个切面类，该类中的两个方法`void check()`和`void log()`分别模拟权限检查和日志记录功能。

切面类如下所示
```java
package top.cairbin.test2;

public class MyAspect {
	public void check() {
		System.out.println("正在模拟权限认证");
	}
	
	public void log() {
		System.out.println("正在模拟日志记录");
	}
}
```

### JDK动态代理


接下来创建代理类`JdkProxy`，这个类实现了JDK动态代理的`InvocationHandler`接口，并实现代理方法。
```java
package top.cairbin.test2;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;


public class JdkProxy implements InvocationHandler{
	private final IUser user;
	
	public JdkProxy(IUser user) {
		this.user = user;
	}
	
	public static Object createProxy(IUser user) {
		ClassLoader classLoader = JdkProxy.class.getClassLoader();
		// 被代理对象实现的所有接口
		Class[] clazz = user.getClass().getInterfaces();
		// 使用代理类，进行增强，返回的是代理后的对象
		return  Proxy.newProxyInstance(classLoader,clazz,new JdkProxy(user));
	}
	
	@Override
	public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
		// 声明切面
		MyAspect myAspect = new MyAspect();
		// 前增强
		myAspect.check();
		// 在目标类上调用方法，并传入参数
		Object obj = method.invoke(user, args);
		// 后增强
		myAspect.log();
		
		return obj;
	}

}
```

然后尝试在`App.java`的`main`方法中使用它们
```java
package top.cairbin.test2;

public class App 
{
    public static void main( String[] args )
    {
    	// 创建目标对象
    	IUser user= new User();
    	// 创建代理，并从代理中获取增强后的目标对象
    	IUser user2 = (IUser)JdkProxy.createProxy(user);
		// 执行方法
		user2.addUser();
		user2.deleteUser();

    }
}
```

输出结果如下图所示
![](http://img.cairbin.top/img/202403200225898.png)

### CGLib动态代理

我们不妨尝试使用CGLib来玩一下

创建一个新的类，名称为`CglibProxy`，并实现接口`MethodInterceptor`以及相应的方法

```java
package top.cairbin.test2;

import java.lang.reflect.Method;
import org.springframework.cglib.proxy.Enhancer;
import org.springframework.cglib.proxy.MethodInterceptor;
import org.springframework.cglib.proxy.MethodProxy;

public class CglibProxy implements MethodInterceptor {
     public static Object createProxy(Object target){
        Enhancer enhancer = new Enhancer();  
        enhancer.setSuperclass(target.getClass());  
        enhancer.setCallback(new CglibProxy()); 
	    return enhancer.create();
 }
	@Override
	public Object intercept(Object obj, Method method, 	Object[] args, MethodProxy proxy) throws Throwable {
		// 声明切面
		MyAspect myAspect = new MyAspect();
		// 前增强
		myAspect.check();
		//获取增强后的目标对象
		Object target = proxy.invokeSuper(obj, args);
		// 后增强
		myAspect.log();
		return target;
	}
}
```

尝试调用下
```java
package top.cairbin.test2;

public class App 
{
    public static void main( String[] args )
    {
		IUser user = (IUser)CglibProxy.createProxy (new User());
		user.addUser();
		user.deleteUser();
    }
}
```

不出所料，果然成功了
![](http://img.cairbin.top/img/202403200234740.png)


我们仔细观察CGLib的这几段代码，在`CglibProxy`类中我们并没有用到`IUser`这个接口，而是返回Object，然后外面也就是调用者那里强制转换为`IUser`类型！

不妨再“懒”一些，我们借助Spring的依赖注入，从Spring的容器中直接返回增强后的实现了`IUser`接口的对象试一试。

首先在`resources/AppCtx.xml`中编写Bean的配置（这里有坑，如果行不通回前面的文章看看）

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans 
	http://www.springframework.org/schema/beans/spring-beans-4.3.xsd">
	<!-- 目标类 -->
	<bean id="user" class="top.cairbin.test2.User" />
	<!-- 切面类 -->
	<bean id="myAspect" class="top.cairbin.test2.MyAspect" />
	<!-- 使用Spring代理工厂定义一个名称为userProxy的代理对象 -->
	<bean id="userProxy" 
            class="org.springframework.aop.framework.ProxyFactoryBean">
		<!-- 指定代理实现的接口-->
		<property name="proxyInterfaces" 
                      value="top.cairbin.test2.IUser" />
		<!-- 指定目标对象 -->
		<property name="target" ref="user" />
		<!-- 指定切面,织入环绕通知 -->
		<property name="interceptorNames" value="myAspect" />
		<!-- 指定代理方式，true：使用cglib，false(默认)：使用jdk动态代理 -->
		<property name="proxyTargetClass" value="true" />
	</bean>
</beans>
```

修改下`MyAspect`类，并实现接口`MethodInterceptor`

```java
package top.cairbin.test2;
import org.aopalliance.intercept.MethodInterceptor;
import org.aopalliance.intercept.MethodInvocation;

public class MyAspect implements MethodInterceptor {
	@Override
	public Object invoke(MethodInvocation mi) throws Throwable {
		this.check();
		// 执行目标方法
		Object obj = mi.proceed();
		this.log();
		return obj;
	}

	public void check() {
		System.out.println("正在模拟权限认证");
	}
	
	public void log() {
		System.out.println("正在模拟日志记录");
	}
}
```

`App`类中的`main`方法调用如下
```java
package top.cairbin.test2;

import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class App 
{
    public static void main( String[] args )
    {
    	ApplicationContext app = new ClassPathXmlApplicationContext("AppCtx.xml");
    	IUser user = (IUser)app.getBean("userProxy");
    	user.addUser();
    	user.deleteUser();
    }
}
```

点击**运行**得到结果
![](http://img.cairbin.top/img/202403200258702.png)

### AspectJ静态代理

使用AspectJ静态代理，我们重新设计下`MyAspect`切面类
```java
package top.cairbin.test2;
import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.After;
import org.aspectj.lang.annotation.AfterReturning;
import org.aspectj.lang.annotation.AfterThrowing;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;
import org.aspectj.lang.annotation.Pointcut;
import org.springframework.stereotype.Component;
/**
 * 切面类，在此类中编写通知
 */
@Aspect
@Component
public class MyAspect {
	// 定义切入点表达式
	@Pointcut("execution(* top.cairbin.test2.*.*(..))")
	// 使用一个返回值为void、方法体为空的方法来命名切入点
	private void myPointCut(){}
	// 前置通知
	@Before("myPointCut()")
	public void myBefore(JoinPoint joinPoint) {
		System.out.print("前置通知 ：模拟执行权限检查...,");
		System.out.print("目标类是："+joinPoint.getTarget() );
		System.out.println(",被织入增强处理的目标方法为："
		               +joinPoint.getSignature().getName());
	}
	// 后置通知
	@AfterReturning(value="myPointCut()")
	public void myAfterReturning(JoinPoint joinPoint) {
		System.out.print("后置通知：模拟记录日志...," );
		System.out.println("被织入增强处理的目标方法为："
		              + joinPoint.getSignature().getName());
	}
	// 环绕通知	
	@Around("myPointCut()")
	public Object myAround(ProceedingJoinPoint proceedingJoinPoint) 
            throws Throwable {
		// 开始
		System.out.println("环绕开始：执行目标方法之前，模拟开启事务...");
		// 执行当前目标方法
		Object obj = proceedingJoinPoint.proceed();
		// 结束
		System.out.println("环绕结束：执行目标方法之后，模拟关闭事务...");
		return obj;
	}
	// 异常通知
	@AfterThrowing(value="myPointCut()",throwing="e")
	public void myAfterThrowing(JoinPoint joinPoint, Throwable e) {
		System.out.println("异常通知：" + "出错了" + e.getMessage());
	}
	// 最终通知
	@After("myPointCut()")
	public void myAfter() {
		System.out.println("最终通知：模拟方法结束后的释放资源...");
	}
}
```

对于切入点注解`@Pointcut("execution(* top.cairbin.test2.*.*(..))")`表示对`top.cairbin.test2`这个包下的所有类的所有方法生效。

我们再来看看Spring中的`Advice`的几种类型：

- `org.springframework.aop.MethodBeforeAdvice`，前置通知，目标方法执行前实施，可用于权限管理。
- `org.springframework.aop.AfterReturningAdvice`，后置通知，在目标方法执行后实施，用于关闭文件流、上传文件、删除临时文件等。
- `org.aopalliance.intercept.MethodInterceptor`，环绕通知，在目标方法实施前后，一般用于日志或事务管理。
- `org.springframework.aop.ThrowsAdvice`，异常抛出通知，在抛出异常后实施。
- `org.springframework.aop.IntroductionInterceptor`，引介通知，在目标类中添加新方法和属性，可以应用于修改老版本程序。

我们还要实现自动扫描和依赖注入，看看我们的`User`类
```java
package top.cairbin.test2;
import org.springframework.stereotype.Component;

@Component
public class User implements IUser{
	@Override
	public void addUser() {
		System.out.println("进行增加用户操作！");
	}
	
	@Override
	public void deleteUser() {
		System.out.println("进行删除用户操作!");
	}
}
```

自然也少不了`AppCtx.xml`的配置
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
  xmlns:aop="http://www.springframework.org/schema/aop"
  xmlns:context="http://www.springframework.org/schema/context"
  xsi:schemaLocation="http://www.springframework.org/schema/beans 
  http://www.springframework.org/schema/beans/spring-beans-4.3.xsd
  http://www.springframework.org/schema/aop 
  http://www.springframework.org/schema/aop/spring-aop-4.3.xsd
  http://www.springframework.org/schema/context 
  http://www.springframework.org/schema/context/spring-context-4.3.xsd">
      <!-- 指定需要扫描的包，使注解生效 -->
      <context:component-scan base-package="top.cairbin.test2" />
      <!-- 启动基于注解的声明式AspectJ支持 -->
      <aop:aspectj-autoproxy />
</beans>
```

在`main`方法中测试下，为了清楚，我这里仅调用了`addUser`一个方法
```java
package top.cairbin.test2;

import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class App 
{
    public static void main( String[] args )
    {
    	ApplicationContext app = new ClassPathXmlApplicationContext("AppCtx.xml");
    	IUser user = (IUser)app.getBean("user");
    	user.addUser();
    }
}
```
![](http://img.cairbin.top/img/202403200318638.png)

**我们发现当环绕通知与前置通知和后置通知同时使用的时候，优先级如下：**

- 环绕通知开始
- 前置通知
- 方法执行
- 后置通知
- 环绕通知结束

想必到了这里，你对AspectJ的使用有了一定的了解，但是对于相应的注解还是不太清楚，请仔细阅读下方图片中的表格，结合一开始的术语体会下：

![](http://img.cairbin.top/img/202403200322816.png)