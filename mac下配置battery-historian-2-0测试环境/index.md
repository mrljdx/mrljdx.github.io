# Mac下配置Battery Historian 2.0测试环境

## 前言
在介绍本文之前扯点淡，就是最近AlphaGo和李世石的围棋比赛，今天最后一场。估计AlphaGo拿四杀应该没问题。有一段时间没写写东西了，听说Google早就弄了个Battery Historian的电量测试神器，今天来介绍一下，这款基于Go语言构建的Android设备电量用量测试分析工具~ 
本文首发于[简书](http://www.jianshu.com/p/bde860ee903c)(一款适合写作的平台)，推荐各位用一下，一款不错的产品！ 好了，不扯了，进入正题~

## Battery Historian 简介

Battery Historian是一款Google开源的分析设备电量用量情况的测试工具，可以通过导出设备的``bugreport``文件，导入到Battery Historian中分析数据。
不过需要注意的是，Battery Historian 2.0 目前只支持分析 Android 5.0及以上的设备，其他版本的暂不支持。
<!--more-->
如果导入Android5.0
设备以下的版本的bugreport进行分析，只会显示一个leagacy的分析图。下面是不同版本的设备的导出结果。
使用``红米Note1s(系统版本4.4.1)`` ，导出后的结果如图：
![红名Note1s](http://upload-images.jianshu.io/upload_images/115071-b471a7e64f8d6f19.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
使用``Nexus 5(系统版本5.1)``导出的分析结果图：
![Nexus5](http://upload-images.jianshu.io/upload_images/115071-b471a7e64f8d6f19.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

Battery Historian 2.0版本目前可以针对每个应用分析应用的其耗电及性能。是测试人员必备的装X神器~

## 配置Go语言环境

Battery Historian电 2.0是基于Go语言完全重写的分析工具，并加入了一些JavaScript库的可视化与平移和缩放功能，显示时间轴上的电池相关的事件。此外，2.0版允许开发者选择一个应用程序，并检查指标具体到所选择的应用程序影响的电池。
所以配置Battery Historian之前需要配置好Go语言环境，下面简单介绍配置的过程和步骤，以及我踩过的坑(说起来都是泪)

### 下载Go安装包
下载地址： https://golang.org/doc/install#install
注意，有些朋友喜欢用brew的命令来安装，貌似brew安装的go版本并不是官方的最新版本。不知道为啥，所以推荐直接点击下载链接下载pkg安装包，安装Go。

### 配置Go语言环境变量
在命令行下，输入

	open -e ~/.bash_profile

添加如下环境配置语句

	##Config Golang environment
	export PATH=$PATH:/usr/local/go/bin
	export GOPATH=$HOME/GoProjects

> 其中 ``GOPATH``中的``GoProjects`` 为个人建立的Go工程文件夹，会用于存放Battery Historian 2.0的项目代码的目录。

保存后，让环境变量修改立即生效

	source ~/.bash_profile

检查 Go 语言环境是否配置成功
	
	go version

查看当前go的版本，本文的Go语言版本是1.6。(PS: 听说 AlphaGo 也主要是用Go写的~)

## 在本地搭建Battery Historian2.0测试环境
	
在本地搭建Battery Historian2.0测试环境比较简单，首先命令行进入到GoProjects目录下，依次执行以下命令：

	go get -u github.com/golang/protobuf/proto

	go get -u github.com/golang/protobuf/protoc-gen-go

	go get -u github.com/google/battery-histrizan

	cd $GOPATH/src/github.com/google/battery-historian/

	bash setup.sh

	go run cmd/battery-historian/battery-historian.go

按照以上步骤，即可完成本地Battery Historian2.0测试环境的搭建，目录结构如下：
![目录结构](http://7u2j5d.com1.z0.glb.clouddn.com/untitled.png)

## adb导出bugreport并分析
导出设备的bugreport命令也比较简单，使用如下命令即可将设备的bugreport信息导出bugreport.txt文件中：

	adb bugreport > bugreport.txt

对应的``bugreport.txt``文件位于执行当前命令的根目录下。然后具体的参数本文就不详解了，看参考资料中的官方文档，当然本文只是简单的介绍环境配置，关于如何结合LeakCanary做内存泄露方面的分析，还有一些高级功能就不一一介绍了，看官方文档介绍。

## 参考资料

* [Battery Historian Github](https://github.com/google/battery-historian)
* [Go in Action实战开发](https://github.com/astaxie/Go-in-Action)

## END

由于个人能力有限，文中如有纰漏还请斧正，多多交流哈，这工具怎么才能用好，得花点时间好好琢磨一下，开卷有益。^_^

-------------------------


