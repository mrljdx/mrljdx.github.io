# Injecting Views

>原文地址：
>https://github.com/excilys/androidannotations/wiki/Injecting-Views

## @ViewById
在Activity中@ViewById标注的字段，必须的在activity所在的布局layout中可以找到相应的ID，类似于调用findViewById()方法。
注释的视图id可以设置参数,即@ViewById(R.id.myTextView)。如果没有设置视图id,将使用的名称字段。字段不能是私有的。
<!--more-->
例如：
```java
@EActivity
public class MyActivity extends Activity {

  // Injects R.id.myEditText
  @ViewById
  EditText myEditText;

  @ViewById(R.id.myTextView)
  TextView textView;
}
```
##@AfterViews
使用@AfterViews注释的一个方法表示应该调用视图绑定了。
>在调用``onCreate()``时,``@ViewById``标注的字段没有初始化。因此你可以在@AfterViews注释的方法中进行视图相关的操作。

例如：
```java
@EActivity(R.layout.main)
public class MyActivity extends Activity {

    @ViewById
    TextView myTextView;

    @AfterViews
    void updateTextWithDate() {
        myTextView.setText("Date: " + new Date());
    }
[...]
```
你可以用``@AfterViews``标注多个方法。别忘了,你不应该在``onCreate()``方法中使用任何与视图有关的字段:
```java
@EActivity(R.layout.main)
public class MyActivity extends Activity {

    @ViewById
    TextView myTextView;

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        // 不要这样做!! 这将会抛出一个NullPointerException的异常, 因为此时myTextView 还没有被赋值为null.
        // myTextView.setText("Date: " + new Date());
    }
[...]
```
回顾一下注释总是很快的，因此使用任何已经被注释的字段，比如，在``@AfterViews`` 注释的方法中 用``@Extra`` 或 ``@InstanceState`` 这些和视图无关的标记值是安全的（和``@AfterView``是一样的），因此在``@AfterViews``注释的方法中，你可以安全的假设这些值已经根据预期设定初始化好了。
```java
@EActivity(R.layout.main)
public class MyActivity extends Activity {

    @ViewById
    TextView myTextView;

    @InstanceState
    Integer textPosition;

    @AfterViews
    void updateTextPosition() {
        myTextView.setSelection(textPosition); //Sets the cursor position of myTextView
    }
[...] // The remaining code must keep textPosition updated.
```
## 警告
如果父类和子类有@AfterViews,@AfterInject或@AfterExtras名称相同的带注释的方法,生成的代码将会有BUG。查看更多细节[问题# 591](https://github.com/excilys/androidannotations/issues/591)。
同时，当我们调用那些@AfterViews、@AfterInject、@AfterExtras注解的方法时会有一个调用顺序，但是如果@AfterView、@AfterInject、@AfterExtras 注解的方法是同名的，那就不能确保这几个方法的调用顺序了 ，请看( [issue #810](https://github.com/excilys/androidannotations/issues/810))。
关于这些注解的方法调用顺序，你可以看我的这篇博客译文：[@AfterXXX 调用顺序](http://blog.csdn.net/mrljdx/article/details/44891189)。

## @ViewsById
这个注解与``@ViewById``有点类似，不同的是这个注解可以注入一组视图，他可以用在``java.util.List`` 或 ``android.view.View`` 子类型字段，注释值应该是一个一组R.id.*的值。
例如：
```java
@EActivity
public class MyActivity extends Activity {

  @ViewsById({R.id.myTextView1, R.id.myOtherTextView})
  List<TextView> textViews;

  @AfterViews
  void updateTextWithDate() {
    for (TextView textView : textViews) {
      textView.setText("Date: " + new Date());
    }
  }
}
```
