# AndroidManifest配置文件简介

在Android开发接触的最多的可能就是AndroidManifest.xml这个配置文件了，这里配置文件中包括了应用程序中的大部分信息，系统在运行代码时，需要知道这些基本信息。比如开发中的Activity、Service、Broadcast都需要在这里定义。如果用到了一些系统自带的服务比如拨号、应用安装、GPS定位等服务也需要在这里声明。
<!--more-->
下面看一段注释过的``AndroidManifest.xml``文件:
```xml
<?xml version="1.0" encoding="utf-8"?>
<!--xmlns:android指定命名空间；应用包；版本号；版本名称-->
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.mrljdx.test"
    android:versionCode="1"
    android:versionName="1.0" >
    <!--用户权限-->
	<uses-permission />
	<!--自定义权限许可-->
    <permission ></permission>
    <permission-tree />
    <permission-group ></permission-group>
    <instrumentation ></instrumentation>
    <!--声明应用程序要求最低的Api级别-->
    <uses-sdk />
    <uses-configuration />
    <uses-feature />
    <supports-screens />
    <!--定义应用名称，图标等-->
    <application >

		<!--注册Activity信息 activity-->
        <activity >
            <!--意图过滤器（活动、服务和广播）-->
            <intent-filter >
                <action />
                <category />
                <data/>
            </intent-filter>
            <meta-data />
        </activity>
        <activity-alias/>
        <!--注册服务信息 service-->
        <service >
            <intent-filter >
                <action/>
                <category/>
                <data/>
            </intent-filter>
            <meta-data/>
        </service>
        <!--注册广播接收者broadcast receiver-->
        <receiver >
            <intent-filter >
                <action />
                <category />
                <data/>
            </intent-filter>
            <meta-data/>
        </receiver>
        <!--注册内容提供者content_provider-->
        <provider >
            <grant-uri-permission />
            <meta-data/>
            <path-permission/>
        </provider>
        <!--列举application所需链接库，比如测试时，引入android.test.runner库-->
        <uses-library/>
        
    </application>
</manifest>
```
Android程序执行流程：
1. 点击应用图标时，系统将这个事件包装成一个Intent，该Intent包含两个参数，代码：
>｛action:"android.intent.action.Main",category:"android.intent.category.LAUNCHER"｝

这个意图被传递给应用之后，会在AndroidManifest.xml配置清单中查找，找到与意图过滤器所在的Activity元素，再根据name属性，来找到对应的Activity类。
2. 接着Android系统创建该Activity类的实例对象，对象创建完成之后，执行onCreate方法，此onCreate方法是重写其父Activity的onCreate方法。此方法用来初始化Activity的实例对象。代码：
```java
 @Override
  public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.main);
  }

3.``super.onCreate(savedInstanceState);`` 的作用是调用父类Activity的onCreate方法来实现对界面的画图绘制工作。在实现自己定义的Activity子类的onCreate时，一定要记得调用这个方法，以确保能正确绘制界面。
4. ``setContentView(R.layout.main)`` 的作用就是加载一个界面。该方法中传入一个参数``R.layout.main``，其含义是R.java类中静态内部类``layout``常量``main``的值，而该值指向``res`` 目录下的``layout`` 子目录下``main.xml`` 文件的标识符。因此代表着显示``main.xml`` 定义的界面。

以上就是一个应用程序启动的整个流程。了解AndroidManifest的内容以及启动的过程可以更加清楚Android应用开发的一些问题。
