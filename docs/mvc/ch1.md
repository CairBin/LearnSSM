# MVC模式与三层架构


## 什么是MVC模式

**MVC模式**代表**Model-View-Controller（模型-视图-控制器）模式**。这种应用模式用于应用程序的分层开发。

- `Model`代表存取数据的对象，它自身可带有逻辑，数据变化时更新`Controller`。
- `View`代表`Model`包含数据的可视化。
- `Controller`作用在`View`和`Model`之上，控制数据流向`Model`，并在数据变化的时候更新`View`，使`Model`与`View`分离。

三者的关系如下图所示：

![MVC](http://img.cairbin.top/img/202403242337531.png)


**最典型的MVC就是JSP+Servlet+JavaBean的模式。**


## 三层架构

三层架构中的“三层”分别指：

- 数据访问层（DAL，Data Access Layer）
- 业务逻辑层（BLL，Business Logic Layer）
- 表示层（USL，User Show Layer）

### DAL

DAL也被称为持久层，位于三层最下层，用于对数据进行处理。该层方法一般为“原子性”的。

在Java程序中，DAL一般都在`dao`包中；有时也将实体类放在Dao包中，Mapper的方法放在Mapper包中。

### BLL

BBL起到数据交换承上启下的作用，对于业务逻辑进行封装，但与DAL不同的是，这里的方法一般不是原子的，而是包含业务逻辑。例如删除，就要先进行逻辑判断或数据校验再进行DAL中的删除。

BLL一般写在`service`包中。


### USL

位于最上层，负责与用户进行交互，为用户提供交互界面。其中USL分为前台和后台，前台指用户能够看到的界面，后台指来调用BLL的代码。


## MVC与三层架构的关系

**MVC与三层架构没有任何关系。**前者是用来解决B/S程序（即浏览器/服务器程序）的耦合关系的，而后者适用于任何技术。二者的设计角度不一样。

之所以会混淆，是因为二者经常一块用，USL可以用View（USL前台）和Controller（USL后台）来实现，而BLL和DAL可以由Model来实现。