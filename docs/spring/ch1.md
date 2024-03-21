# 了解Spring

## 什么是Spring
Spirng是分层的JavaSE/EE全栈轻量级开源框架，以**控制反转IoC**和**面向切面编程AOP**为内核，使用基本的JavaBean来完成EJB的工作。

Spring框架采用分层架构，它的一些列功能被分为若干个模块。
![Spring结构](http://img.cairbin.top/img/a2e1edc343a78fe35fbc9f35041427f0.png)
上图中的红色背景模块为本课程涉及模块。

对于上述各个模块的功能，我并不想在此处多写，而是在接下来的代码中来体会。
（应该没有人刚开始学就想看这么冗长的文字吧，绝大多数人都是想快速构建项目，那些东西熟悉了再回过头来看）

## 创建Maven项目

我们打开**Eclipse**创建一个名称为test1的项目，流程如下：


![](http://img.cairbin.top/img/202403190122103.png)

![](http://img.cairbin.top/img/202403190124698.png)

这里我们使用**quickstart**来创建，刚进入这个界面的时候可能是空白的，稍等一会就好。

![](http://img.cairbin.top/img/202403190125935.png)

如果等待时间较长，还是空白，`Catalog`切换到`Internal`
![](http://img.cairbin.top/img/202403220039748.png)

接下来我们填写**Group Id**和**Artifact Id**，这里有必要说明下

前者一般为域名的反写，比如`com.xxx`一般表示某商业公司；而`org.xxx`一般表示某组织。后者一般为项目名称。

![](http://img.cairbin.top/img/202403190132511.png)

点击完**Finish**按钮需要等一会才能创建完成。当Console里出现下方提示的时候敲击回车继续创建。

![](http://img.cairbin.top/img/202403190135203.png)

接下来肯定是程序员的光荣传统——HelloWorld。

我们在左侧的**Package Explorer**中展开项目，并在`src/main/java`路径下看见了名称为`App`的类。

![](http://img.cairbin.top/img/202403190146532.png)

内容大致如下
```java
package top.cairbin.test1;

/**
 * Hello world!
 *
 */
public class App 
{
    public static void main( String[] args )
    {
        System.out.println( "Hello World!" );
    }
}

```


我们点击上方的**运行**按钮可以看到下方控制台中有输出

![](http://img.cairbin.top/img/202403190147783.png)



## 在项目中使用Spring

我们成功创建了一个**Maven项目**。这里你可能会有疑问，为什么必须是Maven项目而非普通的Java项目？

为了解决这个问题，首先需要了解什么是Maven：Maven是一个项目管理工具，它包含了一个对象模型。一组标准集合，一个依赖管理系统。和用来运行定义在生命周期阶段中插件目标和逻辑。

**简单来说，我们要使用Spring框架，但是手动来操作很麻烦，我们就借助Maven这个工具将Spring的包下载并引入到我们的项目里来。**


操作很简单，只需要在左侧的**Package Explorer**中找到`pom.xml`这个文件，在里面的`<dependencies></dependencies>`标签中添加如下内容即可：

```xml
<!-- https://mvnrepository.com/artifact/org.springframework/spring-context -->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>5.3.16</version>
</dependency>
```


![](http://img.cairbin.top/img/202403190205678.png)

`pom.xml`这个文件是给maven读取的，它除了包含了与你项目相关的一些信息外还负责解决依赖问题。

我们所添加的内容，就是引入`org.springframework`这个包的`spring-context`模块，对应的版本号为`5.3.16`。