# 利用反编译学习Android

自从2014年底到2015年中,全民创业的热潮就已经席卷全国了,一大批新的创业公司在北上广萌芽,也造成了大量的开发人员需求.扯远了,今天不谈创业潮,聊聊如何通过反编译学习Android.
本文只是个人对于学习的一点看法,大神请绕道.

如今市面上有很多优秀的App.这些App比较适合我们拿出来研究,去了解他们使用的技术(用了哪些开源库,^_^).
那么如何去了解呢?
反编译~

<!--more-->

基本上经过反编译之后的代码,就能大致的了解其软件结构了.
下面以一些app为例简单地说明,仅作为学习交流,请勿随意传播,造成不良影响.

## 搭建反编译环境
由于搭建反编译环境比较简单,这里就不详细说明了.
可以参考文章 [Mac下配置Apktool反编译环境](http://mrljdx.com/2015/12/30/Mac%E4%B8%8B%E9%85%8D%E7%BD%AEApktool%E5%8F%8D%E7%BC%96%E8%AF%91%E7%8E%AF%E5%A2%83/)

## 反编译
常用的反编译命令:

* 获取反编译后的资源文件和smail代码等

    ``apktool d xxx.apk``  

* 将dex转换为jar包在jd_gui中查看

    ``d2j-dex2jar.sh classes.dex ``
    
## 分析反编译结果

### 常用开源库

* 注解库[Butterknife](http://jakewharton.github.io/butterknife/)
* 网络请求框架[Retrofit](http://square.github.io/retrofit/) (支持Rxjava)
* 网络请求库[okHttp](http://square.github.io/okhttp/)
* Json解析库[Gson](https://github.com/google/gson)
* okHttp库用到 [okio](https://github.com/square/okio)
* ReactX函数响应式编程框架 [RxJava](https://github.com/ReactiveX/RxJava)
* Rx异步框架同上 [RxAndroid](https://github.com/ReactiveX/RxAndroid)
* 图片控制库[PhotoView](https://github.com/chrisbanes/PhotoView)
* 图片加载库[Glide](https://github.com/bumptech/glide)
* 图片加载库[Picasso](http://square.github.io/picasso/)
* Android解耦库[EventBus](https://github.com/greenrobot/EventBus)
* 内存泄露检测工具[eakcanary](https://github.com/square/leakcanary)
* 支持在低版本(API 11 以下)使用Android 属性动画以及3D 旋转动画的框架 [nineoldandroids](http://nineoldandroids.com/)
* 图片毛玻璃效果库[Android StackBlur](https://github.com/kikoso/android-stackblur)
* 网络请求框架 By Google [Volley](https://github.com/mcxiaoke/android-volley)
* Light weight android easing library [Android-Easing](https://github.com/sephiroth74/Android-Easing)

### 常用三方服务

* 百度地图
* 百度推送
* Umeng更新组件&分析
* 环信及时通信
* 阿里妈妈推广sdk
* 阿里支付sdk
* 微信支付sdk

## END

由于反编译的App不多,大概总结了一下,如果有一些不错的App可以一起分析一下其中使用的开源库和使用的技术.
不过目前RxJava+Retrofit+ButterKnife+OkHttp+EventBus的框架貌似被用的很多.
马上要跨年了,撤之~

祝 2016 All is well~

---------------------

