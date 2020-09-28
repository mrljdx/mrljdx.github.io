# Picasso&Glide&Fresco比较

> 前言:在Android开发中,图片加载OOM一直困扰着很多开发者,在各种不合理的设计之下也容易导致图片加载OOM的问题,目前开源的比较常用的图片加载库也很多,比如老牌的UIL,Volley,AQuery还有比较优秀的Picasso,Glide,Fresco等.
本文仅简单地比较Fresco&Glide&Picasoo,如有错误还请斧正.

<!--more-->

## [**Picasso**](http://square.github.io/picasso/)
   由Square公司开源的一款图片加载和缓存的库,不过Picasso不支持磁盘缓存.也就是说如果想要做磁盘缓存的话需要另外想办法.(可以利用[JakeWharton/DiskLruCache](https://github.com/JakeWharton/DiskLruCache))
## [**Glide**](https://github.com/bumptech/glide)
   一款和Picasso类似的图片加载和缓存的开源库.虽然在函数定义和调用上和Picasso相差无几,但是Glide确实在性能方面比Picasso要好,值得注意的是Glide库仅支持Android 2.3.3及以上的版本.(PS:目前市面上貌似也很少看到Android 4.1以下的机型了,所以版本向下兼容可以不用担心.)

**关于Glide和Picasso的比较文章推荐:**
[<font color="#98FB98">Introduction to Glide, Image Loader Library for Android, recommended by Google</font>](http://inthecheesefactory.com/blog/get-to-know-glide-recommended-by-google/en)

## [**Fresco**](http://frescolib.org/)
   一款由Facebook今年推出的一个强大的图片加载库,Fresco 中设计有一个叫做 image pipeline 的模块。它负责从网络，从本地文件系统，本地资源加载图片。为了最大限度节省空间和CPU时间，它含有3级缓存设计（2级内存，1级文件）。Fresco 中设计有一个叫做 Drawees 模块，方便地显示loading图，当图片不再显示在屏幕上时，及时地释放内存和空间占用。Fresco 支持 Android2.3(API level 9) 及其以上系统。(摘自:http://fresco-cn.org/)
   有关Fresco的使用教程请绕道:http://fresco-cn.org/docs/index.html#_
   
   ** Fresco相比其他图片库如Picasso，UIL，Glide相比，所具有的最重要的几个独有特性：**
   
   > 
   * 在Android 5.0以下系统，图片不存储在Java heap，而是存储在ashmemheap，中间的字节buffer同样位于native heap。使应用有更多内存空间，降低OOM风险，减少GC次数。
   渐进式的JPEG呈现。
   * 图片可以在任意点进行裁剪，而不是中心，即自定义居中焦点。
   * JPEG可以在native进行resize，避免了在缩小图片时的OOM风险。
   * 支持Gif和WebP。
   
## Fresco&Glide&Picasso 比较
Facebook也提供了,关于Fresco和其他图片加载器的比较例子.下载地址:[Fresco](https://github.com/facebook/fresco)

** 加载本地图片:**
![Fresco&Glide&Picasso](http://7u2j5d.com1.z0.glb.clouddn.com/cmp_fgp.png)
由于图片数量比较少,很难看出差别,不过可以发现,在一般的情况下,Fresco和Glide在内存占用上,Fresco更优,这是由于Fresco将图片放置到一块特殊的Native内存，这块内存的管理是由Fresco进行的，所以Fresco的体积比较庞大，会为应用增加接近2M的体积，这算是Fresco的一个缺点.

** 加载网络图片: **
![net](http://7u2j5d.com1.z0.glb.clouddn.com/net_fgp.png)
在加载网络图片时,由于机型和网络环境不同,可能也会导致现实的数据有问题,不过在比较的时候,可以发现,Fresco在加载网络图片时,刚开始Heap内存占用也是58M左右,不过随着时间的推移,Heap内存占用的值会变大,可能是由于Fresco内存释放存在问题导致的.可见Fresco虽然在加载图片上独辟蹊径但是并没有传说中那么完美.

## 图片加载库的选择

根据自己平时在项目中的体会,在综合包的大小及图片加载器的稳定性后觉得Glide是比较适合用在产品中的图片加载库.
选择的原因如下:

> 1. Glide默认提供配置支持本地图片缓存,缓存的机制是DiskLruCache.可以根据自己的需要,自定义图片缓存的路径.所以在考虑节省用户流量来看可以不考虑Picasso;
2. 虽然Fresco也提供更强大的图片缓存和加载机制,不过在比较之后,感觉Fresco还是有待完善.Glide可以很简单的获取网络图片的Bitmap对象,而Fresco需要通过订阅数据源克隆Bitmap对象的引用才能存储值.操作方式不够简洁和友好.
3. Fresco的库文件中,以最新的0.8.1为例,imagepipeline-0.8.1.aar光包得大小就有3.5M ,而Glide包的大小为465K为了让Apk包得体积更小,所以考虑使用Glide.

   
