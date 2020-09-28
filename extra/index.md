# Extra

>原文地址：
>https://github.com/excilys/androidannotations/wiki/Extras#executing-code-after-extras-injection

## @Extra
@Extra注释一个从Activity中启动意图中获取正确值的字段，例如：
```java
@EActivity
public class MyActivity extends Activity {

  @Extra("myStringExtra")
  String myMessage;

  @Extra("myDateExtra")
  Date myDateExtraWithDefaultValue = new Date();

}
```
<!--more-->
如果你不给@Extra注释的字段值赋值，那么这个字段值将直接使用被使用这个字段名称。比如：
```java
@EActivity
public class MyActivity extends Activity {

  // The name of the extra will be "myMessage"
  @Extra
  String myMessage;
}
```
请注意,您可以使用意图构建器传递Extra的值。

	MyActivity_.intent().myMessage("hello").start() ;
 
## 处理onNewIntent()
AndroidAnnotations会覆写setIntent()方法,并当你调用setIntent()方法时会根据给定的意图自动设置extra的值。
这允许你从onNewIntent()方法中通过调用setIntent()方法自动注入extra的值。
```java
@EActivity
public class MyActivity extends Activity {

    @Extra("myStringExtra")
    String myMessage;

    @Override
    protected void onNewIntent(Intent intent) {
        setIntent(intent);
    }
}
```
##在注入extras之后执行代码
如果你需要执行注入extras之后的代码,你应该使用@AfterExtras注释一些方法。
```java
@EActivity
public class MyClass {

  @Extra
  String someExtra;

  @Extra
  int anotherExtra;


  @AfterExtras
  public void doSomethingAfterExtrasInjection() {
    // someExtra and anotherExtra are set to the value contained in the incoming intent
    // if an intent does not contain one of the extra values the field remains unchanged
  }

}
```
## 警告
如果父类和子类有@AfterViews,@AfterInject或@AfterExtras名称相同的带注释的方法,生成的代码将会有BUG。查看更多细节[问题# 591](https://github.com/excilys/androidannotations/issues/591)。
同时，当我们调用那些@AfterViews、@AfterInject、@AfterExtras注解的方法时会有一个调用顺序，但是如果@AfterView、@AfterInject、@AfterExtras 注解的方法是同名的，那就不能确保这几个方法的调用顺序了 ，请看( [issue #810](https://github.com/excilys/androidannotations/issues/810))。
关于这些注解的方法调用顺序，你可以看我的这篇博客译文：[@AfterXXX 调用顺序](http://blog.csdn.net/mrljdx/article/details/44891189)。
