# 核心类、常用注解与RESTful

## DispatcherServlet

`DispatcherServlet`全称`org.springframework.web.servlet.DispatcherServlet`。DispatcherServlet并不直接处理请求，它只负责根据请求的信息把请求转发给合适的处理器，然后由处理器来执行实际的处理过程并生成响应，它是Spring MVC框架的入口点，它将所有这些步骤组合在一起，使得开发者可以更轻松地构建Web应用程序并处理客户端请求。每个Spring MVC应用程序通常只有一个。

![](http://img.cairbin.top/img/202403312348367.png)

我们可以在`web.xml`里进行配置：

```xml
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
    <url-pattern>/springmvc/*</url-pattern>
</servlet-mapping>
```

上面例子中所有以`/springmvc`开头的请求都会被名称为`springmvc`的DispatcherServlet处理。

另外，在上述代码中，有两个可选项`<load-on-startup>`和`<init-param>`。

`<load-on-startup>`如果元素的值为1，则启动程序的时候会立刻加载Servlet；如果元素不存在，则在第一个请求时加载。

`<init-param>`存在并配置了路径，则会根据路径寻找配置文件，否则将导WEB-INF下寻找`servletName-servlet.xml`命名形式的文件。

## ViewResolver

ViewResolver即视图解析器，我们可以在`springCtx.xml`文件中配置它。

回顾之前的`springCtx.xml`

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

再来看看之前的controller

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

我们在控制器中每次都要指定`jsp`格式文件和`/WEB-INF/views/`路径是一件很头疼的事，所以可以修改视图解析器的配置来帮助我们操作。

```xml
<bean id="viewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
	 <!-- 设置前缀 -->
	<property name="prefix" value="/WEB-INF/views/" />
    <!-- 设置后缀 -->
	<property name="suffix" value=".jsp" />
</bean>
```

## @Controller、@RequestParam与映射相关的注解

### @Controller

在程序中我们可以使用`@Controller`来声明这是一个控制器用于依赖注入，而无需去实现`Controller`接口。

### @RequestMapping

`@RequestMapping`注解可以很方便地让我们设置类或者方法对应的URL路径，只需要将它们写在类或者方法上面即可

例如
```java
[...]

@Controller
@RequestMapping("/myController")
class MyController{
    [...]
}
```



```java
[...]
@Controller
@RequestMapping("/myController")
class MyController{
    [...]

    @RequestMapping("/myFunc1")
    public MyType MyFunc1([...]){
        [...]
    }

    //还可以指定路径和请求类型(例如GET)
    @RequestMapping(path="/myFunc2", method=RequestMethod.GET)
    public MyType MyFunc2([...]){
        [...]
    }

    //还有URI模式，路径可随参数变化。
    //比如请求某些文章，都有各自ID，它们对应的方法都是这个，路径仅有id不一样，返回对应内容，就可以这样写，此时这里的param对应文章id。
    //还可以指定路径和请求类型(例如GET)
    @RequestMapping(path="/{param}}", method=RequestMethod.GET)
    public MyType MyFunc3(@PathVariable ParamType param, [...]){
        doForParam(param);
        [...]
    }
}
```

需要注意`@RequestMapping`有两种属性可以指定映射路径，分别是`value`和`path`

`value`被指定会覆盖默认值，`path`也一样，二者也都能匹配占位符。实际上`path`就是`value`的别名，没有区别。

另外使用占位符的URL路径需要对参数添加注解`@PathVariable`

### 组合注解

为了更方便地指定Http请求方法类型，Spring也为我们定义了一些组合注解。

- `@GetMapping`匹配Get请求
- `@PostMapping`匹配Post请求
- `@PutMapping`匹配Put请求
- `@DeleteMapping`匹配Delete请求
- `@PatchMapping`匹配Patch请求

至于这些请求是干什么用的，分别对应什么场景，因为并不属于主要内容，此处不过多描述，请查阅HTTP协议文档。（尽管如此，你必须对这些方法及HTTP语义有一定了解）

### @RequestParam

`@RequestParam`注解为请求指定参数，用法如下。

这是Controller里的一个方法

```java
@RequestMapping("/hello")
public ModelAndView showMessage(@RequestParam(value = "name", required = false, defaultValue = "Spring") String name) {

    ModelAndView mv = new ModelAndView("hello");//指定视图
　　　　　
    mv.addObject("message", message);
    mv.addObject("name", name);
    return mv;
}
```


## 实践

与`test7`一样，创建`webapp`类型的maven项目`top.cairbin.test8`。（如果不会请返回上一节）

在`pom.xml`里引入依赖文件。

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

在`web.xml`里进行配置

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

在`src/main/resources`里创建`springCtx.xml`配置文件

这里与`test7`就不一样了，我们指定jsp文件的前后缀，并利用自动扫描来帮助我们完成控制器以及其他类的注入。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	 xmlns:context="http://www.springframework.org/schema/context"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
	http://www.springframework.org/schema/beans/spring-beans.xsd
	http://www.springframework.org/schema/context 
  http://www.springframework.org/schema/context/spring-context.xsd">
	<context:component-scan base-package="top.cairbin.test8" />
	<bean id="viewResolver" class=
    "org.springframework.web.servlet.view.InternalResourceViewResolver">
	     <!-- 设置前缀 -->
	     <property name="prefix" value="/WEB-INF/views/" />
	     <!-- 设置后缀 -->
	     <property name="suffix" value=".jsp" />
	</bean>	
</beans>
```

请注意自动扫描的路径与你项目是否一致。另外，因为分层设计，我们在之后肯定不会仅在controller层进行依赖注入，在其他层也会，我们创建的包名都是`top.cairbin.test8.xxx`，也就是说都是`top.cairbin.test8`的子包，所以这里指定`top.cairbin.test8`进行扫描即可。

在WEB-INF文件夹下创建`views`文件夹，并在`views`下创建`hello.jsp`文件

```jsp
<%@ page language="java" contentType="text/html; charset=utf-8"
	pageEncoding="utf-8"%>
<%@ page isELIgnored="false"%>
<!DOCTYPE html>
<html>

<head>
	<title>hello</title>
</head>

<body>
	<h1>Hello, ${name}!</h1>
</body>

</html>
```


接下来创建`top.cairbin.test8.controller`包，包下创建类`MyController`，并将控制器映射到`/hello`路径下，将`sayHello`方法映射到`/hello/sayHello/{name}`路径，其中`{name}`是一个占位符，随用户提供的参数而定，让`hello.jsp`模板渲染，并显示`hello,`后面跟这个参数的值。

```java
package top.cairbin.test8.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.servlet.ModelAndView;

@Controller
@RequestMapping("/hello")
public class MyController {
	
	@GetMapping(path="/sayHello/{name}")
	public ModelAndView sayHello(
			@PathVariable String name) throws Exception {
		//指定视图
		ModelAndView mv = new ModelAndView("hello");
		//向视图中添加所要展示或使用的内容，将在页面中使用
		mv.addObject("name", name);
		return mv;
	}	
}
```

我们来运行一下，右键项目->`Run as...`->`Run on server`

选择你的Tomcat服务器并运行，这里有问题同样返回上一节。

浏览器访问`http://localhost:8080/test8/hello/sayHello/CairBin`

![](http://img.cairbin.top/img/202404010139510.png)

我们来看看在URL里不用占位符的方式传参是怎么样的。
在`MyController`里添加一个方法

```java
@GetMapping(path="/sayHello2")
public ModelAndView sayHello2(
		String name) throws Exception {
	//指定视图
	ModelAndView mv = new ModelAndView("hello");
	//向视图中添加所要展示或使用的内容，将在页面中使用
	mv.addObject("name", name);
	return mv;
}
```

在浏览器访问`http://localhost:8080/test8/hello/sayHello2`

![](http://img.cairbin.top/img/202404010143462.png)

我们发现并没有任何数据。

换这个URL试试`http://localhost:8080/test8/hello/sayHello2?name=cairbin`

![](http://img.cairbin.top/img/202404010144066.png)

amazing，竟然有了。实际上这对应两种不同的风格。
你可能迷茫了，我到底以什么方式传递参数，接下来将具体说明。


## RESTful与RPC风格


请容许我从网上找一张图片

![](http://img.cairbin.top/img/202404010202676.png)

学过网络的都知道，TCP/IP模型的应用层实际上对应OSI的应用层、表示层和会话层。

对于我们WEB使用的HTTP协议来讲，它是应用层协议。

但是在上古年代，人们还在为网络世界里不同机器之间的通信方式而头疼，而最基础的方式是通过socket编程来实现的，不过这个难度有些大，人们迫切需要一个对新手友好且简单的方式。

于是，1984年，有位大佬叫 Bruce Jay Nelson，发表了一篇论文叫做 *Implementing Remote Procedure Calls*，定义了一种标准叫RPC。

RPC希望客户端可以像调用本地接口一样来调用服务端，RPC模式分为三层，RPCRuntime 负责最底层的网络传输，Stub 处理客户端和服务端约定好的语法、语义的封装和解封装，这些调用远程的细节都被这两层搞定了，用户和服务器这层就只要负责处理业务逻辑，调用本地 Stub 就可以调用远程。

也就是说RPC可以基于TCP/UDP传输层协议传输，也可以使用它们的上层协议HTTP。也正如此HTTP更关注服务提供方，对于客户端怎么调用是不关心的；而对于RPC还要满足更多的东西，也就是需要客户端接口和服务端保持一致（客户端可以像调用本地接口一样来调用服务端）。

这种API接口的设计风格就被称为RPC风格。

而对于REST，中文描述表述性状态传递，实际上主要关注资源定位，REST是把服务端方法写好，客户端不想知道方法，只想获取资源，这与HTTP语义天然一致。

所以从设计上看，RPC是面向方法的，REST是面向资源的。

总的来说，RPC通常是服务器和服务器之间的通信，比如和中间件的通信，MQ、分布式缓存、分布式数据库等等。而REST通常是面向客户端的（一般是浏览器）。使用场景不一样。

因此你在设计Web的时候两种接口风格实际上都会用到，而对于浏览器来讲，我们为方便实现客户端与服务器的缓存等功能，还是尽量用RESTful风格比较好。

但是RESTful，它毕竟只是一种风格，或者说是一种建议，即使你不按照它来设计或者说“接口不够RESTful”也符合规范，但是我们为了尽量靠拢HTTP语义，还是尽量去这样做。


接下来我们对比两种风格差异，分别以CRUD为例。

### RPC

```
http://127.0.0.1/article/query?id=1  查询文章(GET)

http://127.0.0.1/article/add         新增文章(POST)

http://127.0.0.1/article/update      更新文章(POST)

http://127.0.0.1/article/delete?id=1 删除文章(GET或POST)

```

### RESTful

```
http://127.0.0.1/article/1  查询文章(GET)

http://127.0.0.1/article    新增文章(POST)

http://127.0.0.1/article    更新文章(PUT)

http://127.0.0.1/article/1  删除文章(DELETE)

```