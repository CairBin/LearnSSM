# Spring MVC项目创建

## 什么是Spring MVC

Spring MVC是Spring内置的，实现了Web MVC设计模式的框架。

它解决了Web开发过程中很多的问题，例如参数接收、表单验证等。另外它采用松散耦合可插拔组件等结构，具有相对较高的灵活性和扩展性。

[Spring MVC官方文档](https://docs.spring.io/spring-framework/reference/web.html)可参考 [docs.spring.io](https://docs.spring.io/spring-framework/reference/web.html)。


## Spring MVC执行流程

按顺序可分为如下流程

- 用户请求被`DispatcherServlet`进行拦截处理
- `DispatcherServlet`收到请求调用`HandlerMapping`
- `HandlerMapping`找到具体的处理器，生成处理器对象及处理器拦截器，再一起返回给`DispatcherServlet`
- `DispatcherServlet`调用`HandlerAdapter`
- `HandlerAdapter`经过适配调用具体的处理器
- Controller执行完成返回`ModelAndView`对象
- `HandlerAdapter`将`ModelAndView`返回给`DispatcherServlet`
- `DispatcherServlet`将`ModelAndView`传给`ViewReslover`
- `ViewReslover`解析`ModelAndView`后返回View（给`DispatcherServlet`
- `DispatcherServlet`根据View进行渲染
- `DispatcherServlet`响应View给用户

通过上面流程可知，程序员需要配置`DispatcherServlet`，并开发`View`和`Controller/Handler`。


## 项目创建

即然如此，我们就来创建一个Spring MVC项目。


打开Eclipse，创建一个Maven项目（想必经过前面学习已经很熟了），项目名称`top.cairbin.test7`。

但是需要注意，创建时请勾选下图的`Add project(s) to working set`

![](http://img.cairbin.top/img/202403280208090.png)

在`Select an Archetype`这一步需要注意，我们不再用之前的`maven-archetype-quickstart`，而是`maven-archetype-webapp`，如下图所示

![](http://img.cairbin.top/img/202403250029378.png)

创建完成后项目目录结构大概如下

![](http://img.cairbin.top/img/202403250033942.png)

我们还需要引入Spring MVC的依赖包以及Servlet，这你很清楚应该在`pom.xml`里配置

```xml
<!-- 添加servlet依赖 -->
<dependency>
	<groupId>javax.servlet</groupId>
	<artifactId>javax.servlet-api</artifactId>
	<scope>provided</scope>
	<version>3.1.0</version>
</dependency>
    
<!-- 添加spring依赖 -->
<dependency>
	<groupId>org.springframework</groupId>
	<artifactId>spring-core</artifactId>
	<version>5.3.23</version>
</dependency>
<dependency>
	<groupId>org.springframework</groupId>
	<artifactId>spring-web</artifactId>
	<version>5.3.23</version>
</dependency>
<dependency>
	<groupId>org.springframework</groupId>
	<artifactId>spring-webmvc</artifactId>
	<version>5.3.23</version>
</dependency>
```

最终效果看起来是这样

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>top.cairbin</groupId>
  <artifactId>test7</artifactId>
  <packaging>war</packaging>
  <version>0.0.1-SNAPSHOT</version>
  <name>test7 Maven Webapp</name>
  <url>http://maven.apache.org</url>
  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>3.8.1</version>
      <scope>test</scope>
    </dependency>
    
    <!-- 添加servlet依赖 -->
	<dependency>
		<groupId>javax.servlet</groupId>
		<artifactId>javax.servlet-api</artifactId>
		<scope>provided</scope>
		<version>3.1.0</version>
	</dependency>
    
	<!-- 添加spring依赖 -->
	<dependency>
		<groupId>org.springframework</groupId>
		<artifactId>spring-core</artifactId>
		<version>5.3.23</version>
	</dependency>
	<dependency>
		<groupId>org.springframework</groupId>
		<artifactId>spring-web</artifactId>
		<version>5.3.23</version>
	</dependency>
	<dependency>
		<groupId>org.springframework</groupId>
		<artifactId>spring-webmvc</artifactId>
		<version>5.3.23</version>
	</dependency>
    
  </dependencies>
  <build>
    <finalName>test7</finalName>
  </build>
</project>
```


相比于以往的项目，你会发现`webapp`已经为我们创建了一个resources资源文件夹，并且`src/main`下多了一个`webapp`目录。`webapp`它一般存放前端相关的文件。

![](http://img.cairbin.top/img/202403250033942.png)


而`webapp`目录下有一个`WEB-INF`，里面有一个默认生成的`web.xml`配置文件。还记得我们刚才提到的Spring MVC执行流程吗，用户请求会被`DispatcherServlet`拦截，它叫**前端控制器**，而这个XML文件就是用来对它进行设置的。

我们对`web.xml`进行修改

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	      xmlns="http://java.sun.com/xml/ns/javaee"
	      xsi:schemaLocation="http://java.sun.com/xml/ns/javaee 
	      http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
	id="WebApp_ID" version="3.0">
	<servlet>
	    <!-- 配置前端过滤器 -->
		<servlet-name>springmvc</servlet-name>
		<servlet-class>
              org.springframework.web.servlet.DispatcherServlet
         </servlet-class>
		<!-- 初始化时加载配置文件 -->
		<init-param>
			<param-name>contextConfigLocation</param-name>
			<param-value>classpath:springCtx.xml</param-value>
		</init-param>
		<!-- 表示容器在启动时立即加载Servlet -->
		<load-on-startup>1</load-on-startup>
	</servlet>
	<servlet-mapping>
		<servlet-name>springmvc</servlet-name>
		<url-pattern>/</url-pattern> 
	</servlet-mapping>
</web-app>
```

注意`<param-value>classpath:springCtx.xml</param-value>`这一段引入了一个文件，是用来配置Controller映射信息的。


我们再来看看，`webapp`下还有一个`index.jsp`文件，它正是你的前端代码文件，它的内容如下

```jsp
<html>
<body>
<h2>Hello World!</h2>
</body>
</html>
```

你可能对此表示疑惑，这不就是一个HTML文件，现实一个标题，标题内容为`Hello World!`吗。没错，但是对于传统的HTML来讲页面是静态的，而我们的JSP文件则能动态获取后端一些内容。

有了View来实现USL前台还不够，我们还要实现USL后台，接下来我们在`src/main/java`下创建包`top.cairbin.test7.controller`，并在这个包下创建控制器类`TestController`

```java
package top.cairbin.test7.controller;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import org.springframework.web.servlet.ModelAndView;
import org.springframework.web.servlet.mvc.Controller;


public class TestController implements Controller{
	@Override
	public ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response)  {
         // 创建ModelAndView对象
		ModelAndView mav = new ModelAndView();
         // 向模型对象中添加数据
		mav.addObject("message", "Hello Spring");
         // 设置逻辑视图名
		mav.setViewName("/WEB-INF/views/hello.jsp");
         // 返回ModelAndView对象
		return mav;
	}
}
```

还记得刚才说的`<param-value>classpath:springCtx.xml</param-value>`吗，我们创建了控制器，还要告诉Spring MVC控制器与View的映射关系。

`springCtx.xml`这个文件并不存在我们项目中，需要手动添加。在resources资源文件夹里创建它。

![](http://img.cairbin.top/img/202403250105758.png)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
	http://www.springframework.org/schema/beans/spring-beans.xsd">
	<!-- 配置处理器Handle，映射“/hello”请求 -->
	<bean name="/hello" 
			class="top.cairbin.test7.controller.TestController" />
	<!-- 处理器映射器，将处理器Handle的name作为url进行查找 -->
	<bean class=
	"org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping" />
	<!-- 处理器适配器，配置对处理器中handleRequest()方法的调用-->
	<bean class=
	"org.springframework.web.servlet.mvc.SimpleControllerHandlerAdapter" />
	<!-- 视图解析器 -->
	<bean class=
	"org.springframework.web.servlet.view.InternalResourceViewResolver">
	</bean>
</beans> 
```

请务必注意`<bean name="/hello"  class="top.cairbin.test7.controller.TestController" />`这一段的包名与你控制器一致。它告诉Spring MVC，将这个控制器映射到`/hello`路径上去，一会你就能在浏览器通过`http://ip:port/项目名/hello`的形式去访问它。

在`WEB-INF`里创建`views`文件夹，请注意是`WEB-INF`下创建。

并在这个`viewsview`文件夹里新建jsp文件`hello.jsp`

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8"
     pageEncoding="UTF-8"%>
<%@page isELIgnored="false" %>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" 
     "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<title>Hello SpringMvc</title>
</head>
<body>
     <h1>${message}</h1>
</body>
</html> 
```

我们在这里使用`${...}`的形式来使用后端提供的接口，以此获取后端返回的对象。

那么问题来了，我们这个项目该怎么运行呢？显然对于Web项目我们需要Http服务器。

还记得我们在配置环境的时候安装的**Tomcat9**吗，现在终于要到了使用它的时候了！

另外，我们注意之前编写的`TestController`这个控制器，它需要导入的包中可能有两个报错。

```java
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
```

如果出现这种情况，我们右键这个项目，打开属性

![](http://img.cairbin.top/img/202403280224696.png)

找到`Java Build Path`选项的`Libraries`选项卡，点击`Add Library...`

![](http://img.cairbin.top/img/202403280226644.png)

选择`Server Runtime`选项，然后下一步

![](http://img.cairbin.top/img/202403280226130.png)

选择之前配置好的服务器（就是在环境配置那篇文章里的）

![](http://img.cairbin.top/img/202403280227540.png)

保存并退出。

接下来我们尝试运行它，与以往不同的是，我们需要**Run As里面的Run on server**

![](http://img.cairbin.top/img/202403280230711.png)

如果你第一次这样做，会弹出个窗口

![](http://img.cairbin.top/img/202403280231148.png)

选择Apache里对应的服务器，我这里是Tomcat9.0

![](http://img.cairbin.top/img/202403280232536.png)

然后点击完成。很显然，你的浏览器已经打开了，说明你的项目跑成功了。并且在Eclipse的Package Explorer中多了个Servers文件夹。

![](http://img.cairbin.top/img/202403280234526.png)

我们在浏览器访问的是`index.jsp`所渲染的页面

![](http://img.cairbin.top/img/202403280235498.png)

在地址栏输入`http://localhost:8080/test7/hello`访问控制器和`hello.jsp`所渲染的页面

![](http://img.cairbin.top/img/202403280237978.png)

## 问题

在这一节其实有很多坑，一是Tomcat版本在10.0以上的话，包名变化问题。

另外如果当初配置环境的时候，Eclipse没选Web选项，这里有些配置是没有的。

上面这些问题请参考环境配置和此教程的**问题**模块。

如果你项目菜单里的`Run as`没有`Run on server`选项的话需要如下操作

打开项目属性

![](http://img.cairbin.top/img/202403280224696.png)

搜索`project facets`，勾选`Dynamic Web Module`选项

![](http://img.cairbin.top/img/202403280241112.png)

保存并退出。


接下来如果操作不当的话可能导致`Servers`文件夹下配置文件里有多个context，这时候需要打开`server.xml`进行就该

![](http://img.cairbin.top/img/202403280244251.png)

![](http://img.cairbin.top/img/202403280245998.png)

找到如下片段`<Context>`标签，**把重复的删除**

![](http://img.cairbin.top/img/202403280247074.png)
