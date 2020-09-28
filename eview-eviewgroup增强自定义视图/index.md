# @EView@EViewGroup增强自定义视图


>译文地址：
>https://github.com/excilys/androidannotations/wiki/Enhance-custom-views

如果你想创建自定义组建，可以使用@EView和@EViewGroup的注解方法。
## 为什么要使用自定义组建？
如果你发现在应用程序中的不同位置的某些布局部分需要复制相同的内容和调用相同的方法，那么自定义组建可以让你的开发轻松很多！
<!--more-->
## @EView自定义视图
只要创建一个类，让这个类继承View，并在这个类前加上@EView注释，如下所示，你可以这样使用@EView：
```java
@EView
public class CustomButton extends Button {

        @App
        MyApplication application;

        @StringRes
        String someStringResource;

	    public CustomButton(Context context, AttributeSet attrs) {
	        super(context, attrs);
	    }
}
```
然后就可以在你的布局中开始使用它（注:不要忘记 ``_`` <下划线>）:
```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical" >

    <com.androidannotations.view.CustomButton_
        android:layout_width="match_parent"
        android:layout_height="wrap_content" />

    <!-- ... -->

</LinearLayout>
```
您也可以通过编程创建它：（注意：别忘了下划线）
```java
CustomButton button = CustomButton_.build(context);
```
##@EViewGroup自定义ViewGroups
###如何创建？
首先，让我们来创建此组件的布局XML文件。
```xml
<?xml version="1.0" encoding="utf-8"?>
<merge xmlns:android="http://schemas.android.com/apk/res/android" >

    <ImageView
        android:id="@+id/image"
        android:layout_alignParentRight="true"
        android:layout_alignBottom="@+id/title"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:src="@drawable/check" />

    <TextView
        android:id="@+id/title"
        android:layout_toLeftOf="@+id/image"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:textColor="@android:color/white"
        android:textSize="12pt" />

    <TextView
        android:id="@+id/subtitle"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_below="@+id/title"
        android:textColor="#FFdedede"
        android:textSize="10pt" />

</merge>
```
> 您知道合并标签`<merge>`吗？当这种布局被直接添加到父视图中，会直接展开，可以减少在视图层次。

如你所见，我用了一些RelativeLayout的特定布局的属性``(layout_alignParentRight, layout_alignBottom, layout_toLeftOf, 等等..)`` ，那是因为我假设我的布局将在RelativeLayout的展开。而这确实是我做的！
```java
@EViewGroup(R.layout.title_with_subtitle)
public class TitleWithSubtitle extends RelativeLayout {

    @ViewById
    protected TextView title, subtitle;

    public TitleWithSubtitle(Context context, AttributeSet attrs) {
        super(context, attrs);
    }

    public void setTexts(String titleText, String subTitleText) {
        title.setText(titleText);
        subtitle.setText(subTitleText);
    }

}
```
看，是不是很简单？
现在，让我们来看看如何使用这个全新的组件。
###如何使用？
自定义组件可以声明，并放置在一个布局就像任何其他的视图一样：
```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:gravity="center"
    android:orientation="vertical" >

    <com.androidannotations.viewgroup.TitleWithSubtitle_
        android:id="@+id/firstTitle"
        android:layout_width="match_parent"
        android:layout_height="wrap_content" />

    <com.androidannotations.viewgroup.TitleWithSubtitle_
        android:id="@+id/secondTitle"
        android:layout_width="match_parent"
        android:layout_height="wrap_content" />

    <com.androidannotations.viewgroup.TitleWithSubtitle_
        android:id="@+id/thirdTitle"
        android:layout_width="match_parent"
        android:layout_height="wrap_content" />

</LinearLayout>
```
> 不要忘了，在组件名称以`_`（下划线）结尾！

而且因为我使用AA，我可以很容易地让我的自定义组件在我的Activity中注入，并利用它们！
```java
@EActivity(R.layout.main)
public class Main extends Activity {

    @ViewById
    protected TitleWithSubtitle firstTitle, secondTitle, thirdTitle;

    @AfterViews
    protected void init() {

        firstTitle.setTexts("decouple your code",
                "Hide the component logic from the code using it.");

        secondTitle.setTexts("write once, reuse anywhere",
                "Declare you component in multiple " +
                "places, just as easily as you " +
                "would put a single basic View.");

        thirdTitle.setTexts("Let's get stated!",
                "Let's see how AndroidAnnotations can make it easier!");
    }

}
```
大多数自定义组建都可以使用AA的@EViewGroup注释，不妨一试！

