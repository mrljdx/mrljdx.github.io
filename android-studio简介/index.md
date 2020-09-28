# AndroidStudio简介

Android Studio是Google官方基于IntelliJ IDEA开发的一款Android应用开发工具。这款工具提供以下的功能：

 - 基于Gradle的灵活编译系统
 - 能通过配置编译生成多个不同的apk文件
 - 强大的布局编辑器
 - 轻便的工具来捕获应用的性能、版本兼容性和一些其他的开发问题
 - ProGuard和app签名(可以直接编译安装不需要像eclipse一样打包签名)

<!--more-->
## Android Studio工程文件结构
### 1. Android项目视图
默认情况，AndroidStudio会以Android项目试图来显示你的工程。此试图显示项目的结构，能让你快速的找到项目的主要源文件。并且能帮助你直接使用Gradle来构建项目。
如图所示：
![图1.Android项目视图](http://7u2j5d.com1.z0.glb.clouddn.com/1.png)
 - **java/** 工程的源码模块
 - **manifests/** 工程的配置文件
 - **res/** 工程资源文件
 - **Gradle Scripts**  Gradle编译配置文件（以后再讲）

### 2. Eclipse vs AndroidStudio
转用Android Studio开发前，你需要明确一些事情：[链接地址](http://www.open-open.com/news/view/1b554f1)
AndroidStudio引入了Module的概念，这个和Eclipse的Project有点像。每一个Module需要有属于自己的Gradle build file（当你新建一个Module时会自动帮你生成的，当你导入一个Eclipse的项目时需自己创建）。这些Gradle文件包含了一些很重要的内容，比如所支持的安卓版本和项目依赖的东西，以及安卓项目中其它重要的数据。
和Eclipse上的一样，一些Modules可能是”Library Modules”，功能上与”Library projects”一样的。
### 3. 导入第三方jar库文件
![导入jar库文件](http://jbcdn2.b0.upaiyun.com/2014/03/eff7ecbcc1ed532e8f93e2fe95fb6981.png)
与Eclipse中遇到的一样，你会经常需要用到第三方开发的JAR文件。然而你现在需习惯将这些.jar依赖包加入到你的Gradle中。右 击“libs”目录下的.jar文件，然后选择“Add As Library”。这样你所选择的Jar文件将会自动地添加成Gradle的依赖包在你对应的Moule中。
### 4. Gradle 基础知识
新增的Gradle将会是你转到Android Studio上最大的障碍。下面有几个你需要知道的基础知识：
- 你的Android Studio项目将有一个关于整个项目的settings.gradle文件。
- settings.gradle文件包括项目中所有modules的引用，当你导入或者创建一个新的module时，这个文件会自动更新。
- 每一个Andorid Studio module会有自己的build.gradle文件。
- 如果一个Module向上依赖于另一个module，你需要添加这个依赖到所依赖部分的build.gradle文件上。
- 如果你的Module需要一个jar文件，这个jar必须列在Module的build.gradle文件中。
- 你可以在module的build.gradle文件上列出你要添加的远程依赖到你的项目中。
- 有时候，你需要人工修改这些gradle文件。

更多有关Gradle的内容可以在[这里](http://www.jayway.com/2013/02/26/using-gradle-for-building-android-applications/)找到。
### 5. Android项目生成的详细过程
![Build Img](http://7u2j5d.com1.z0.glb.clouddn.com/build.png)

