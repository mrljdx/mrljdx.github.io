# AndroidStudio参考资料整理(持续更新)


这篇文章主要记录一些关于AndroidStudio的参考资料，文章内容会不定期更新。主要记录博主在使用AndroidStudio的过程中碰到的一些问题以及这些解决这些问题的博客链接。
<!--more-->
<hr>

## Gradle Android插件用户指南翻译
通过阅读Google官方的Gradle Android插件用户指南，可以让你在写build.gradle的时候不至于不知道这里面的语法意思。
>1. 原文地址:[Gradle Plugin User Guide](http://tools.android.com/tech-docs/new-build-system/user-guide#TOC-Introduction)
>2. 译文地址:[Gradle Android插件用户指南](http://avatarqing.gitbooks.io/gradlepluginuserguidechineseverision/content/)

## AndroidStudio系列教程
作者写的一系列教程干货颇多，从"AndroidStudio的下载于安装"到"使用Gradle多渠道打包"都讲的比较清楚易懂。
>1. [AndroidStudio系列教程(一)下载与安装](http://stormzhang.com/devtools/2014/11/25/android-studio-tutorial1)
>2. [AndroidStudio系列教程(二)基本设置与运行](http://stormzhang.com/devtools/2014/11/28/android-studio-tutorial2)
>3. [AndroidStudio系列教程(三)快捷键](http://stormzhang.com/devtools/2014/12/09/android-studio-tutorial3)
>4. [AndroidStudio系列教程(四)Gradle基础](http://stormzhang.com/devtools/2014/12/18/android-studio-tutorial4)
>5. [AndroidStudio系列教程(五)Gradle命令详解于导入第三方包](http://stormzhang.com/devtools/2015/01/05/android-studio-tutorial5)
>6. [AndroidStudio系列教程(六)Gradle多渠道打包](http://stormzhang.com/devtools/2015/01/15/android-studio-tutorial6)

## Android Studio关于SVN的相关配置简介
虽然个人喜欢Git版本控制，但是在公司工作还是需要使用SVN作为版本控制使用。
> [Android Studio关于SVN的相关配置简介](http://www.it165.net/pro/html/201404/11412.html)

## Build Android Project with Gradle
这篇文章里介绍的内容和"AndroidStudio系列教程(六)"的内容有部分重合，但是如果你一个项目想要生成不同的APK包，有不同的包名，或者不同的资源，那么这就是看这篇文章的时候了。(PS:不常用，除非你上传应用到市场的时候碰到重名的包)
> 1. [Android Studio 混合多渠道打包](http://forum.android-studio.org/forum.php?mod=viewthread&tid=66)
> 2. [Build Android Project with Gradle](http://forum.android-studio.org/forum.php?mod=viewthread&tid=38&extra=)

## 使用Gradle发布Android开源项目到JCenter
喜欢做些开源项目的朋友，相信有不少人都希望能把自己的项目发布到公共的中央仓库，如maven中央仓库，以供别人方便地集成使用。而使用了Android Studio的同学，应该也对gradle和jcenter印象深刻，不少开源库都是发布到这里的。这一篇就主要来介绍一下，如何使用Gradle发布到jcenter。
> [使用Gradle发布Android开源项目到JCenter](http://blog.csdn.net/maosidiaoxian/article/details/43148643)

## Android studio 代码混淆和破解apk
这篇文章讲解了代码混淆的一些内容，提供了一个注释清晰的混淆模版。这个值得一看，其次，要注意一下，在AndroidStudio中，混淆是由两个文件`proguard-android.txt`(在`androidsdk/tools/proguard/proguard-android.txt`该文件已经包含了基本的混淆声明)以及`proguard.cfg`(在`app/proguard_rules.pro`声明一些第三方依赖的一些混淆规则)决定的。
> [Android studio 代码混淆和破解apk](http://my.oschina.net/aibenben/blog/371889)


