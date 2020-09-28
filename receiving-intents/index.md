# Receiving intents

>原文地址：
>https://github.com/excilys/androidannotations/wiki/Receiving-intents

## 接收意图
``@Receiver``注释通知你的代码的意图,而不必手工申报和注册一个BroadcastReceiver。例如：
```java
@EActivity
public class MyActivity extends Activity {

  @Receiver(actions = "org.androidannotations.ACTION_1")
  protected void onAction1() {
    // Will be called when an org.androidannotations.ACTION_1 intent is sent.
  }

}
```
<!--more-->
``@Receiver`` 注释支持 [activities](https://github.com/excilys/androidannotations/wiki/Enhance-activities), [fragments](https://github.com/excilys/androidannotations/wiki/Enhance-Fragments) 和 [services](https://github.com/excilys/androidannotations/wiki/Enhance-services) (以及[intent services](https://github.com/excilys/androidannotations/wiki/Enhance-IntentServices))。
 
## 参数说明
你可能只有一个``android.content.intent``或没有参数。
使用``@Receiver`` 注解的方法可能有以下参数：

+ 一个在方法``void onReceive(Context context, Intent intent)`` 中给定的上下文参数``android.content.Context``。
+ 在方法``void onReceive(Context context, Intent intent)`` 中给定的意图``android.content.Intent``。
+ 任何放入Intent中被本地``android.os.Parcelable`` 或``java.io.Serializable`` 序列化的被``@Receiver.Extra``注释的参数，key就是用``@Receiver.Extra`` 声明的值。
 
## Data Schemes
``dataScheme``参数可以设置一个或多个dataSchemes,这些值会交给由Receiver处理。
```java
@EActivity
public class MyActivity extends Activity {

  @Receiver(actions = android.content.Intent.VIEW, dataSchemes = "http")
  protected void onHttp() {
    // Will be called when an App wants to open a http website but not for https.
  }

  @Receiver(actions = android.content.Intent.VIEW, dataSchemes = {"http", "https"})
  protected void onHttps() {
    // Will be called when an App wants to open a http or https website.
  }

}
```
##Registration(注册)
注释以编程方式在父类（Activity, Fragment or Service）的生命周期内注册和注销BroadcastReceiver。
``registerAt``可选参数指定注册/注销时发生。默认值是**OnCreateOnDestroy**。
下表列出了``registerAt``值,当注册/ deregisration发生和显示时可以使用的值。
| Tables| Register| Unregister|Activity|Fragment|Service|
| -------- |:----:|:-----:|:----:|:-----:|:----:|
| **OnCreateOnDestroy**      | onCreate | onDestroy | X| X | X |
| **OnStartOnStop**      | onStart| onStop|X|X| |
| **OnResumeOnPause** | OnAttachOnDetach | onAttach ||X||

这里有一段示例代码：
```java
@EFragment
public class MyFragment extends Fragment {

  @Receiver(actions = "org.androidannotations.ACTION_1")
  protected void onAction1RegisteredOnCreateOnDestroy() {
  }

  @Receiver(actions = "org.androidannotations.ACTION_2", registerAt = Receiver.RegisterAt.OnAttachOnDetach)
  protected void onAction2RegisteredOnAttachOnDetach(Intent intent) {
  }

  @Receiver(actions = "org.androidannotations.ACTION_3", registerAt = Receiver.RegisterAt.OnStartOnStop)
  protected void action3RegisteredOnStartOnStop() {
  }

  @Receiver(actions = "org.androidannotations.ACTION_4", registerAt = Receiver.RegisterAt.OnResumeOnPause)
  protected void action4RegisteredOnResumeOnPause(Intent intent) {
  }

}
```

## Local broadcasting（本地广播）
可选参数的``local`` 注册BroadcastReceiver本地使用[LocalBroadcastManager](http://developer.android.com/reference/android/support/v4/content/LocalBroadcastManager.html) 来代替父类上下文(activity,fragment或service)。
其中 ``local`` 的默认值为false。
```java
@EService
public class MyService extends Service {

  @Receiver(actions = "org.androidannotations.ACTION_1", local = true)
  protected void onAction1OnCreate() {  
  }

  @Override
  public IBinder onBind(Intent intent) {
    return null;
  }

}
```

