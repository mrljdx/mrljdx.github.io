# @EBean增强自定义类

>原文地址：
>https://github.com/excilys/androidannotations/wiki/Enhance-custom-classes

## 加强自定义类
您可以在一个不是一个标准的Android组件（如Activity，Service）的类使用注解@EBean。
你只需要使用@EBean注释这个类即可：
```java
@EBean
public class MyClass {

}
```
<!--more-->
>一个@EBean注解类必须只有一个构造函数。此构造必须不含参数，或者作为AndroidAnnotations2.7，它在有些语境中的可以有一个参数。

## 注射增强类
在另一个增强类或增强的Android组件使用增强类，则可以使用``@Bean``：
```java
@EBean
public class MyOtherClass {

  @Bean
  MyClass myClass;

}
```
请注意链的依赖关系：
```java
@EActivity
public class MyActivity extends Activity {

  @Bean
  MyOtherClass myOtherClass;

}
```
> 在使用``@Bean``时你总可以获得该类的一个新实例，除非这个类被声明为单例类（a Singleton）后面会讲到。

值得注意的是，生成的子类被加上了final限定，这意味着你不能扩展生成的子类。但是，您可以扩展原有的类和生成的代码将在子类中也同时发生扩展。这适用于每一个注解。
比如下面这个被注解的类，也会有一个子类的注入：
```java
@EActivity
public class MyChildActivity extends MyActivity {

}
```
##在代码区域注入一个接口的实现
如果你想通过其API（无论是超类或接口）来使用你的代码的依赖，可以作为注解``@Bean``的值参数指定的实现类。
```java
@EActivity
public class MyActivity extends Activity {

    /* A MyImplementation instance will be injected. 
     * MyImplementation must be annotated with @EBean and implement MyInterface.
     */
    @Bean(MyImplementation.class)
    MyInterface myInterface;

}
```
这样有解耦的作用么？这里的类仍然知道它的依赖实现，但至少使用这些依赖的代码必须依靠API，这是一个有趣的目标。

## 支持其他注解的使用
您可以在``@EBean``类使用大多数AA标注：
```java
@EBean
public class MyClass {

  @SystemService
  NotificationManager notificationManager;

  @UiThread
  void updateUI() {

  }

}
```
## 使用View相关的注解
您可以在``@EBean``类中使用视图相关的注释（@View，@Click...）：
```java
@EBean
public class MyClass {

  @ViewById
  TextView myTextView;

  @Click(R.id.myButton)
  void handleButtonClick() {

  }

}
```
>注意：这里视图注解相关的控件``myTextView``和点击方法``handleButtonClick()`` 只有在引用MyClass的Activity里包含相应内容视图中有这个``myTextView`` id 和 ``R.id.myButton``才有效。不然的话，``myTextView``会默认为空，而``handleButtonClick()``方法永远也不回被调用。

## 注射根上下文
你可以注入这就要看你@EBean类，使用@RootContext注解根的Android组件。请注意，它仅在上下文具有正确的类型情况下被注入。
```java
@EBean
public class MyClass {

  @RootContext
  Context context;

  // Only injected if the root context is an activity
  @RootContext
  Activity activity;

  // Only injected if the root context is a service
  @RootContext
  Service service;

  // Only injected if the root context is an instance of MyActivity
  @RootContext
  MyActivity myActivity;

}
```
通过以下Activity引用的MyClass的实例，MyClass的（上文的）``service``将是``null``。
```java
@EActivity
public class MyActivity extends Activity {

  @Bean
  MyClass myClass;

}
```
##依赖注入后执行代码
当你的``@EBean``注解类的构造函数被调用时，它的领域还没有被注入了呢。如果您需要在编译时执行代码，依赖注入之后，你应该对用一些方法使用``@AfterInject``注释。
```java
@EBean
public class MyClass {

  @SystemService
  NotificationManager notificationManager;

  @Bean
  MyOtherClass dependency;

  public MyClass() {
    // notificationManager and dependency are null
  }

  @AfterInject
  public void doSomethingAfterInjection() {
    // notificationManager and dependency are set
  }

}
```
##警告
> 如果一个父类和他的子类拥有同名的方法，这些方法被``@AfterInject``、``@AfterView`` 或者``@AfterExtras`` 注解，那么就会出现BUG，详情请见[ issue #810](https://github.com/excilys/androidannotations/issues/591)。
> 另外，虽然有关于当我们称之为@AfterViews保证顺序，-Inject或-Extras注解的方法，就不能保证每个具有相同方法名的@AfterXXX注解方法的调用顺序（参见[问题＃810](https://github.com/excilys/androidannotations/issues/810)）。
> 想知道这些注解之一的方法什么时候被调用，你可以在这里找到的[答案](https://github.com/excilys/androidannotations/wiki/%40AfterXXX-call-order)。

## Scopes
AndroidAnnotations目前支持两种范围：

+ 默认范围：在每次被注入的时候新生成一个类的实例对象；
+ 单例范围：在第一次被需要的时候创建一个Bean的实例，它然后被保留，并且在同一实例始终注入。
 
 ```java
 @EBean(scope = Scope.Singleton)
	public class MySingleton {
	
	}
 ```
 > **注意：**
 > + 如果你的bean有一个Singleton范围，它只会保持一个参考应用程序上下文（应该使用context.getApplicationContext())，这是因为它的生活时间比任何``Activity/Service`` 要长，所以它不应该保持一个参考任何此类的Android组件，以避免内存泄漏。
 > + 我们还认为应该禁用与Singleton范围@EBean组件注入和查看事件的绑定，完全一样的原因（View参考一个Activity，因此它可能会造成内存泄漏）。


