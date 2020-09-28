# Android应用开发框架经验小结

## 前言
在Android应用开发过程中一般会涉及到如下几个方面的问题：
1.多张大图的加载OOM
2.本地数据库存储
3.事件/消息的传递
4.在非Root设备上查看应用数据
5.网络请求，Json/XML数据解析
6.方法数超过65535限制
7.混淆代码

<!--more-->

## AGES开发框架
AGES开发框架是我根据``AndroidAnnotations+GreenDao+EventBus+SpringAndroid``的首字母命名的。下面分别介绍这个框架各个开源库：

### AndroidAnnotaions(注解库)
使用注解简化代码，省去写``findViewById``的麻烦，主要是简化了异步的调用方式，使用@Background和@UiThread可以极大的简化工作线程和主线程之间的切换。
>官方网址：http://androidannotations.org/

### GreenDao(开源数据库)
简化Sqlit数据的操作，比OrmLite数据库基于反射不同的是，GreenDao是基于映射，在编译阶段生成数据库相关的代码，这点类似于AAs(AndroidAnnotations的缩写)。而且比较成熟，有很好的可扩展性，设计的非常不错，在很多公司的项目中被使用到。另外兼容Android 2.3及以上设备。
>官方网址：http://greenrobot.org/greendao/

### EventBus(事件传递解耦库)
EventBus一款将事件的发送方和接收方解耦的库，在项目集成中也非常方便，特别是经过开源社区的完善，目前EventBus3的体积和性能都有了很好的提升，可以说是事件传递解耦的库不错选择。
>官方网址：http://greenrobot.org/eventbus/

