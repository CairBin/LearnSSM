# 工厂模式、核心容器与Spring Bean

## 工厂模式
**工厂模式**是Java中常用的一种设计模式，这种类型的设计模式属于创建型模式。说白了在代码层面就是取消了new的使用。

工厂模式有三种：

- 简单工厂模式
- 工厂方法模式
- 抽象工厂模式

举个例子，我们去买手机，假设手机品牌有两种，分别是Xphone和Luwei，你显然不用关心手机是怎么生产的，手机零件怎么组装的，这都是工厂干的活。下面我们以这个例子来讲解三种工厂模式。

### 简单工厂模式

**简单工厂模式**示意图如下：

![简单工厂模式示意图](http://img.cairbin.top/img/202403192017957.png)

我们用Java代码来表示，首先需要定义一个接口，来表示手机(Phone)，不妨声明一个方法，用来获取品牌名称。

```java
public interface Phone{
    String getBrand();
}
```

接下来我们该写这两种品牌的手机类，它们都实现`Phone`这个接口。

`Xphone`手机类如下
```java
public class Xphone implements Phone{
    @Override
    public String getBrand(){
        return "Xphone";
    }
}
```

同理`Luwei`手机类如下
```java
public class Luwei implements Phone{
    @Override
    public String getBrand(){
        return "Luwei";
    }
}
```

我们需要一个手机工厂`PhoneFactory`

```java
public class PhoneFactory{
    public static Phone getPhone(String phoneBrand){
        if(phoneBrand.equals("Xphone")){
            return new Xphone();
        }

        if(phoneBrand.equals("Luwei")){
            return new Luwei();
        }
    }
}
```

作为消费者的你，想要获取手机只需要通过工厂即可

```java
public class Customer{
    public static void main(String[] args){
        //获取手机实例对象
        Phone xphone = PhoneFactory.getPhone("Xphone");
        System.out.println(xphone.getBrand());
    }
}
```

但是细心观察你可能会发现，我们目前只有两个手机品牌，但是当品牌增多的时候，工厂内部的代码也需要修改，我们不得不面临一个麻烦——内部代码也会增加。这违反了设计模式的一个原则，即**对扩展开放，对修改关闭**。

### 工厂方法模式

为了解决简单工厂模式的问题，聪明的你可能会想到，干脆将工厂也变为抽象的接口，我们让每个手机厂商去实现它们各自的工厂不久行啦！没错，这正是**工厂方法模式**。

![工厂方法模式示意图](http://img.cairbin.top/img/202403192039953.png)

我们在上文的基础上将`PhoneFactory`从类变为接口

```java
public interface PhoneFactory{
    Phone getPhone();
}
```

然后为`Xphone`类实现它的工厂`XphoneFactory`

```java
public class XphoneFactory implements PhoneFactory{
    @Override
    public Phone getPhone(){
        return new Xphone();
    }
}
```

同理为`Luwei`手机类也实现它的工厂
```java
public class LuweiFactory implements PhoneFactory{
    @Override
    public Phone getPhone(){
        return new Luwei();
    }
}
```


而对于消费者我们可以这样调用
```java
public class Customer{
    public static void main(String[] args){
        Phone xphone = (new XphoneFactory).getPhone();
        System.out.println(xphone.getBrand());
    }
}
```

这样我们就不必去更改Factory的代码就能增加品牌了，但工厂方法模式会带来另外一个问题，当我们增加品牌的时候就需要增加工厂，而且我们如果新增一款产品，那么每个工厂就必须增加相应的方法，这大大降低了代码的可维护性。

### 抽象工厂模式

**抽象工厂模式**相对来讲比较复杂，在抽象工厂模式中，接口是负责创建一个相关对象的工厂，不需要显式指定它们的类。每个生成的工厂都能按照工厂模式提供对象。

假设我们前面提到的两个品牌都新增了一样产品——个人电脑。

我们来定义手机接口和电脑接口。

```java
public interface Phone{
    String getBrand();
}
```

```java
public interface Pc{
    String getBrand();
}
```

然后两家供应商都各自实现这两个产品

对于`Luwei`品牌

```java
public class LuweiPhone implements Phone{
    @Override
    public String getBrand(){
        return "Phone_Luwei";
    }
}
```

```java
public class LuweiPc implements Pc{
    @Override
    public String getBrand(){
        return "PC_Luwei";
    }
}
```


对于`Xphone`品牌

```java
public class Xphone implements Phone{
    @Override
    public String getBrand(){
        return "Phone_Xphone";
    }
}
```

```java
public class XphonePc implements Pc{
    @Override
    public String getBrand(){
        return "PC_Xphone";
    }
}
```

接下来就是工厂了，这两个品牌必须提供工厂来实现这两个产品。

我们定义抽象的工厂，即工厂接口`IFactory`。
```java
public interface IFactory{
    Phone getPhone();
    Pc getPc();
}
```

各自实现这个接口

```java
public class LuweiFactory implements IFactory{
    @Override
    public Phone getPhone(){
        return new LuweiPhone();
    }

    @Override
    public Pc getPc(){
        return new LuweiPc();
    }
}
```

```java
public class XphoneFactory implements IFactory{
    @Override
    public Phone getPhone(){
        return new Xphone();
    }

    @Override
    public Pc getPc(){
        return new XphonePc();
    }
}
```

我们为了统一它们的工厂，还需要一个**超级工厂**，即工厂的工厂，者有些类似于供应商与代理商之间的关系，供应商有自己的品牌和工厂生产自己的产品，然后交给代理商售卖给客户，同时代理商可以代理多家供应商的产品。

为了方便，我们直接在工厂接口里定义一个静态方法来完成这个“代理商”

```java
public interface IFactory{
    public static IFactory createFactory(String factoryName){
        if(factoryName.equals("Xphone")){
            return new XphoneFactory();
        }

        if(factoryName.equals("Luwei")){
            return new LuweiFactory();
        }
    }

    Phone getPhone();
    Pc getPc();
}
```

这样我们就完成了多个品牌多个产品统一对客户的供应。

当然这个模型还可以更复杂一些，比如每个品牌每个商品都有自己的工厂，然后这些品牌创建自己商品工厂的工厂来对外，与客户之间还有一个超级工厂来创建大工厂，不过我们仅是演示原理没必要这么麻烦。


## Spring核心容器

Spring容器会负责控制程序之间的关系，而不是由程序代码直接控制。

我们在上一节中，我们提到了IoC，而Spring的Ioc依赖于Spring的核心容器。

框架为我们提供了两种核心容器——`BeanFactory`和`ApplicationContext`。

前者位于`spring-beans`模块中，后者位于`spring-context`模块。（我们曾在`pom.xml`里引入过后者）

`BeanFactory`的`Bean`都是懒加载的方式，也就是说只有你去调用的时候才被实例化，而`ApplicationContext`容器启动时`Bean`就被实例化了。如果不是性能要求特别苛刻或者有其它限制，一般会用后者。


### BeanFactory与FactoryBean

看到`BeanFactory`与`FactoryBean`，你是不是就明白我们刚才为什么要花这么大的篇幅来讲解**工厂模式**了。

`BeanFactory`的重要程度我们用框架源代码中的一句话来讲——**IoC的根接口！！！**

但是我们只是个简单版本的教程，`BeanFactory`创建`Bean`的流程以及生命周期非常复杂，我们篇幅有限，而且并非是框架的文档，所以这里仅会简单描述一下它。

对于`BeanFactory`的主要作用就是：处理的`BeanDefinition`经过`BeanFactory`来生成`Bean`。

![BeanFactory作用](http://img.cairbin.top/img/202403192229434.png)

`BeanFactory`是一个大工厂，IoC的根基，可以生产各种`Bean`；而`FactoryBean`是一个小工厂，自己也是一个`Bean`，可以生产其它`Bean`。

看不懂没关系，我们来了解下`BeanFactory`怎么用。

还记得我们前两章创建的`top.cairbin.test1`项目吗，现在打开它的`App.java`文件，就是主类所在的文件，我们目前内容如下：

```java
package top.cairbin.test1;

import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;


public class App 
{
    public static void main( String[] args )
    {
    	ApplicationContext applicationContext = new ClassPathXmlApplicationContext("AppCtx.xml");
        User user = (User)applicationContext.getBean("user");
        user.testSay();
    }
}
```

可以看到我们是用的`ApplicationContext`来加载的`AppCtx.xml`文件，现在我们可以使用`BeanFactory`来加载它（不过我们一般不这么做），不要忘记导入相应的包。

```java
BeanFactory beanFactory = new BeanFactory(new FileSystemResource("这里写AppCtx.xml的绝对路径"));
```

### ApplicationContext

`BeanFactory`相对`ApplicationContext`比较低层，后者是前者的一个子接口，在开发中我们经常用后者。

获取`ApplicationContext`的方法这里举两个例子，分别是`ClassPathXmlApplicationContext`创建和`FileSystemXmlApplicationContext`创建。

前者形式如下
```java
ApplicationContext app = new ClassPathXmlApplicationContext(String configLocation);
```
也就是说`ClassPathXmlApplicationContext`是从类路径来找XML的配置文件。

如果你想要使用绝对路径来寻找可以使用`FileSystemXmlApplicationContext`

```java
ApplicationContext app = new FileSystemXmlApplicationContext(String configLocation);
```

什么，你问我啥是**相对路径**和**绝对路径**？前者是相对于你当前位置的；后者是目标文件在文件系统下真实路径，比如Windows中以盘符开始（例如`C:\\`），在Mac/Linux下以`/`开始。

## Spring Bean

Bean是被实例化的、组装的以及被Spring容器接管的Java对象。听起来很费解，但是没关系我们往下看。

我们在编写`AppCtx.xml`文件的时候就注意到，根元素为`<beans></beans>`，其中又包含了若干个子元素`<bean>`，这些子元素通常携带一些属性来帮助Spring实现依赖注入。

### Bean的实例化

Bean的实例化方法大致有三种：

- 构造器实例化（最常用）
- 静态工厂实例化
- 实例工厂实例化

#### 构造器实例化

构造器实例化需要类中拥有一个**默认构造方法**来实例化Bean，实际上你如果不写构造方法也会有一个默认的构造方法。

你只要创建符合条件的类，然后在`AppCtx.xml`文件里配置下就行了。我们之前章节就是使用构造器实例化的，所以这里不给出代码。

#### 静态工厂实例化

我们要自己创建一个工厂，并实现一个用于实例化的静态方法，假设我们的Bean对应类名为`MyBean`

```java
public class MyBeanStaticFactory{
    public static MyBean createBean(){
        return new MyBean();
    }
}
```

我们需要在`AppCtx.xml`的`<beans></beans>`里添加

```xml
<bean id="myBean" class="top.cairbin.test1.MyBean" factory-method="createBean"/>
```
这个标签的`factory-method`属性指定了工厂实例化Bean的静态方法。

#### 实例工厂实例化

实例工厂实例化Bean使用的是非静态方法，即**没有**`static`关键字修饰的方法。

```java
public class MyBeanFactory{
    public MyBean createBean(){
        return new MyBean();
    }
}
```

只不过这种情况下，配置文件的标签要配置两个属性，除了指定工厂的方法外还要通过`factory-bean`指定实例化后的工厂。

```xml
<bean id="myBean" class="top.cairbin.test1.MyBean" factory-bean="myBeanFactory" factory-method="createBean"/>
```

### Bean的属性

我们来介绍下之前涉及的几个属性：

- `id`：Bean的唯一标识符。
- `name`：该属性可以为Bean指定多个名称，用英文逗号`,`隔开。
- `class`：指定Bean对应的类，它必须是一个完整的类名，从包名一直到类。
- `scope`：用来设置Bean的作用域，至于作用域有哪些我们之后讲。

### Bean的作用域与生命周期

Bean的作用域用下图来描述

![Bean的作用域](http://img.cairbin.top/img/202403192232454.png)

至于Bean的生命周期，可讲解的点太多了，这里仅放一张部分的示意图

![Bean声明周期](http://img.cairbin.top/img/202403192241085.png)

### Bean的装配方式

Bean的装配方式对应我们前面的Spring实现DI的几种方式。Spring Bean的方式有以下几种：

- 基于XML装配
- 基于注解装配
- 自动装配

看完这里你就应该明白IoC、DI、工厂、Bean以及Bean的装配它们之间的关系了。

#### 基于XML装配

**基于XML装配**主要常用于两种注入方式——**Setter注入**和**构造器注入**。

Setter注入要求有Setter和**一个无参的构造方法**，在XML配置文件中用`<property>`来为每个属性注入值。

构造器注入**必须提供有参数的构造方法**，在XML配置文件中使用`<constructor-arg>`来为参数注入值。

#### 基于注解的装配

用于装配的注解主要有以下几种

![](http://img.cairbin.top/img/202403192309060.png)


#### 自动装配

自动装配要求属性有对应的Setter方法，在XML文件里进行属性设置，实际上是在`<bean>`元素标签上添加属性`autowire`，其值可以如下。
![](http://img.cairbin.top/img/202403192327045.png)


### @Autowired注解与@Resource注解

`@Resource`的作用相当于`@Autowired`，只不过`@Autowired`按`byType`自动注入，而`@Resource`默认按 `byName`自动注入。

Spring将`@Resource`的`name`属性解析为`bean`的名字，而`type`属性则解析为bean的类型。如果既不指定`name`也不指定`type`属性，这时将通过反射机制使用`byName`自动注入策略。

另外需要注意`@Resource`这个注解不属于Spring，它是属于J2EE的。

## 自动扫描与装配的关系

自动扫描主要用到了后面两种装配方式，按照基于注解的装配方式将相应的注解转换成Bean纳入Spring容器后，会继续扫描`@Resource`和`@Autowired`注解，然后按自动装配的方式进行关系建立。

然而后两种装配方式本身就是对XML的一种自动化处理，可能借助了Java的某些机制，例如反射，但实际上这几种装配方式在根本上还是一样的，不要将它们割裂来看。