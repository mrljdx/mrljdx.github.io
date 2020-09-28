# @AfterXXX调用顺序

>原文地址：
>https://github.com/excilys/androidannotations/wiki/%40AfterXXX-call-order

## 用@AfterXXX标注的方法调用次序
同一个类中的同一个注释的方法调用顺序不能保证，你不应该依赖于它。如果你想多种方法按照可靠的顺序被被调用，你应该只创建创建一个注解的方法，并用它来调用它们所需顺序的其他方法。
<!--more-->
```java
@AfterInject
protected void callInOrder() {
    methodA();
    methodB();
    methodC();
}

private void methodA() {
    // your code
}

private void methodB() {
    // your code
}

private void methodC() {
    // your code
}
```
如果你要处理的父/子类，你可以试试这个。
```java
/**
*Parent
*/
@EActivity
public class Parent extends Activity {

    @AfterViews
    protected void afterViews() {
        // do here something related to parent
    }
}
```
```java
/**
*Child
*/
@EActivity
public class Child extends Parent {

    @Override // no @AfterViews !
    protected void afterViews() {
        super.afterViews(); // does something related to parent
        // do here something related to child
    }
}
```
## 初始化调用顺序（初始化后）
1. [@AfterExtras](https://github.com/excilys/androidannotations/wiki/Extras#executing-code-after-extras-injection)
2. [@AfterInject](https://github.com/excilys/androidannotations/wiki/Enhance-custom-classes#executing-code-after-dependency-injection)
3. [@AfterViews](https://github.com/excilys/androidannotations/wiki/Injecting-Views)

**注意：**当``@AfterExtras``注释方法被调用时，除和视图无关的注入变量都会被初始化并赋值。因此在标注``@Bean`` 的类中使用变量值是安全的。一些视图(view)只能在``@AfterViews`` 注释方法被调用是才能安全使用。

## 后续调用
+ ``@AfterExtras``方法会在每个新的Intent Activity启动时调用。
+ ``@AfterViews``将在每次的setContentView()方法被执行后被调用。 （Activity专用）
