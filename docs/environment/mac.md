# MacOS环境搭建

## 安装Homebrew

Homebrew是一个包管理器，我们可以通过它来安装许多软件

首先打开[Homebrew中文官网(brew.sh/zh-cn)](https://brew.sh/zh-cn/)

![Homebrew官网](http://img.cairbin.top/img/202403182345561.png)

如图所示，复制下面那行命令到你的Macbook终端
```shell
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

这个可能会由于网络导致失败，可以替换成以下命令
```shell
/bin/bash -c "$(curl -fsSL https://gitee.com/cunkai/HomebrewCN/raw/master/Homebrew.sh)"
```

完成后使用以下命令检验是否安装成功

```shell
brew --version
```

如果出现下图则安装成功

![brew version](http://img.cairbin.top/img/202403182350228.png)

我在写这篇文章的时候是这个版本，你的版本号可能比我更高，这一般没啥问题。

接下来还是因为网络问题我们更换为国内的下载源，同样在终端中执行以下命令

```shell
git -C "$(brew --repo)" remote set-url origin https://mirrors.aliyun.com/homebrew/brew.git
```

然后更换homebrew-bottles

```shell
echo 'export HOMEBREW_BOTTLE_DOMAIN=https://mirrors.aliyun.com/homebrew/homebrew-bottles' >> ~/.bash_profile
```

让配置文件生效
```shell
source ~/.bash_profile
```
如果是zsh就写入zsh的配置文件中（如果你不懂这是啥意思，而且没有改过Mac的终端，可以忽略这句话）

最后来更新下软件包
```shell
brew update
```
## 安装JDK
我选择安装并使用**OpenJDK**，它是JDK的开源版本。

我们借助刚才安装好的**homebrew**包管理器来安装它，打开终端输入下面的命令

```shell
brew install openjdk
```

你可以指定版本，例如安装openjdk17可以使用以下命令

```shell
brew install openjdk@17
```

接下来我们会用到**vim**这个终端文件编辑器
如果vim不存在我们可以用以下命令安装它

```shell
brew install vim
```

我们打开配置文件

```shell
vim ~/.bash_profile
```
这里要根据是`zsh`还是`bash`，一般默认`bash`（你不懂的话而去终端没有修改过的话忽略这句话）

这里来讲下vim操作，点击键盘上的`i`来进入编辑模式。
在文件最下方写入一行代码

```
export PATH="/opt/homebrew/opt/openjdk/bin:$PATH"
```

![](http://img.cairbin.top/img/202403190032851.png)

然后按键盘上的`esc`退出编辑模式，然后输入`:wq`保存并退出。

让配置文件生效
```shell
source ~/.bash_profile
```

在终端中检查Java版本

```shell
java --version
```

如果出现以下提示（版本号不必与我一致），则表示安装成功
![java version](http://img.cairbin.top/img/202403190056005.png)


## 安装Maven
接下来我们来安装**Maven**，同理还是使用**homebrew**。

```shell
brew install maven
```

打开配置文件，配置环境变量

```shell
vim ~/.bash_profile
```
这里要根据是`zsh`还是`bash`，一般默认`bash`（你不懂的话而去终端没有修改过的话忽略这句话）

文件最下方追加一行代码

```
export PATH="/opt/homebrew/opt/maven/bin:$PATH"
```

让配置文件生效

```shell
source ~/.bash_profile
```

最后检查maven是否安装成功，出现版本信息则成功

```shell
mvn -version
```

## 安装Eclipse IDE

打开[Eclipse官网的下载页面](https://www.eclipse.org/downloads/)

![Eclipse Downloads](http://img.cairbin.top/img/202403182356305.png)

这里**Download**按钮下有两个选项，如果你的Macbook是老款的Intel处理器就选上面的**x86_64**，如果是苹果的芯片比如M1、M2等，就选下面的**AArch64**。

下载后会有个以**.dmg**后缀结尾的文件，打开它后如下图

![dmg](http://img.cairbin.top/img/202403190000278.png)

将右边的**Eclipse Installer**拖入到左面的**Applications**中去，然后在dock栏的起动台中打开它。

![启动台](http://img.cairbin.top/img/202403190002953.png)

打开后选择第二项`Eclipse IDE for Enterprise Java and Web Developers`，注意千万别选错了，我们要开发WEB项目

![](http://img.cairbin.top/img/202403250133368.png)


如图所示有两个输入框，上面那个是你的JDK路径，下面那个是你的安装目录。

如果没有出现如图所示的路径，那大概率是因为你JDK没有配置好，请去检查JDK配置。

**请将安装目录调整至/Applications**，否则你在启动台中看不到它！

**请将安装目录调整至/Applications**，否则你在启动台中看不到它！

**请将安装目录调整至/Applications**，否则你在启动台中看不到它！

最后点击**Install**按钮进行安装即可。

![](http://img.cairbin.top/img/202403250135403.png)

安装完成并首次启动**Eclipse**会有一个工作目录的配置，默认即可，点击**Launch**按钮。

![workpath](http://img.cairbin.top/img/202403190019604.png)

**这里注意，工作目录不能已经存在，否则会报错！如果存在请到目录下删除对应文件夹！**

完成后，主界面就应该如下图所示。

![Eclipse Home](http://img.cairbin.top/img/202403190022169.png)

### 安装Tomcat

**这里一定要指定Tomcat9版本，用10及以上的话会有大坑**

```shell
brew install tomcat@9
```

完成后会提示你配置环境变量

```shell
echo 'export PATH="/opt/homebrew/opt/tomcat@9/bin:$PATH"' >> ~/.bash_profile
```

让配置文件生效

```shell
source ~/.bash_profile
```

检查是否安装成功

```shell
catalina -h
```

我们可以使用以下命令来启动tomcat服务

```shell
brew services start tomcat@9
```

同理也可以关闭它

```shell
brew services stop tomcat@9
```

检查服务状态

```shell
brew services info tomcat@9
```

当服务为开启状态的时候我们**浏览器**访问`http://localhost:8080`大概如下图

![tomcat test](http://img.cairbin.top/img/202403190110134.png)

我这里的图片为了演示还是Tomcat10的

## 安装MySQL

安装MySQL数据库

```shell
brew install mysql
```
你可以指定版本，推荐与教程版本一致，版本较高可能会有问题
```shell
brew install mysql@5.7
```

配置环境变量
```shell
echo 'export PATH="/opt/homebrew/opt/mysql@5.7/bin:$PATH"' >> ~/.bash_profile
```

让配置文件生效

```shell
source ~/.bash_profile
```

启动mysql服务

```shell
brew services start mysql
```

如果指定5.7版本可以用下方命令
```shell
brew services start mysql@5.7
```

使用`root`用户登陆，默认无密码，默认端口`3306`

```shell
mysql -uroot
```


## 配置Tomcat

![](http://img.cairbin.top/img/202403250126199.png)

如图所示，找到Eclipse工具栏 -> Settings -> 左侧选项中的Server -> Runtime Environment

![](http://img.cairbin.top/img/202403250147680.png)

单击Add按钮，然后在“Apache”里根据你的Tomcat版本来选择（你之前访问过Tomcat的测试页面，里面就有版本号）

![](http://img.cairbin.top/img/202403280144342.png)

指定Tomcat路径，如果不知道，在终端中执行`catalina -h`命令就行

![](http://img.cairbin.top/img/202403280141047.png)

![](http://img.cairbin.top/img/202403280145008.png)

最后保存并退出。

**在执行SpringMVC项目的时候，需要先关闭Tomcat！！！交给Eclipse去打开！**