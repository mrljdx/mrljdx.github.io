# AA-4.0升级指南


> 笔者注：AA-4.0为AndroidAnnotations-4.0的简称

## AA4.0已经模块化了
现在AA-4.0的库被分割为一些小的模块，这意味着你以前在AA-3.x版本之前库中使用的一些注解方法都被分拆为一些单独的模块，你需要在Gradle配置文件中添加新的依赖库，改变import的包名。

<!--more-->

### @OrmLiteDao
以[@OrmLiteDao](https://github.com/excilys/androidannotations/wiki/Ormlite)的注解使用为例，在引入AA-4.0后，如果你的项目中有用到[@OrmLiteDao](https://github.com/excilys/androidannotations/wiki/Ormlite)，那么你需要在app的``build.gradle``项目依赖中添加下文件：

    compile "org.androidannotations:ormlite-api:4.0.0"
    apt  "org.androidannotations:ormlite:4.0.0"

修改import的包名：
  
    org.androidannotations.ormlite.annotations.OrmLiteDao

### Otto
将Otto库整合到AA-4.0中配置如下：

1. 添加[AA-4.0](https://github.com/excilys/androidannotations/wiki/GettingStarted)到工程；
2. 添加``Otto``库的插件：
        dependencies { 
            apt "org.androidannotations:otto:$AAVersion" 
            compile "org.androidannotations:otto-api:$AAVersion"
        } 
3. 添加[Otto](http://square.github.io/otto/)库依赖
        dependencies { compile 'com.squareup:otto:1.3.8'}

### Rest Client
如果项目中使用[Rest Client](https://github.com/excilys/androidannotations/wiki/Rest-API#rest) ，需要在``build.gradle``中添加配置依赖库：

         dependencies { 
            apt "org.androidannotations:rest-spring-api:$AAVersion" 
            compile "org.androidannotations:rest-spring:$AAVersion"
        } 
    
更改导入的包名:
    
    org.androidannotations.rest.spring.annotations.Rest //etc
    org.androidannotations.rest.spring.api.RestClientHeaders // etc

### @RoboGuice

使用 [@RoboGuice
](https://github.com/excilys/androidannotations/wiki/RoboGuiceIntegration), 在项目的*build.gradle*中添加依赖：

      compile "org.androidannotations:roboguice-api:4.0.0"
      apt "org.androidannotations:roboguice:4.0.0"

修改 imports :

    org.androidannotations.roboguice.annotations.RoboGuice

## 在Fragment中注解的View对象会被清理

在AA-4.0中对在Fragment中使用``@ViewById``和``@ViewsById``的View对象会在Fragment的声明周期onDestoryView()时设为``null``，如果程序执行了Fragment的声明周期事件onDestoryView()后，仍然访问该View，则会得到*NullPointerExceptions*异常，为了避免发生这种情况，可以使用新的注解``@IgnoreWhen``，如下：
        
      //当前状态为DESTORYED，则忽略此方法，不调用
      @IgnoreWhen(IgnoreWhen.State.VIEW_DESTROYED)
      void someMethodCalledAfterViewDestroyed() { 
          injectedView.setText("Hello"); 
      }

## 自AA-4.0不再支持2.3.1以下的Android设备
现在市面上的Android设备已经都是4.0及以上了，2.3的设备都比较少，所以不支持2.3.1以下的安卓设备也没啥影响。

## 编译运行最小支持 JDK7不再支持JDK 6
AA-4.0不再支持JDK 6的编译，不过你依然可以使用JDK 6的源码来编译。话说现在的环境都支持JDK8了，到时候在*build.gradle*的``android{} ``DSL域中配置编译选项即可。
  
      //java版本
      compileOptions {    
          sourceCompatibility JavaVersion.VERSION_1_7    
          targetCompatibility JavaVersion.VERSION_1_7
      }

## @ReceiverAction 修改
在AA-3.x版本及以前，如果使用``@ReceiverAction``没有指定Action的值，则会将方法名作为Action，从AA-4.0开始不再支持这种方式，必须在注解中指定Action的值。
实例说明：
    
      @ReceiverAction
      public void simpleAction() { // AA 3.x, action = "simpleAction"
      }

      @ReceiverAction(actions = "simpleAction")
      public void simpleAction() { // AA 4.0.0
      }

## 去掉了 @OrmLiteDao 注解的model参数
虽然在AA-4.0中移除了model的参数配置选项，不过在编译期间，会自动根据``Dao``类判断model的值，减少了冗余。
实例说明：
    
    @OrmLiteDao(helper = DatabaseHelper.class, model = Car.class) // AA 3.x
    Dao<Car, Long> injectedDao;

    @OrmLiteDao(helper = DatabaseHelper.class) // AA 4.0.0
    Dao<Car, Long> injectedDao;


## 去掉了 @NoTitle 注解
可能是为了代码规范，在AA-4.0中去掉了``@NoTitle``注解，可以通过``@WindowFeature``注解来实现同样的功能(不显示标题)
    
      @NoTitle // AA 3.x
      @EActivity
      public class MyActivity extends Activity {
      }
      
      @WindowFeature(Window.FEATURE_NO_TITLE) // AA 4.0.0
      @EActivitypublic class MyActivity extends Activity {
      }

## 不再支持ActionBarSherlock
由于Android官方的AppCompat比ActionBarSherlock要好，而且ActionBarSherlock这个项目也不再维护，所以AA-4.0也不再提供对ActionBarSherlock的支持。在项目中可以使用 AppCompat Support Library来替代ActionBarSherlock。
> PS：现在应该没有人用ActionBarSherlock这么古老的库了吧~

## @Extra 和 @AfterExtras 的变动
在AA-4.0中，不会再在onNewIntent()方法中调用setIntent()方法。这个对于3.x的用户基本上没影响。

##  在@Rest 方法中所有的参数都需要添加注解
对于以前项目使用AA-3.x的，这个改动有点小大，得改不少关于Rest Client的代码，从AA-4.0开始你必须明确的告诉编译处理器每个Rest请求方法中的参数代表的意思。
使用``@Body``注解指明请求的body类型：

     @Rest(converters = GsonHttpMessageConverter.class)
     public interface MyRestClient { 
        @Put 
        void putEvent(Event event); // AA 3.x request body is the event variable  
      }

      @Rest(converters = GsonHttpMessageConverter.class)
      public interface MyRestClient { 
        @Put 
        void putEvent(@Body Event event); // AA 4.0.0
      }

使用``@Path``指明url中的参数:
    
      @Rest(converters = GsonHttpMessageConverter.class)
      public interface MyRestClient { 
          @Get("/events/{id}" 
          Event getEvent(int id); // AA 3.x the id method param is an url variable
      }

      @Rest(converters = GsonHttpMessageConverter.class)
      public interface MyRestClient { 
          @Get("/events/{id}" 
          Event getEvent(@Path int id); // AA 4.0.0
      }

总体来说，AA-4.0中关于@Rest这块的注解方法，有很多借鉴了Retrofit。这是一个不错的开端，后续版本可能回减少对于SpringAndroid的项目库的依赖。逐渐的形成一种插件化的网络请求注解库。

## END
本文大部分内容参考[AA 4.0 migration guide](https://github.com/excilys/androidannotations/wiki/4.0.0-migration-guide) ，此次的AA-4.0版本是一次重大更新，很明显的改变就是将之前一个库包含所有的功能拆分成了模块化的组件，根据项目需求导入相应的组件库，减少了项目的库依赖，同时也减少了项目的包体积。
其次，对于一些代码上的规范也做了不少调整，更加重要的是，对于Fragment生命周期的一些视图组建的内存管理做了进一步的调整。整体来说建议各位同学升级到AA-4.0版本，这个版本的改动有点大，升级需谨慎，别赶着发版本升个级。
还是那句经典的话，“no zuo no die,don't try”。

-------------------------