### SpringAndroid(网络请求框架)
一款设计不错的Android网络请求框架，目前已经支持OkHttp 3.x。关键是在配合AndroidAnnotations使用时，写出来的Rest-API会非常优雅。具体可以参考：[Rest-API](https://github.com/excilys/androidannotations/wiki/Rest-API#rest)
目前最新支持OkHttp3的SpringAndroid版本在Github上:[SpringAndroid](https://github.com/spring-projects/spring-android) 有这方面需要的朋友可以去看看提交的更新和源码。
> 官方网址：http://projects.spring.io/spring-android/

## AGES开发框架的好处
这篇文章主要是讲关于AGES开发框架在平时Android应用开发中的一些经验，首先因为这套框架使用AndroidAnnotaitons作为注解，AAs会为每个注解的Application、Activity、Fragment、Service、Receiver、View、ViewGroup、JavaBean等类生成一个``final``类型的子类，子类的命名规则为父类后面加下划线("\_")，这里不具体讲AAs的具体使用方法，如果要讲可能会让文章又臭又长。（具体使用规则参考：[Avalaible Annotations](https://github.com/excilys/androidannotations/wiki/AvailableAnnotations)）
下面讲一下这个框架的好处：
1. 和Butterknife的注解框架的功能类似，但是性能要比ButterKnife要好，而且在工作线程和主线程切换使用@Background和@UiThread注解切换非常方便便捷。
2. AAs+SpringAndroid极大的简化Rest Api，感觉比起Retrofit提供的还要简单。这里用我项目里的截图感受一下：
![BaseApi.png](http://upload-images.jianshu.io/upload_images/115071-12af6aead3ecf139.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
具体的代码调用也非常简单：
![代码调用.png](http://upload-images.jianshu.io/upload_images/115071-4e8cf6788a0e17a4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

3. 使用GreenDao可以极大的简化数据库的操作流程，由于GreenDao封装了一套数据库增删改查的Api，具体参考官网使用方法。
4. 使用EventBus解耦事件传递回调，特别是在处理Http请求、Fragment和Activity之间的事件传递、后台Service将数据传递给Activity/Fragment等，EventBus在任何一段代码中都可以post一个事件。具体使用请参考官方网站。

## 开发中涉及其他内容
### 图片加载
推荐使用Glide或Picasso，虽然Fresco也不错，不过感觉Glide和Picasso更加轻量，毕竟包的体积Fresco更大，如果为了性能考虑可以使用Fresco。另外需要注意的是Picasso并不提供图片磁盘缓存功能而Glide在这方便考虑的更加全面。

>* 关于Glide和Picasso的比较，可以参考文章：[Introduction to Glide, Image Loader Library for Android, recommended by Google](http://inthecheesefactory.com/blog/get-to-know-glide-recommended-by-google/en)
* 关于Glide&Picasoo&Fresco的比较可以参考我的博客：[Picasso&Glide&Fresco比较](http://mrljdx.com/2015/12/22/Picasso-Glide-Fresco%E6%AF%94%E8%BE%83/)

### 应用调试
现在很多公司的Android调试设备都没有Root，而且现在很多人也没那个闲功夫去Root自己的手机。以前流行Root是因为一些厂商的Rom做的很垃圾，有刷机的需求，不过现在随着各家厂商注重定制的系统体验和Android系统安全的完善，Root手机的越来越少了。
那么在非Root环境下可以使用Facebook开源的一款调试神器：[Stetho](https://github.com/facebook/stetho) 集成在你的项目中之后可以利用``chrome://inspect ``来查看应用内的数据库，甚至可以修改数据库内容，做一些sqlite数据库命令操作。
此外，结合OkHttp配置可以同时查看你的网络请求，具体关于Stetho的使用可以参考简书的文章:[[译] 如何更高效地使用 OkHttp](http://www.jianshu.com/p/715940dc309b)，这里就不深入说明了。

### 内存泄露检测
Square公司开源的[Leakcanary](https://github.com/square/leakcanary)是目前一种比较好的内存泄露检测工具，集成在项目中也非常简单。可以说这是一款内存泄露检测神器，在运行App的过程中，会生成对应的内存泄露堆栈跟踪结果显示在手机中，便于解决潜在的内存泄露问题。可以参考简书文章：
[利用 LeakCanary 来检查 Android 内存泄漏](http://www.jianshu.com/p/0049e9b344b0)

### 函数式编程RxJava的引用
RxJava是目前在Android开发中比较流行的函数式编程的解决方案之一，结合RxAndroid+Retrofit（默认支持RxJava）+OkHttp可以构建一个很好的基于流的网络请求库，避免了多重网络请求的回调问题，特别是在多线程编程方面，个人觉得不应该在整个项目工程中都滥用RxJava，毕竟在平时的工作中是一个团队协作，而RxJava的学习曲线确实比较麻烦。具体文章可以参考：
1. [RxJava开发精要系列文章](http://www.devtf.cn/?p=1221)：一系列的文章，译自国外技术博客，内容还行，不过还是需要自己实战的好。
2. [给Android开发者的Rxjava详解](http://gank.io/post/560e15be2dca930e00da1083) ： 适合对Rxjava不了解的同学作为入门的参考文章。
3. [RxJava官方教程](http://reactivex.io/tutorials.html) ： RxJava官方教程，适合英语能力不错的猿学习，毕竟官方的内容讲解的更加到位，适合全面深入了解RxJava。
4. [JavaFunctional Programming](https://realm.io/cn/news/droidcon-gomez-functional-reactive-programming/)：足足的干货，让你了解什么是函数式编程，以及它的好处。推荐多看几遍。

 
## END
以上内容为我个人的开发经验小结，由于个人能力有限，文中如有有遗漏或错误之处还请斧正。在后续的文章中，我会对AndroidAnnotations、GreenDao、EventBus以及SpringAndroid这四个开源框架的源码进行解析，加深对这个框架的理解以及摸清这个框架所存在的瓶颈和不足之处加以修缮。如果对这个开发框架感兴趣的可以和我交流，关注我的简书/[博客](http://mrljdx.com)查看后续更新的内容，多多交流。

---------------------

