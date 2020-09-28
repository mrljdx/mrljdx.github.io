# AA工作原理及启动Activity的方式


> 译文地址：
> https://github.com/excilys/androidannotations/wiki/HowItWorks#starting-an-annotated-activity

## 概述
AndroidAnnotations是以一种非常简单的方式工作。它会使用[标准的Java注释处理工具](http://docs.oracle.com/javase/6/docs/technotes/guides/apt/index.html)自动添加一个额外的编译步骤来生成的"源代码"。
这里所谓的源代码是指，对于每个增强型类，例如每@EActivity注解的Activity，它就会生成一个具有相同名称而且末尾追加一个下划线的子类。

<!--more-->

例如下面的类：
```java
package com.some.company;
@EActivity
public class MyActivity extends Activity {
  // ...
}
```
会生成以下的子类，是同样的包名，但在另一个源文件夹下：
```java
package com.some.company;
public final class MyActivity_ extends MyActivity {
  // ...
}
```
这个子类通过在调用``super.onCreate(savedInstanceState)`` 之前重写一些方法（例如的``onCreate（）``）增加行为的活动。
这就是为什么你必须在``AndroidManifest.xml``中的Activity名称后面加下划线的原因。
```java
<activity android:name=".MyActivity_" />
```
## 启动一个被注解的Activity
在Android中，你平时启动一个Activity是这样的：
```java
startActivity(this, MyListActivity.class);
```
然而，使用AndroidAnnotations，启动一个真正的活动，必须要以MyListActivity_作为开始参数：
```java
startActivity(this, MyListActivity_.class);
```
## 意图生成器（Intent Builder）
我们提供了一个静态的帮手来启动一个生成的Activity：
```java
// Starting the activity
MyListActivity_.intent(context).start();

// Building an intent from the activity
Intent intent = MyListActivity_.intent(context).get();

// You can provide flags
MyListActivity_.intent(context).flags(FLAG_ACTIVITY_CLEAR_TOP).start();

// You can even provide extras defined with @Extra in the activity
MyListActivity_.intent(context).myDateExtra(someDate).start();
```
您也可以使用``startActivityForResult（）``相当于:
```java
MyListActivity_.intent(context).startForResult();
```
## 启动一个注解服务
在Android中，你平时启动服务是这样的：
```java
startService(this, MyService.class);
```
然而，在使用AndroidAnnotations后，启动一个服务是这样的：
```java
startService(this, MyService_.class);
```
## 意图生成器
我们提供了一个静态的帮手，让你开启一个服务：
```java
// Starting the service
MyService_.intent(context).start();

// Building an intent from the activity
Intent intent = MyService_.intent(context).build();

// You can provide flags
MyService_.intent(context).flags(Intent.FLAG_GRANT_READ_URI_PERMISSION).start();
```
## 是否有任何性能的影响？
简单的答案是否定的。因为AndroidAnnotations有一点点编译开销，但是生成的类都是很好的经典的Android代码，没有任何反射。没有启动时间，并没有运行时的性能影响。更多常见问题请看[FAQ](https://github.com/excilys/androidannotations/wiki/FAQ#perf-impact)

## 译者注
我们在使用Eclipse是，AndroidAnnotations会在编译阶段为每个注解类生成一个子类，而这些子类会保存在``.apt_generated`` 文件夹下，而在AndroidStudio中，生成的子类会保存在``app/build/generated/source/apt/debug/`` 目录下，release版本会生成到相应的release目录下。
