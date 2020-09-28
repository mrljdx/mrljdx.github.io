# @EProvider加强contentprovider

>原文地址：
>https://github.com/excilys/androidannotations/wiki/Enhance-contentproviders

你可以使用@EProvider注释加强一个Android内容提供商与:
```java
@EProvider
public class MyContentProvider extends ContentProvider {

}
```
<!--more-->
然后,您可以开始使用大多数AA注释（除了相关的views和extras）:
```java
@EProvider
public class MyContentProvider extends ContentProvider {

  @SystemService
  NotificationManager notificationManager;

  @Bean
  MyEnhancedDatastore datastore;

  @OrmLiteDao(helper = DatabaseHelper.class, model = User.class)
  UserDao userDao;

  @UiThread
  void showToast() {
    Toast.makeText(getContext().getApplicationContext(), "Hello World!", Toast.LENGTH_LONG).show();
  }

  // ...
}
```

