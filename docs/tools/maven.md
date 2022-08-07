# maven的使用总结
## 前言
最近帮助同事解决项目依赖的问题，发现自己对maven组织的依赖关系，并不是很清晰，对一些配置也不甚了解，在此做一总结。

## maven是什么
虽然平日里编译，打包，mvn compile，mvn package，敲的很熟练，但仅仅把maven当成一个打包的工具，是不能很好的帮助我们理解项目构成的。Apache Maven is a software project management and comprehension tool.上述是maven官方的定义，定位为项目管理工具。既然是项目管理，必然涉及到项目的生命周期的各个阶段。从开发，编译，测试，发布，这当中的每个阶段maven都是可以参与其中的。

## pom
pom（project object model ）项目对象模型，是maven的一个核心概念。既然要对项目进行管理，那么需要一个对项目抽象来进行描述，这就是pom。pom.xml文件就是项目模型的xml描述。maven通过读取pom文件，来处理对应项目的各阶段工作。

## phrase 和 goal
上面说到，一个项目周期中，是包含很多阶段的，这些阶段就是phrase；而每个阶段，具体执行哪些操作，具体的操作就是goal。

以下是maven中的一些phrase：

**validate**: validate the project is correct and all necessary information is available
**compile**: compile the source code of the project
**test**: test the compiled source code using a suitable unit testing framework. These tests should not require the code be packaged or deployed
**package**: take the compiled code and package it in its distributable format, such as a JAR.
**integration-test**: process and deploy the package if necessary into an environment where integration tests can be run
**verify**: run any checks to verify the package is valid and meets quality criteria
**install**: install the package into the local repository, for use as a dependency in other projects locally
**deploy**: done in an integration or release environment, copies the final package to the remote repository for sharing with other developers and projects.
**clean**: cleans up artifacts created by prior builds
**site**: generates site documentation for this project

![](https://images.xiaozhuanlan.com/photo/2019/bd3b6db0fbc5073834d7f4916ba83514.png)
这是maven官方的例子，在build标签中，绑定了phrase test 和 goal query，也就是说，在maven构建时，当执行到test阶段，会执行query定义的具体操作。

![](https://images.xiaozhuanlan.com/photo/2019/04c116884c92a9a6efa44fee94fc0724.png)
这是官方提供的phrase的默认绑定。

## repository

再来看下另一个概念**仓库**。我们开发项目，多半不可能都是自己手写，或多或少我们都会依赖第三方的框架或库。而仓库就是存储这些额外依赖的地方。以下有几种不同的仓库。
**中央仓库**： maven官方定义的仓库，默认需要下载依赖会去中央仓库下载
**远程仓库** ： 一般为我们自己搭建的nexus私服，主要用于管理自己的依赖和加速依赖下载。
**本地仓库** ： 这是我们本地磁盘的一个目录，从远程和中央仓库下载的依赖会保存一份到本地仓库，重新构建项目时，本地已有的依赖不用再去远程下载。也可以用于离线环境。
**镜像**： 对中央仓库的镜像仓库，由于某种不可抗的原因，我们直接从中央仓库下载依赖会很慢，在我们便于访问的地方，有一些企业提供了镜像站点，加速我们获取依赖。我们常用的有华为和阿里的镜像仓库。

## 聚合和继承

一个庞大的项目，拥有很多子模块。如果我们每个模块独立打包，操作无疑是繁琐的。通过聚合的功能，可以为子模块增加一个父pom，从而使maven一开始解析父pom，再根据module获取子模块信息。这样只需要在父pom测试调用相关命令，即可把整个项目打包。
![](https://images.xiaozhuanlan.com/photo/2019/8356932c46b5c9f262833a81988ccfd3.png)
在父pom中定义modules，即可统一一个入口。

同样的，为了使子模块可以使用同样版本的依赖，免除各自为政的问题，我们可以把共用的一些依赖放到父pom中，并用DependencyManagement标签包起来，这样在子pom中依赖可以不加版本号，使用和父pom相同的版本。
![](https://images.xiaozhuanlan.com/photo/2019/6c437a9a99bbeee0682d7936b6e8f732.png)

## 应用
上述简单介绍了maven的相关概念和执行流程。下面我们看看一些常见的使用

### setting
maven 的配置文件在conf中setting.xml中
![](https://images.xiaozhuanlan.com/photo/2019/e26dfb2f05787510a31da650be0556a2.png)
重点说一下servers
![](https://images.xiaozhuanlan.com/photo/2019/2f8494a8eda887d206536ca878d16fcf.png)
server是pom中对应的repository标签和 distributionManagement 标签的服务器信息，但对于服务器的用户名密码及权限配置不应该和pom文件分发，而应该作为一个个人应用的配置，保留在本地。

### maven的命令
 mvn [options] [goal(s)] [phase(s)]
以这种形式使用:
例如 mvn clean install

### 创建新项目
>  mvn archetype:generate -DgroupId=com.mycompany.app -DartifactId=my-app -DarchetypeArtifactId=maven-archetype-quickstart -DarchetypeVersion=1.4 -DinteractiveMode=false

### 打印依赖树
> mvn dependency:tree

### 打包安装到本地
>mvn clean install

### 发布到远程服务器
>mvn clean deploy

### 解决jar包冲突
一般可能由于不同模块引入了相同依赖的不同版本引起。
可以先使用dependency:tree 看一下整体依赖，或者通过idea等工具分析冲突的版本，然后在dependency中exclude掉冲突的版本


## 参考资料
[https://www.yiibai.com/maven/](https://www.yiibai.com/maven/)
[http://maven.apache.org/](http://maven.apache.org/)
[https://www.jianshu.com/p/b07e7dbd8e4c](https://www.jianshu.com/p/b07e7dbd8e4c)