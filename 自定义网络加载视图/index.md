# 自定义网络加载视图


## NetworkImageView简介：
在布局中使用自定义视图：
```xml
<com.mrljdx.loadimage.NetWorkImageView
        android:id="@+id/myNetImgView"
        android:layout_width="200dp"
        android:layout_height="200dp"
        android:layout_gravity="center_horizontal" />
```
<!--more-->
在代码中使用：
```java
NetWorkImageView myImageView = findViewById(R.id.myNetImgView);
//参数 图片的url 以及 加载图片时的默认图片
myImageView.load("http://7u2j5d.com1.z0.glb.clouddn.com/test.jpg",R.drawable.ic_launcher);
```
## 目的：
简化在程序开发中加载图片的操作流程，你只需要在xml中引用该自定义视图，调用加载图片的方法即可。
## 源码地址：
Github: https://github.com/mrljdx/LoadImageView

## 扩展性：
在源码中除了自定义NetWorkImageView外，还定义了一个RoundImage继承NetWorkImageView,实现了一个显示圆形的网络图片的自定义View。其他的形状可以根据自己的需要修改。

## 效果图：
![NetWorkImageView](http://img.blog.csdn.net/20150416205612954)

