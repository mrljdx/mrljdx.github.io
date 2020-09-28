# Android开发减少Apk大小及统计方法数实践

## 前言
一款好的App首先需要保证功能稳定性（在用户手上不Crash），同时也要考虑节省用户的流量，电量等。不过现在的App市场同类型的App已经很多了，那么如何保证你的产品能在做推广时用户愿意下载呢？就是通过精简APK大小来减少用户的下载成本。

下面就Androdi开发中一些控制APK大小的经验来简单说明一下。
<!--more-->
## APK分析
在开发一款APP时我们都会引用一些Github上的开发库，一款App基础框架如下：

1. Retrofit+OkHttp+OkIO作为网络请求框架；
2. Glide/Fresco/Picasso/AIL/等作为图片加载库；
3. EventBus/Otto 作为事件发布/订阅总线；
4. ORMLite/GreenDao作为数据库框架的普遍较多；
5. Gson/Jackson等JSON解析库；
6. 基于RxJava函数式编程思想构建的精简开发框架（代码不容易看懂）；

以上是笔者总结的大部分比较常用的库，当然有时候还需要集成一些三方平台提供的SDK：

1. Umeng/ShareSDK/LeanCloud等三方统计平台服务（三方登录/数据统计/软件更新等）;
2. 集成微信/支付宝支付SDK；
3. 高德/百度/腾讯地图SDK；
4. 讯飞/百度语音等语音识别/语音合成 SDK；

以上基本上是开发中需要用到的库和文件，而Android作为开发者，在开发中最好尽量避免在Apk中一些不必要的库，因为过多的集成库会导致Apk的方法数超过65535的限制。这时候只能通过MultiDex方法来解决这个问题。
不过使用MutliDex解决方案会导致一些问题：

1. 增加了在较旧和低版本的Android设备上应用加载的时间；
2. 在一些三星设备上，由于厂商设备出厂对应用的内存限制会导致一些问题。比如运行时崩溃。

所以不在万不得已的情况下，不要轻易使用MultiDex方案。应该尽量避免方法数超过65535。

## 解决方案
### 使用dex-method-counts来统计项目方法数量
[dex-method-counts]是一款基于Gradle构建的Java工程，它包含一些Android的构建工具，可以帮助我们统计Android工程中的每个三方库的方法数量并给出详细的数据分析。可以让我们快速定位Apk中方法数的问题。
Github地址：https://github.com/mihaip/dex-method-counts
这个项目是基于IDEA开发开发工具Gradle工程构建的，如果你的电脑中有IDEA软件，可以直接导入项目，传入APK的文件绝对路径作为参数，运行即可。

如果没有IDEA，也可以直接通过Gradle来编译整个工程：

on OSX

> $ ./gradlew assemble
> $ ./dex-method-counts path/to/App.apk

on Windows

> $ gradlew assemble
> $ java -jar path\to\build\jar\dex-method-counts.jar path\to\App.apk

本文在IDEA中编译并执行后得到如下结果（部分片段）：
如图，可以看出整个Apk的方法数为59805，小于65535的数量限制，可以不必使用Multidex。
![图IDEA1](http://7u2j5d.com1.z0.glb.clouddn.com/90FD8B25-8A54-4A15-8794-4BD40D6A1331.png)
在图中可以看出，ShareSDK的方法总计1941个；以com.包名的模块占方法数32327个，其中友盟Update（怎么是alimama额）占用293个，amap(高德地图)占用6003个方法。
![图IDEA2](http://7u2j5d.com1.z0.glb.clouddn.com/QQ20160222-0%402x.png)

### 使用Gradle脚本来计算Apk的大小
在每次编译完Apk后，可以通过脚本自动计算Apk各个版本的大小情况，包括debug-unProguard版本和release-proguard版本。
下面是一段简单的代码，打印app-debug版本和app-unaligned版本的大小：

	task reportApkSize << {
	    def sdkProject = project(':app')
	    def nonSdkProject = project(':app')
	    def footprint = getSizeDifferent(
	            new File("${sdkProject.buildDir}/outputs/apk/app-debug.apk"),
	            new File("${nonSdkProject.buildDir}/outputs/apk/app-debug-unaligned.apk"))
	    println footprint
	}

	def getSizeDifferent(File file1,File file2){
	    return "Debug Size is : "+file1.size()/1024/1024+"M"+" Debug unaligned Size is : "+file2.size()/1024/1024+"M";
	}

通过简单的配置就可以打印Apk的其他版本大小，通过每次发布之前来比较，并结合dex-counts就可以随时的分析和了解自己Apk的大小情况，便于分析和解决问题。

## END
由于作者能力有限，如文中有问题还请指正，开卷有益，欢迎多多交流。

-------------------




