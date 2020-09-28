# @EReceiver加强广播接收器

>原文地址：
>https://github.com/excilys/androidannotations/wiki/Enhance-broadcastreceivers

你可以使用@EReceiver注释增强一个Android BroadcastReceiver ：
```java
@EReceiver
public class MyReceiver extends BroadcastReceiver {

}
```
<!--more-->
然后,您可以开始使用大多数AA注释(除了相关的view和extras):
```java
@EReceiver
public class MyReceiver extends BroadcastReceiver {

  @SystemService
  NotificationManager notificationManager;

  @Bean
  SomeObject someObject;

}
```
##@ReceiverAction
``@ReceiverAction``注释允许使用增强的接收器来简单地处理广播。
默认情况下使用方法名来确定处理哪个Action,但是你可以定义``@ReceiverAction``动作名称的值来筛选Action名称。
被``@ReceiverAction``注解的方法一般会包含以下参数：

+ 一个在``void onReceive(Context context, Intent intent)`` 方法中给定的上下文``android.content.Context``。
+ 一个在``void onReceive(Context context, Intent intent)`` 方法中给定的意图(Intent)``android.content.Intent`` 。
+ 任何本地，一个放入Intent中被序列化（``android.os.Parcelable`` 或 ``java.io.Serializable``） 的被``@ReceiverAction.Extra`` 注解的参数。extras的key就是``@ReceiverAction.Extra`` 中注解的参数名称。

具体代码如下：
```java
@EReceiver
public class MyIntentService extends BroadcastReceiver {

    @ReceiverAction("BROADCAST_ACTION_NAME")
    void mySimpleAction(Intent intent) {
        // ...
    }

    @ReceiverAction
    void myAction(@ReceiverAction.Extra String valueString, Context context) {
        // ...
    }

    @ReceiverAction
    void anotherAction(@ReceiverAction.Extra("specialExtraName") String valueString, @ReceiverAction.Extra long valueLong) {
        // ...
    }

    @Override
    public void onReceive(Context context, Intent intent) {
        // empty, will be overridden in generated subclass
    }
}
```
> **注意:**由于BroadcastReceiver # onReceive是抽象的,你必须添加一个空实现。为了方便,我们提供AbstractBroadcastReceiver类,实现了这一方法的,所以你不需要在你的实际类中去做，如果你得到它

>**注意:**你现在可以通过操作@ReceiverAction的值，来传递多个Action。

```java
  @ReceiverAction({"MULTI_BROADCAST_ACTION1","MULTI_BROADCAST_ACTION2"})
    void multiAction(Intent intent) {
        // ...
    }
```
## Data Schemes
通过使用设置``dataScheme``参数你可以设置一个或多个dataSchemes交由Reciver来处理。
```java
@EReceiver
public class MyIntentService extends BroadcastReceiver {

  @ReceiverAction(actions = android.content.Intent.VIEW, dataSchemes = "http")
  protected void onHttp() {
    // Will be called when an App wants to open a http website but not for https.
  }

  @ReceiverAction(actions = android.content.Intent.VIEW, dataSchemes = {"http", "https"})
  protected void onHttps() {
    // Will be called when an App wants to open a http or https website.
  }

}
```
##@Receiver注释
使用``@Receiver`` 注释可以通知你的activity/fragment/service的意图,从而替代声明一个``BroadcastReceiver``。
```java
@EActivity
public class MyActivity extends Activity {

  @Receiver(actions = "org.androidannotations.ACTION_1")
  protected void onAction1() {

  }

}
```
更多关于@Reciver注解的信息请参考[@Reciver annotations](https://github.com/excilys/androidannotations/wiki/Receiving-intents)

