# Mac下配置Apktool反编译环境

在Windows下配置Apktool环境相对于Mac要简单很多,这篇文章主要记录自己在Mac下配置Apktool的一些过程,仅供参考.

## 官方安装介绍
地址:http://ibotpeaches.github.io/Apktool/install/
### 检查Java环境是否Ready

> java -version

<!--more-->

### 环境配置

1. 下载 ``apktool`` 和 ``apktool.jar`` 文件。[点击下载apktool](https://bitbucket.org/iBotPeaches/apktool/downloads)
2. 拷贝这两个文件到 ``/usr/local/bin`` 目录下
3. 设置文件为可执行文件(获取root权限)
> chmod +x apktool
> chmod +x apktool.jar

## 下载JD_GUI
JD_GUI 主要是为了方便查看 dex2jar 转换的jar文件结构和部分代码(未混淆)
> jd_gui下载地址:http://jd.benow.ca/ [点击下载](http://jd.benow.ca/)

## 下载 dex2jar
dex2jar主要用于将解压后的Apk包中classes.dex转换为classesdex.jar包便于jd_gui查看
> dex2jar下载地址:http://sourceforge.net/projects/dex2jar/files/ [点击下载](http://sourceforge.net/projects/dex2jar/files/)

## dex2jar使用说明
使用说明地址:http://sourceforge.net/p/dex2jar/wiki/UserGuide/
### For Linux

> // For Linux, Mac OSX, Cygwin
> sh /your_dex2jar_dir/dex2jar-version/d2j-dex2jar.sh /your_apk_dir/someApk.apk

### For windows

> // For Windows
> C:\dex2jar-version\d2j-dex2jar.bat  someApk.apk


### 配置Mac下地dex2jar环境
为了方便在Mac中方便的使用dex2jar,可以将dex2jar配置到环境变量中

> 设置为执行文件
> chmod +x d2j-dex2jar.sh d2j_invoke.sh
> //将d2j-dex2jar.sh 添加到环境变量
> open -e ~/.bash_profile
> //添加
> export PATH=$PATH:~/HackApk/tools/dex2jar

## Android Apk反编译步骤
1. 使用Apktool反编译.apk文件,目的是获取未被混淆的资源文件(除混淆资源文件/加壳的apk外)
> apktool d your.apk

2. 使用解压软件解压.apk文件,获取classes.dex文件,使用dex2jar转换为可以使用jd_gui打开的jar文件
> d2j-dex2jar.sh classes.dex

生成文件 classes-dex2jar.jar ,使用jd_gui打开即可.

## END
仅记录~

---------------------




