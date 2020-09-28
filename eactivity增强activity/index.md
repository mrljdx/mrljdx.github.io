# @EActivity增强Activity


> 译文地址：
> https://github.com/excilys/androidannotations/wiki/Enhance-activities

<!--more-->
## @EActivity
使用@EActivity注解表明一个Activity将通过AndroidAnnotations增强。其值的参数必须是有效的布局ID，这将被用作该Activity的内容视图。
您可以将value参数为空，这意味着不会设置内容视图。完成绑定之前，你可能会想在Activity的onCreate（）方法中设置自己的内容视图。
设置布局ID的示例：
```java
@EActivity(R.layout.main)
public class MyActivity extends Activity {

}
```
不设置布局ID的示例：
```java
@EActivity
public class MyListActivity extends ListActivity {

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.main);
    }

}
```
另请参阅[如何启动一个注解的活动](https://github.com/excilys/androidannotations/wiki/HowItWorks#starting-an-annotated-activity)。
