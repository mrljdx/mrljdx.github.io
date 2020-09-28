# @EApplication增强Application类

> 译文地址：
> https://github.com/excilys/androidannotations/wiki/Enhancing-the-Application-class

## 加强应用类
您可以使用@EApplication注释来增强Android的Application类：
```java
@EApplication
public class MyApplication extends Application {

}
```
<!--more-->
那么你就可以开始使用最AA注解，除了那些视图和extras先关的内容：
```java
@EApplication
public class MyApplication extends Application {

  public void onCreate() {
    super.onCreate();
    initSomeStuff();
  }

  @SystemService
  NotificationManager notificationManager;

  @Bean
  MyEnhancedDatastore datastore;

  @RestService
  MyService myService;

  @OrmLiteDao(helper = DatabaseHelper.class, model = User.class)
  UserDao userDao;

  @Background
  void initSomeStuff() {
    // init some stuff in background
  }
}
```
##注入您的应用程序类
您可以使用@EApplication注释来加强你的Android应用程序类：
```java
@EApplication
public class MyApplication extends Application {

}
```
它也适用于任何类型的注释部件中使用，如@EBean：
```java
@EBean
public class MyBean {

  @App
  MyApplication application;

}
```

