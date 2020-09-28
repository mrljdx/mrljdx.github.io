# 浅聊Android布局优化


## 前言
在Android开发中，常用的布局就只有FrameLayout&LinearLayout&RelativeLayout，这些布局由于年代恒久远，一直流传下来。但是也存在着一些遗留问题，在Activity展开布局时用到setContentView()方法也许大家并不陌生。在使用这个方法时发生了什么呢？
<!--more-->
现在大多数开发使用的都是``android.support.v7``的包吧，具体的Activity实现，在``android.support.v7.app.AppCompatDelegateImplV7`` 中setContentView()代码如下：  

    @Override 
    public void setContentView(int resId) { 
            ensureSubDecor(); 
            ViewGroup contentParent = (ViewGroup)       
            mSubDecor.findViewById(android.R.id.content);         
            contentParent.removeAllViews(); 
            LayoutInflater.from(mContext).inflate(resId, contentParent); 
            mOriginalWindowCallback.onContentChanged(); 
     }
首先contentParent会移除所有的布局文件，然后LayoutInflater会根据当前的context来展开布局并填充到contentParent中。
可以看到一个布局在Activity中展开取决于inflate的resId，再看看inflater相关的代码：   
        
      public View inflate(XmlPullParser parser, @Nullable ViewGroup root) { 
          return inflate(parser, root, root != null); 
      } 
      public View inflate(@LayoutRes int resource, @Nullable ViewGroup root, boolean attachToRoot) { 
      final Resources res = getContext().getResources(); if (DEBUG) { 
          Log.d(TAG, "INFLATING from resource: \"" + res.getResourceName(resource) + "\" (" + Integer.toHexString(resource) + ")"); } 
          final XmlResourceParser parser = res.getLayout(resource); 
          try { 
                return inflate(parser, root, attachToRoot); 
          } finally { 
                parser.close(); 
          } 
      }  

首先Android读取应用的资源数据(APK文件内，存储在内部存储或者SD卡中)；
其次使用``XmlResourceParaser``来解析资源数据，展开布局； 
最后将展开后的布局添加到Activity的顶层视图。 

这里花费的时间取决于布局的复杂性，资源数据越大，解析越慢，而更多的组件实例也让布局实例化变慢。

## 现状

一般布局通常在Activity的onCreate()方法中展开，这里花费的时间会直接影响Activity的启动时间。
所以尽量减少布局花费的时间可以提升应用的界面流畅度。
现在大多数的方案，减少创建对象数量，使用``<merge>``标签减少``<FrameLayout>``包裹的布局对象，或者是使用``<ViewStub>``推迟创建对象。
不过现在产品界面复杂，光靠这几个方法也不能完全解决对象的数量创建过度的问题，比如一些布局需要按照比例来适配，那么只能利用LinearLayout的``layout_weight``布局属性来实现。
其中会增加一大堆布局对象，显然不符合我们对布局优化的需求。
Google官方考虑到这种需求，也发布了解决这些问题的包[Android Support Library](https://developer.android.com/topic/libraries/support-library/features.html#recommendation)（需要翻墙）。其中包含一个[Percent Support Library](https://developer.android.com/topic/libraries/support-library/features.html#percent) 顾名思义，是一个支持百分比布局的包。

在Gradle中配置如下：  

        com.android.support:percent:23.3.0

即可获取Android百分比布局的包。关于这个包的使用，可以参考文章[《Percent Support Library: Bring dimension in % to RelativeLayout and FrameLayout》](https://inthecheesefactory.com/blog/know-percent-support-library/en) 具体的使用就不再本文追叙了。毕竟有一篇这么好的参考文章，我就不添乱了。

另外，Github上也有相关的开源示例项目：[Android Precent Support lib Sample](https://github.com/JulienGenoud/android-percent-support-lib-sample) 请自行查看。

## 优化布局的使用建议

### 避免使用嵌套线性布局

嵌套的线性布局存在两个问题：
* 嵌套线性布局会深化布局层次，从而导致布局和按键处理变慢；
* 造成很多用于定位的无用对象；

解决方案：
*  对于不要求布局中的组件需要使用百分比的，可以优先考虑使用RelativeLayout来替代LinearLayout，减少布局中的组件数量。
* 对于要求使用百分比的，可以考虑使用PercentRelativeLayout。

### 使用``<merge>``标签

通常，在使用``<merge>``标签来合并布局时，其布局的顶层元素是一个FrameLayout可以考虑使用``<merge>``标签，在Activity中，其默认的顶层布局就是一个FrameLayout，对应的id为``android.R.id.content`` 。

### 使用``<include>``重用布局

重用布局，在布局展开时，``<include>``的布局是动态处理的，因为包含是在编译时完成的，但是Android并不知道这里包含的布局具体对应哪种屏幕情况（比如横屏和竖屏布局）。 
其实严格来讲，``<include>``不属于布局的优化，属于代码结构的优化，可以帮助开发人员减少很多重复的代码，减少包的体积。

### 使用``<ViewStub>``推迟加载布局

推迟初始化是一个很好的技术，可以推迟实例化在需要用到的时候再实例化，提高应用的性能，还能节省内存。关于ViewStub的使用有很多文章介绍，这里本着不添乱的精神，简单的介绍一下``<ViewStub>``和``<include>`` 的区别。
首先，这两者都需要使用``android:layout`` 属性来定义需要引入的布局文件，其次``<include>``在加入布局文件不能实现延迟加载，而``<ViewStub>``在未展开布局之前在视图中是不可见的，需要调用inflate()或者setViewVisibility(View.VISIBLE)来显示，并且在显示的同时，会在父布局中移除``<ViewStub>``，由展开的布局所替代。

此外，``<ViewStub>``中还得定义一个``android:inflatedId``供Activity找到展开后的布局实例。

## 布局优化工具

为了优化已有的布局，Android SDK提供了两个易于使用的工具：HierarchyViewer 和 LayoutOpt。这些工具位于SDK的tools目录。
不过在使用HierarchyViewer时，我就碰到一个问题，就是怎么都不显示所谓的高端大气上档次的布局层次依赖关系图，解决办法：集成ViewServer开源项目，地址：https://github.com/romainguy/ViewServer

另外，在最新的sdk中，LayoutOpt貌似被lint工具给替代了，这个工具比较牛逼的地方就在于静态代码检查，会提出一些优化方案，比如使用HashMap的地方，会告诉你使用SparseArray，一些布局优化的建议也会给出。

## END

由于个人能力有限，文中如有疏漏之处还请斧正~多多交流~

----------------------
