# EventBus3新特性及用法

> 本文首发于简书：[EventBus3新特性及用法](http://www.jianshu.com/p/2737b4a22cac)，欢迎关注[我的简书](http://www.jianshu.com/users/ca00dd5e52a4/latest_articles)。

![题图来自greenrobot官网.png](http://upload-images.jianshu.io/upload_images/115071-06255b488bb0ae86.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
## EventBus 3 简介
EventBus是一种为了优化Android组件之间事件传递的解耦工具，通过发布/订阅事件总线来实现事件在不同组件之间的事件传递。
在EventBus 3之前，greenrobot团队因为考虑性能原因所以比较抵触使用注解框架。目前的EventBus3开始使用注解来申明订阅事件的处理方法。虽然目前Android 6 和ART都有了，但是对于Java反射造成的性能影响还是没能很好的解决。
在EventBus3中，greenrobot团队通过利用在编译时检索所有注解代码，然后生成一个包含所有在运行时要花很大代价才能获取的数据的类，通过这种新的注解处理方式来提升性能，让EventBus3比其他的eventbus会更加快。在后文中会贴出和otto的性能比较。
<!--more-->
## EventBus 3 和 EventBus 2.x 的区别
### 回调方法改动
由于API的改动，会导致EventBus3和之前使用老版本的EventBus不兼容，因为之前版本（EventBus 2.x），在注册完事件之后，会要求写相应 onEvent()方法，包括``onEvent()``、``onEventAsync()``、``onEventBackground()``、``onEventMainThread()`` 分别对应 ``@Subscrible`` 、``@Subscrible(threadMode = ThreadMode.ASYNC)``、``@Subscribe(threadMode = ThreadMode.BACKGROUND)``、``@Subscribe(threadMode = ThreadMode.MAIN)`` 。EventBus 3中在未声明threadMob时，默认的线程模式为``ThreadMode.POSTING``。
### 异常容错处理
在EventBus3中，如果在@Subscrible标注的方法中，如果程序出错，不会立即使程序crash，而是由EventBus拦截异常，并打印错误日志。
用户可以通过EventBusBuilder来配置获取EventBus实例后的对象，来决定在处理event时是否需要抛出异常信息：
    
      eventBus = EventBus.builder().sendNoSubscriberEvent(false)        
                  .sendSubscriberExceptionEvent(false)                       
                  .throwSubscriberException(BuildConfig.DEBUG) //只有在debug模式下，会抛出错误异常 
                  .build();

以上代码使用Builder设计模式，来构建返回一个eventBus实例。在调试阶段，可以在程序出现异常时直接Crash发现错误。

## EventBus 3的使用
### 引入EventBus库文件
在这里以``gradle``引用EventBus3库为例进行说明
    
      compile 'org.greenrobot:eventbus:3.0.0'

### 定义一个事件集合
在使用EventBus时，由于我个人比较喜欢将事件进行统一管理和注释（代码如下），在其中定义了一个常用事件CommonEvent 实现了BaseEvents接口的两个方法。

      public interface BaseEvents{
              setObject(Object obj);
              Object getObject();
              //事件定义
             enum CommonEvent implements BaseEvent {    
                    LOGIN, //登录    
                    LOGOUT; //登出    
                    private Object obj;    
                    @Override    
                    public void setObject(Object obj) {
                            this.obj = obj;    
                    }    
                    @Override    
                    public Object getObject() {        
                            return obj;    
                    }
              }
              // ... 其他事件定义
      }

这样定义基本事件，主要是为了便于事件管理添加和维护。避免新建很多不必要的类和重载过多的事件处理方法。

### 注册事件及声明事件处理方法
在Activity/Frament中，可以定义一个基类，在基类的onStart()和onStop()方法中对EventBus进行注册和注销（官方推荐）。然后其他的子类集成基类，复写和实现对应的注册方法即可。
    
      @Override
      public void onStart() {    
          super.onStart();  
          registEventBus();
      }

      @Override
      public void onStop() {   
          unRegistEventBus(); 
          super.onStop(); 
      }
      
      protected void registEventBus(){
          //子类如果需要注册eventbus，则重写此方法
          //EventBus.getDefault().register(this);
      }
      
      protected void unRegistEventBus(){
          //子类如果需要注销eventbus，则重写此方法
          //EventBus.getDefault().unregister(this);
      }

注册方法后，需要使用注解@Subscribe声明处理事件的方法，以下代码摘自[官方介绍](http://greenrobot.org/eventbus/documentation/how-to-get-started/)
  
      // This method will be called when a MessageEvent is posted
      @Subscribe
      public void onMessageEvent(MessageEvent event){    
            Toast.makeText(getActivity(), event.message, Toast.LENGTH_SHORT).show();
      }
      // This method will be called when a SomeOtherEvent is posted
      @Subscribe
      public void handleSomethingElse(SomeOtherEvent event){  
          doSomethingWith(event);
      }

### 发送事件
在发送事件时，可以利用我们定义的一个BaseEvent进行值的传递，因为定义的是一个Object对象，只需要保证类型转换正确的前提下就可以随意传值了。示例如下：
      
      BaseEvents. CommonEvent event = BaseEvents.CommonEvent.LOGIN;
      event.setObject("Send Event"); //传入一个String对象
      eventBus.post(event); //发布事件

如此就简单的实现了一个EventBus事件的发布。

### StickyEvent
有时候，我们会面对这样一个问题，那就是我们要把一个Event发送到一个还没有初始化的Activity/Fragment，即尚未订阅事件。那么如果只是简单的post一个事件，那么是无法收到的，这时候，需要使用 StickyEvent，实例说明：
    
    BaseEvents. CommonEvent event = BaseEvents.CommonEvent.LOGIN;
    event.setObject("Send StickyEvent"); 
    EventBus.getDefault().postSticky(event);

在启动的Activity，暂且定义为StickyActivity 继承 BaseActivity 实现    ``registEventBus()``和``unRegistEventBus()``，代码片段如下：
  
      @Override
      public void onStart() {    
          super.onStart();  
          registEventBus();
      }

      @Override
      public void onStop() {   
          unRegistEventBus(); 
          super.onStop(); 
      }
      
      @Subscribe(sticky = true)    
      public void onEvent(BaseEvents. CommonEvent event) {        
          // UI updates must run on MainThread  
          if(event==BaseEvent.CommonEvent.LOGIN){
                 String conent = (String)event.getObject();
                 Log.d(TAG,"Content is : "+content);
          } 
          
      }      

      protected void registEventBus(){
          EventBus.getDefault().register(this);
      }
      
      protected void unRegistEventBus(){
          //EventBus.getDefault().unregister(this);
      }

以上就是发送一个stickyEvent的全过程，下面讲一下event的优先级和拦截

### 事件处理优先级及事件拦截
有时候在开发中，需要定义处理相同事件的优先级，我们先来看一下EventBus3对于Subscrible的注解定义源码：
    
    @Documented
    @Retention(RetentionPolicy.RUNTIME)
    @Target({ElementType.METHOD})
    public @interface Subscribe {   
           ThreadMode threadMode() default ThreadMode.POSTING;    
            /**    
             * If true, delivers the most recent sticky event (posted with    
             * {@link EventBus#postSticky(Object)}) to this subscriber (if event available).     
             */    
            boolean sticky() default false;    
            /** Subscriber priority to influence the order of event delivery.     
              * Within the same delivery thread ({@link ThreadMode}), higher priority subscribers will receive events before     
              * others with a lower priority. The default priority is 0. Note: the priority does *NOT* affect the order of     
              * delivery among subscribers with different {@link ThreadMode}s! 
              */    
            int priority() default 0;
    }

可以看到 priority 的值默认为0，优先级随定义的priority的值越大优先级越高。注意，在注解``@Retention(RetentionPolicy.RUNTIME)`` 这一项是将注解写入编译的class文件中，这个也是前面介绍到EventBus3解决注解反射导致性能问题的一个关键点。
priority 实例简介：
    
    @Subscribe(priority = 10)
    public void handleEvent(BaseEvents.CommonEvent event){
          //....
    }

有了事件处理优先级的定义，那么我们就可以在高优先级的事件处理中，将事件传递拦截下来，经过实际测试，只能在 ``threadMode = ThreadMode.POSTING`` 的注释方法中才能拦截事件。官方如下介绍的：

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/115071-bd882c9936e7a143.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

事件通常是被高优先级的订阅者取消，目前取消事件的处理方法仅限在ThreadMode 为 POSTING才能取消拦截事件。
> 注：POSTING的事件，表示当前的post event和subscribe event两者在同一个线程之中。

### 注意事项

* 工作线程和UI线程之间的事件传递
> 在使用EventBus3时，由于event 在任何地方都可以发布一个事件，那么在不同线程之间传递事件，比如在工作线程传递一个事件更新UI线程中的一个控件，则需要注意threadMode的切换。
* 取消线程仅限于 threadMode处于POSTING模式才可以
  > 否则容易导致错误:
  org.greenrobot.eventbus.EventBusException: This method may only be called from inside event handling methods on the posting thread

### 混淆
关于正式打包签名，混淆就不多说了，把下面的混淆代码，粘贴到你的``proguard-rules.pro``下就ok了

      -keepattributes *Annotation*
      -keepclassmembers class ** {    
          @org.greenrobot.eventbus.Subscribe <methods>;
      }
      -keep enum org.greenrobot.eventbus.ThreadMode { 
            *; 
      }
      # Only required if you use AsyncExecutor
      -keepclassmembers class * extends org.greenrobot.eventbus.util.ThrowableFailureEvent {    
          <init>(java.lang.Throwable);
      }

## EventBus与otto的对比
这里的对比图来自官方Github网站，对于线程之间传递Event的比较做了一个简化，简化为不同线程之间的事件传递。
### 功能对比
|               |  EventBus | otto |
|--------|---------|-----|
| 声明事件处理方法| 注解（从3.0开始使用预编译解决性能问题）| 注解|
| 事件可扩展 | 可以 | 可以|
| 订阅者扩展 | 可以 | 不行 |
| 缓存大量最近事件| 可以（用sticky events） | 不行|
| 事件生产者(比如编码缓存的事件)|不行| 可以 |
| 事件相同线程中传递| 可以 | 可以|
| 事件不同线程之间传递| 可以 | 不可以|

### 性能对比
性能对比，由于个人没有去写具体的Demo测试，不过[官方源码](https://github.com/greenrobot/EventBus)有关于性能的比较测试源码，一下截图来自官方Github的比较结果：

![Compare.png](http://upload-images.jianshu.io/upload_images/115071-4db9bc75ac0d66eb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
总体而言，EventBus在各个方面完爆otto，当然在实际项目中，还是应该考虑性能和稳定做出取舍。
## END
由于本人水平有限，文中难免有所疏漏，有问题还请斧正，开卷有益多多交流。如有疑问可以直接留言讨论~ O(∩_∩)O哈哈~ 
>参考资料：
[EventBus官网文档](http://greenrobot.org/eventbus/documentation/)
 
--------------------
