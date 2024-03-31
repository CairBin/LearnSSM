# JSP语法入门

## 前提

在前一节中我们已经写过JSP的代码了，这一节将单独介绍JSP一些基础语法。当然，你可以跳过这一节，当后面有代码不太理解的时候再回来阅读。

## 中文编码问题

如果中文乱码，看看JSP是否是以UTF8的方式编码，使用此编码在JSP文件最上方需要添加下面内容

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
```

## 脚本程序

脚本程序是包含任意Java语句、变量、方法或表达式。

形式如下：

```jsp
<% 脚本程序代码片段 %>
```

我们借助上一节的`top.cairbin.test7`，稍微对`hello.jsp`改造下看看效果。

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
     <h1><% out.println("Hello JSP"); %></h1>
</body>
</html> 
```

按照上述逻辑来讲应该会在页面显示一个h1的标题，内容为`Hello JSP`

我们访问`http://localhost:8080/test7/hello`来看看，果然如此。哦，对了，在写语句的时候别忘了那该死的**分号**。

![](http://img.cairbin.top/img/202403312157673.png)

你也可以选择以下等价的XML语句（仍是在JSP文件里写，这里对其他部分省略，用`[...]`表示）

```jsp
[...]

<h1>
    <jsp:scriptlet>
 		out.println("Hello JSP");
	</jsp:scriptlet>
</h1>

[...]
```

你大致已经知道JSP书写方式了，为了简洁，在接下来的介绍中除非特殊我们将不再给出演示结果，而是仅介绍用法，请自行在项目中进行测试。


## JSP表达式

表达式先被转化成String，然后被插入到该出现的地方。
由于表达式的值会被转化成String，所以您可以在一个文本行中使用表达式而不用去管它是否是HTML标签。

形式如下

```jsp
<%= 这里写jsp表达式，注意前面那个等号 %>
```

它也有等价的XML形式

```jsp
<jsp:expression>
    表达式
</jsp:expression>
```

示例，注意这里不能用分号结尾

```jsp
<h1>
    日期 <%= (new java.util.Date()).toLocaleString() %>
</h1>
```

## JSP声明

JSP声明可以声明一个或多个变量、方法等供后面代码使用。

JSP声明形式

```jsp
<%! 这里写声明 %>
```

等价的XML

```jsp
<jsp:declaration>
   声明的代码片段
</jsp:declaration>
```

示例

```jsp
<%! int num = 114514; %>
```

## JSP注释

```jsp
<%-- 我就是JSP注释，此处不会被页面展示，也不会被浏览器看到 --%>

<!-- 我是HTML注释，此处内容页面不展示，但会在浏览器查看源码中看到 -->
```

你可以在JSP文件中使用JSP注释和HTML注释，**但应当注意JSP注释与HTML有些不同，JSP注释不会被发送到浏览器，而HTML注释会被发送至浏览器并且能在浏览器查看源代码功能中看到！！！在HTML注释中不要写敏感信息！！！**

## JSP指令

形式如下

```jsp
<%@ 指令 属性="值" %>
```

经常用到的JSP指令标签有：

- `<%@ page ... %>`定义页面依赖属性，比如脚本语言、error页、缓存需求、编码等等
- `<%@ include ... %>`包含其他文件
- `<%@ taglib ... %>`引入标签库的定义，可以自定义标签


对于page常用属性有：

- `language`声明当前页面脚本的语言，默认为java
- `extends`指定jsp编译成servlet之后所需要继承的类或者所实现的接口
- `import`用于导入当前脚本中可能使用到的其他包里面的类
- `info`包含jsp的信息，一般作为当前jsp文件的说明用。可以通过`getServletInfo()`来获取
- `errorPage`指定当前jsp文件发生错误时，自动调用改属性值指定的jsp文件。如果不指定当前属性值，当发生错误时，会抛出异常信息给客户。
- `contentType`指定生成网页的文件格式和编码字符集
- `isErrorPage`用于指定当前jsp文件是否为错误处理jsp文件


## JSP行为

JSP行为使用XML语法结构控制servlet引擎。

它需要严格遵守XML标准：

```jsp
<jsp:行为名称 属性="值" />
```

- `jsp:include`用于在当前页面中包含静态或动态资源
- `jsp:useBean`寻找和初始化一个JavaBean组件
- `jsp:setProperty`设置JavaBean组件的值
- `jsp:getProperty`将JavaBean组件的值插入到output中
- `jsp:plugin`用于在生成HTML页面中包含Apple和JavaBean对象
- `jsp:element`动态创建一个XML元素
- `jsp:forward`从一个JSP文件向另一个文件传递一个包含用户请求的request对象
- `jsp:attribute`定义动态创建的XML元素属性
- `jsp:body`定义动态创建的XML元素的主体
- `jsp:text`用于封装模板数据

## JSP隐式对象

JSP有九个无需额外声明或初始化的对象:

- `request`: HttpServletRequest类的实例，代表 HTTP 请求的对象，包含客户端发送到服务器的信息，如表单数据、URL参数等。
- `response`: HttpServletResponse类的实例，代表 HTTP 响应的对象，用于向客户端发送数据和响应。
- `out`: JspWriter类的实例，用于向客户端输出文本内容的对象，通常用于生成HTML。
- `session`: HttpSession类的实例，代表用户会话的对象，可用于存储和检索用户特定的数据，跨多个页面。
- `application`: ServletContext类的实例，代表 Web 应用程序的上下文，可以用于存储和检索全局应用程序数据。
- `config`: ServletConfig类的实例，包含有关当前 JSP 页面的配置信息，例如初始化参数。
- `pageContext`: PageContext类的实例，提供对JSP页面所有对象以及命名空间的访问
- `page`: 类似于 Java 类中的 this 关键字，代表当前 JSP 页面的实例，可以用于调用页面的方法。
- `exception`: exception 类的对象，代表发生错误的 JSP 页面中对应的异常对象，用于处理 JSP 页面中的异常情况，可用于捕获和处理页面中发生的异常。

## 控制流语句

### 判断语句

```jsp
<%! int num=114514; %>

<% if(num==114514) { %>
    <p>114514</p>
<% }else{ %>
    <p>不是114514</p>
<% } %>
```

### switch语句

switch语句与判断语句不一样，switch语句整个都被包裹在`<% %>`之中

当然，别忘了`break`。

```jsp
<%! int num=114514; %>

<%
switch(num){
    case 114514:
        out.println("114514");
        break;
    default:
        out.println("不是114514");
}
%>
```

### for语句

```jsp
<% for(int fontSize=1;fontSize<=5;fontSize++){ %>
    <font color="blue" size="<%= fontSize %>">
    	fontSize = <%= fontSize %>
	</font>
	<br/>
<%}%>
```

输入出结果如下

![](http://img.cairbin.top/img/202403312316765.png)

### while与do...while语句

while语句与do...while语句类似，这里仅演示while语句

```jsp
<%! int fontSize=1; %>
<% while(fontSize<=5){ %>
    <font color="blue" size="<%= fontSize %>">
    	fontSize = <%= fontSize %>
	</font>
	<br/>
    <% fontSize+=1; %>
<%}%>
```


![](http://img.cairbin.top/img/202403312316765.png)


## JSP字面量

JSP定义了如下几个字面量

- 布尔类型`boolean`
- 整型`int`
- 浮点型`float`
- 字符串`string`，单引号包裹或者双引号包裹
- Null

## JSP运算符

JSP支持Java所有逻辑和算术运算符，这里不详细介绍。