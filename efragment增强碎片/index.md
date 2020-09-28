# @EFragment增强碎片

>原文地址：
>https://github.com/excilys/androidannotations/wiki/Enhance-Fragments

## 支持FragmentActivity
在AndroidAnnotations 2.6之前,没有支持片段注入。然而,我们确保至少继承FragmentActivity的活动不会打破AndroidAnnotations的规则：
```java
@EActivity(R.id.main)
public class DetailsActivity extends FragmentActivity {

}
```
<!--more-->
##支持Fragment
AndroidAnnotations同时支持``android.app.fragment``和``android.support.v4.app.fragment``,并会自动使用正确基于片段的类型的api。
##增强碎片
开始在一个片段使用AndroidAnnotations特性,可以使用``@EFragment``注释该Fragment类：
```java
@EFragment
public class MyFragment extends Fragment {

}
```
AndroidAnnotations将生成一个Fragment的增强子类：``MyFragment_``。当你需要创建新的片段时，应该在xml布局中使用生成的子类(``MyFragment_``)。
```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="fill_parent"
    android:layout_height="fill_parent"
    android:orientation="horizontal" >

    <fragment
        android:id="@+id/myFragment"
        android:name="com.company.MyFragment_"
        android:layout_width="fill_parent"
        android:layout_height="fill_parent" />

</LinearLayout>
```
代码中生成实例对象，也需要使用其子类实例化：
```java
MyFragment fragment = new MyFragment_();
```
你可以在``@EFragment`` 注解的类中，使用各种注释方法：
```java
@EFragment
public class MyFragment extends Fragment {
    @Bean
    SomeBean someBean;

    @ViewById
    TextView myTextView;

    @App
    MyApplication customApplication;

    @SystemService
    ActivityManager activityManager;

    @OrmLiteDao(helper = DatabaseHelper.class, model = User.class)
    UserDao userDao;

    @Click
    void myButton() {
    }

    @UiThread
    void uiThread() {

    }

    @AfterInject
    void calledAfterInjection() {
    }

    @AfterViews
    void calledAfterViewInjection() {
    }

    @Receiver(actions = "org.androidannotations.ACTION_1")
    protected void onAction1() {

    }
}
```
>视图注入和事件监听器绑定只能基于视图包含里面的片段。但是请注意,它不是目前的@EBean注入内部碎片:他们能访问活动的视图。

## 碎片布局
将一个视图与一个片断联系到一起的标准方法是覆写``onCreateView()`` 方法：
```java
@EFragment
public class MyFragment extends Fragment {
    public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
        View view = inflater.inflate(R.layout.my_fragment_layout, container, false);
        return view;
    }
}
```
你可以通过设置@EFragment注释参数的值让AndroidAnnotations为您处理视图问题：
```java
@EFragment(R.layout.my_fragment_layout)
	public class MyFragment extends Fragment {
}
```
在使用``@EFragment``时你需要覆写``onCreateView()`` ，因为您需要访问``savedInstanceState``,你仍然可以让AndroidAnnotations处理创建通过返回null布局:
```java
@EFragment(R.layout.my_fragment_layout)
public class MyFragment extends Fragment {
    public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
        return null;
    }
}
```
## 强制注入布局
你可能会在一些使用@EActivity 、@EFragment、@EView、@EViewGroup、@EBean的注解类中通过使用@FragmentById 或 @FragmentByTag注入Fragment。如果你不指定任何注释的值,使用字段名作为值。
> 我们建议使用@FragmentById来注入Fragment而不是使用@FragmentByTag，因为没有编译时执行验证后者。

请注意,@FragmentById @FragmentByTag只能注入片段,不创建它们,所以他们必须已经存在于活动中(通过定义在布局或以编程方式在onCreate()创建）。
>你可以注入碎片，即使这些碎片类没有使用@EFragment注释。

```java
@EActivity(R.layout.fragments)
public class MyFragmentActivity extends FragmentActivity {
  @FragmentById
  MyFragment myFragment;

  @FragmentById(R.id.myFragment)
  MyFragment myFragment2;

  @FragmentByTag
  MyFragment myFragmentTag;

  @FragmentByTag("myFragmentTag")
  MyFragment myFragmentTag2;
}
```

