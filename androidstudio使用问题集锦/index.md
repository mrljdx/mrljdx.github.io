# AndroidStudio使用问题集锦


## 配置AndroidStudio让编译更快

### 在个人.gradle文件夹下添加 gradle.properties 文件：
	
文件内容如下:
	
	org.gradle.daemon=true

<!--more-->

### 修改Android项目更目录中的gradle.properties文件：
文件内容如下:
	
	# Project-wide Gradle settings.

	# IDE (e.g. Android Studio) users:
	# Settings specified in this file will override any Gradle settings
	# configured through the IDE.

	# For more details on how to configure your build environment visit
	# http://www.gradle.org/docs/current/userguide/build_environment.html

	# The Gradle daemon aims to improve the startup and execution time of Gradle.
	# When set to true the Gradle daemon is to run the build.
	# TODO: disable daemon on CI, since builds should be clean and reliable on servers
	org.gradle.daemon=true

	# Specifies the JVM arguments used for the daemon process.
	# The setting is particularly useful for tweaking memory settings.
	# Default value: -Xmx10248m -XX:MaxPermSize=256m
	org.gradle.jvmargs=-Xmx2048m -XX:MaxPermSize=512m -XX:+HeapDumpOnOutOfMemoryError -Dfile.encoding=UTF-8

	# When configured, Gradle will run in incubating parallel mode.
	# This option should only be used with decoupled projects. More details, visit
	# http://www.gradle.org/docs/current/userguide/multi_project_builds.html#sec:decoupled_projects
	org.gradle.parallel=true

	# Enables new incubating mode that makes Gradle selective when configuring projects.
	# Only relevant projects are configured which results in faster builds for large multi-projects.
	# http://www.gradle.org/docs/current/userguide/multi_project_builds.html#sec:configuration_on_demand
	org.gradle.configureondemand=true

## 解决方法数超过65535问题

### 错误描述 
一般情况下一款简单的Android应用的方法数不会超过65535，但是随着大量优秀的开源项目导入到项目中，方法数一般会超过这个数，这时如果你再编译应用，则会报错。

	NoClassDefFoundError  //某个方法找不到
	//错误情况大致如下
	Could not find class ‘xxxx’ , referenced from method ‘xxxxx’ 

### 解决办法
[<font color="#aaaa">stackoverflow地址</font>](http://stackoverflow.com/questions/27698287/noclassdeffounderror-with-android-studio-on-android-4?lq=1) [<font color="##eeee">Google官方解答</font>](http://developer.android.com/tools/building/multidex.html)

** (1)在app/build.gradle中添加: **
	
	dependencies {
		//multidex support
    	compile 'com.android.support:multidex:1.0.0'
	}

** (2)让自定义的类继承自 MultiDexApplication，并修改onCreate()方法如下： **
	
	@Override
    public void onCreate() {
        //MultiDex 支持 65535 方法数量限制，注意，此方法要在super.onCreate()之前调用
        MultiDex.install(getApplicationContext());
        super.onCreate(); 
    }

## 三方包的导入及build.gradle配置

	android{

		//添加.so库依赖
	    sourceSets {
	        main {
	            jniLibs.srcDirs = ['libs']
	        }
	    }
	}

	// 为了让编译器能在libs 目录下找到引用的 .aar 文件
	repositories {
	    flatDir {
	        dirs 'libs' //this way we can find the .aar file in libs folder
	    }
	}
	

	dependencies {
		//引用远程仓库包
		compile 'com.squareup.picasso:picasso:2.5.2' 
		//引用app/libs 目录下包
		compile files('libs/gson-2.2.4.jar')  
		//引用项目库文件 文件结构  /app /stickyListView ，注意配置 setting.gradle 文件 : include ':app',':stickyListView'
		compile project(':stickyListView') 
		//引入aar库文件
		compile(name:'aar_library_name', ext:'aar') //改.aar文件位于app/libs目录下，并且已经配置repositories
	}

## Android常用命令

### ADB相关命令

** (1)获取连接到的手机序列号 **

	adb get-serialno

** (2)查看当前连接的设备**

	adb devices

** (3)重启adb server**
  
  	adb kill-server
  	adb start-server

### 秘钥相关命令

** (1)查看keystore的信息 **

	keytool -list -v -keystore xp.jks
	或者：
	keytool -list -keystore (keystore文件) -alias (key的别名) -v

** (2)查看keystore的公钥证书信息 **
	
	keytool -list -keystore (keystore文件) -alias (key的别名) -rfc

	（注：获取Base64格式的公钥证书，RFC 1421）

** (3)查看apk的签名信息 **

	jarsigner -verify -verbose -certs <your_apk_path.apk>

** (4)生成keystore **
创建keystore，需要用到keytool.exe (位于jdk_xx\jre\bin目录下)，具体做法如下：

	keytool -genkey -alias mykey -keyalg RSA -validity 40000 -keystore demo.keystore

	#说明：
	#    -genkey 产生密钥
	#    -alias mykey 别名 mykey
	#    -keyalg RSA 使用RSA算法对签名加密
	#    -validity 40000 有效期限4000天
	#    -keystore demo.keystore

** (5)对apk签名 **

使用产生的keystore对apk签名，使用到的是jarsigner.exe ，该工具位于jdk_xx\bin目录下，命令如下：

	jarsigner -verbose -keystore demo.keystore -signedjar test_signed.apk test.apk mykey

	#    test_signed.apk是签名之后的文件
	#    test.apk是需要签名的文件

另外需要注意的是，如果你的jdk版本在1.7以上，你在对apk签名时，需要加上这个参数：
	
	-digestalg SHA1 -sigalg MD5withRSA

否则同样会出现：Failure [INSTALL_PARSE_FAILED_NO_CERTIFICATES]的错误。

	


-------


