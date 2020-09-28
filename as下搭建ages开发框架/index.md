# AS下搭建AGES开发框架


使用AndroidStudio（以下简称AS）作为首选的开发工具已经很普及了，但是作为一个Eclipse的老手来说，转AS还是需要点时间去适应的。比如以前常用的快捷键，代码提示： ``Alt+/`` 在AS下是``ctrl+alt+space`` 而 查看函数则由之前的鼠标悬浮变成了按``F2``查看，导入包则由之前的``ctrl+shift+o``编程了现在的``alt+enter`` 
说了这么多不适应的快捷键，下面来谈谈如何在AS下搭建AEG开发框架。
<!--more-->
**什么是AEGS开发框架？**
容我装X的解释一下``AndroidAnnotations+GreenDao+EventBus+SpringAndroid`` 的简称AGES。
至于AGES开发框架的使用，以后会专门介绍以及这个框架在项目中实践的例子。由于本文是介绍在AS中初步搭建AGES开发框架，我们还是回归主题，来看看如何搭建AGES开发框架吧。

## 准备工作
+ 下载AndroidAnnotations包: 
	+ [androidannotations-3.2](https://github.com/excilys/androidannotations/wiki/Download)  
	+ [androidannotations-api-3.2.jar](https://github.com/excilys/androidannotations/wiki/Download)
+ 下载GreenDao相关包：
	+  [greendao-1.3.7.jar](http://search.maven.org/#search%7Cga%7C1%7Cg%3A%22de.greenrobot%22%20AND%20a%3A%22greendao%22)
	+ [greendao-generator-1.3.1.jar](http://search.maven.org/#search%7Cga%7C1%7Cg%3A%22de.greenrobot%22%20AND%20a%3A%22greendao-generator%22)
	+ [freemarker-2.3.22.jar](http://search.maven.org/#search%7Cga%7C1%7Cg%3A%22freemarker%22) 
+ 下载EventBus相关包：
	+ [eventbus-2.4.0.jar](de.gsearch.maven.org/#search%7Cga%7C1%7Cg%3A%22reenrobot%22%20AND%20a%3A%22eventbus%22) 
+ 下载SpringAndroid相关包：
	+ [spring-android-core-1.0.1.RELEASE.jar](http://mvnrepository.com/artifact/org.springframework.android/spring-android-core)
	+ [spring-android-rest-template-1.0.1.RELEASE.jar](http://mvnrepository.com/artifact/org.springframework.android/spring-android-rest-template/1.0.1.RELEASE)
 
## 在AS下搭建的步骤
**1. 下载好必须的安装包之后，按照如图所示，添加到AGESFramework工程中。**
为了更加直观的明白jar包是如何添加的，直接贴图。
![工程目录试图](http://img.blog.csdn.net/20150405132718972)
**2. app下的build.gradle文件配置如下：**
```xml
apply plugin: 'com.android.application'
//添加android-apt插件
apply plugin: 'android-apt'

buildscript {
    repositories {
        mavenCentral()
    }
    dependencies {
        classpath 'com.neenbedankt.gradle.plugins:android-apt:1.4'
    }
}
//对android-apt插件的参数定义
apt {
    arguments {
        androidManifestFile variant.outputs[0].processResources.manifestFile //win
        //定义apt的注解包名和工程的包名一致
        resourcePackageName 'com.mrljdx.agesframework'
    }
}
//设置默认编码格式
tasks.withType(JavaCompile) { options.encoding = "UTF-8" }
android {
    compileSdkVersion 22
    buildToolsVersion "22.0.1"

    defaultConfig {
        applicationId "com.mrljdx.agesframework"
        minSdkVersion 15
        targetSdkVersion 22
        versionCode 1
        versionName "1.0"
    }
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }

    packagingOptions {
        exclude 'META-INF/license.txt'
        exclude 'META-INF/notice.txt'
    }
}

dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    compile 'com.android.support:appcompat-v7:22.0.0'
    apt files('compile-lib/androidannotations-3.2.jar')
    compile files('libs/androidannotations-api-3.2.jar')
    compile files('libs/eventbus-2.4.0.jar')
    compile files('libs/spring-android-core-1.0.1.RELEASE.jar')
    compile files('libs/spring-android-rest-template-1.0.1.RELEASE.jar')
    compile files('libs/greendao-1.3.7.jar')
}

```
> 注：在配置完build.gradle文件之后需要到``AndroidManifest.xml`` 就可以按照AndroidAnnotations的官方文档开发了。官方文档地址：[AA Doc](http://androidannotations.org/)

**3 以上两步仅配置完成了AGES中的A、E、S而GreenDao需要在AS当前工程建立一个java library模块，来专门负责生成需要的Dao类**
如图所示，greendao的模块，依赖两个包，一个是``freemarker-2.3.22.jar`` 用来生成Dao类模版，``greendao-generator-1.3.1.jar`` 用于提供生成Dao类的Schema和Entity。
![工程依赖](http://img.blog.csdn.net/20150405133101322)
在创建greendao模块之后，可以创建一个数据库类的生成类，如下所示，以添加一个User表为例：
```java
package com.mrljdx.greendao;

import de.greenrobot.daogenerator.DaoGenerator;
import de.greenrobot.daogenerator.Entity;
import de.greenrobot.daogenerator.Schema;

public class AGESDao {

    public static void main(String args[]) throws Exception {
        Schema schema = new Schema(1, "com.mrljdx.agesframework.dao");

        UserEntity(schema);

        new DaoGenerator().generateAll(schema, "app/src/main/java"); //生成的目录地址
    }

    private static void UserEntity(Schema schema) {
        Entity user = schema.addEntity("User");
        user.addIdProperty().notNull().autoincrement();
        user.addIntProperty("sysno").notNull();
        user.addStringProperty("usname");
        user.addStringProperty("name");
        user.addStringProperty("nkname");
        user.addStringProperty("mobile");
        user.addStringProperty("email");
        user.addStringProperty("avatar");
        user.addStringProperty("birthday");
        user.addIntProperty("gender");
        user.addIntProperty("level");
    }
}
```
运行这段代码之后，就会在工程目录com.mrljdx.agesframework下生成一个dao package，存放数据库相关的类文件。如图所示：
![Dao图](http://img.blog.csdn.net/20150405133456832)
>GreenDao的使用文档和分析，网上很多，如果英文不错，推荐直接看官方文档和官方的Github，包括对数据库性能的比较，GreenDao比起其他的ORM数据库框架的性能要好。当然在实际项目中，还是要根据项目需要来选择。
>官方文档地址： [greenDao -Android ORM for SQLite ](http://greendao-orm.com/) 
> GreenDao Github地址: [greenDao](https://github.com/greenrobot/greenDAO)

## 相关技术文档汇总
**EventBus Github开源地址：**
https://github.com/greenrobot/EventBus

**AndroidAnnotations Cookbook:**
https://github.com/excilys/androidannotations/wiki/Cookbook

**Spring Android RestTemplate Module Documentation:**
http://docs.spring.io/spring-android/docs/1.0.x/reference/htmlsingle/#rest-template

**GreenDao官网**
http://greendao-orm.com/

**GreenDao Github**
https://github.com/greenrobot/greenDAO

## 小结
熟悉了在AS下如何构建gradle文件，就能很容易的从Eclipse转到AS下开发。关于AS的开发资料有很多，可以看我的另外一篇博客[《AndroidStudio参考资料整理》](http://mrljdx.com/2015/02/22/AndroidStudio%E5%8F%82%E8%80%83%E8%B5%84%E6%96%99%E6%95%B4%E7%90%86-%E6%8C%81%E7%BB%AD%E6%9B%B4%E6%96%B0/)

