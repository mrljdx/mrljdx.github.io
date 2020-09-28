# Gradle实战及学习建议

> 本文首发于简书：[Gradle实战及学习建议](http://www.jianshu.com/p/b7fc7a6abffb)，欢迎关注[我的简书](http://www.jianshu.com/users/ca00dd5e52a4/latest_articles)。

## 前言
相信不少使用Android Studio开发Android的朋友都在为Gradle中的一些配置疑惑，今天来介绍一下我在学习Gradle的一些经验和总结，希望能对大家有所帮助。先大致的看一张Gradle学习的结构图，对正片文章有个大致的了解，其次逐一说明一些Android Gradle 插件中的一些变量的含义及用法实例。
<!--more-->
![Gradle学习框架图](http://upload-images.jianshu.io/upload_images/115071-0bc086b5b76ed8b8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## Groovy简介
由于Gradle使用Groovy作为默认的编译语言，所以在学习Gradle之前，可以简单的了解一下Groovy的一些简单语法会事半功倍。
### Groovy特点
1.  Groovy和Java一样，源码都会被编译成class字节码文件然后可以在JVM中运行；
2. Groovy兼容所有的Java语法，也就是说，你可以在.groovy中直接写Java代码，编译运行；
3. Groovy中自定义变量和方法都使用关键字 def 来定义；
4. 闭包（注意，这将是在Gradle配置中经常使用的）

关于Groovy的语法学习教程很多，这里篇幅原因，就不详细解释了。可以参看简书的另一篇博文：[《Groovy语法》](http://www.jianshu.com/p/1e95d03060f7)

## Gradle语法介绍
这里就具体的Gradle语法进行介绍，如果要详细介绍可能需要一本书才能讲完。本文仅结合具体的项目来介绍一些关键的知识点。
在Gradle中，每个Project都对应一个``build.gradle``文件，而buidl.gralde中定义了每个Project所需要执行的Task，而这些Task是由具体的插件(Plugin)决定的。
比如**Java Project**的Gradle插件在``build.gradle``中配置如下：

    group 'Sample'
    version '1.0-SNAPSHOT'
    //配置java插件
    apply plugin: 'java'
    //编译源文件时使用的Java版本
    sourceCompatibility = 1.5
    repositories {    
        mavenCentral()
    }
    dependencies {    
        testCompile group: 'junit', name: 'junit', version: '4.11'
    }

以上是我基于IDEA新建的一个简单的Java工程，build.gralde为自动生成文件。由于项目为Java工程，所以需要使用Gradle的Java插件来定义编译规则。
同理，我们在使用AndroidStudio 新建一个Android项目时，也会生成对应的``build.gradle``文件。以新建一个Sample工程为例：
***/app/build.gradle***如下（注释详解）：

    //使用Gradle Android插件
    //DSL语法网址：http://google.github.io/android-gradle-dsl/current/index.html
    apply plugin: 'com.android.application'
    //android{...} 表示插件的 dsl 入口，凡是在DSL中定义的都只在此闭包中生效
    android {    
        //编译的sdk版本（必填）
        compileSdkVersion 23  
        //构建工具版本  （必填）
        buildToolsVersion "23.0.2" 
        //默认配置   
        defaultConfig {        
                //应用标识
                applicationId "com.mrljdx.sample" 
                //最小sdk版本       
                minSdkVersion 9      
                //编译sdk版本  
                targetSdkVersion 23  
                //版本号      
                versionCode 1        
                //版本名称
                versionName "1.0"    
          }   
          //编译类型，在构建编译变种时，可以在此定义 
          buildTypes {        
                 release {            
                        minifyEnabled false            
                        proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'        
                 }    
           }
    //android dsl 结束
    }

    dependencies {    
             compile fileTree(dir: 'libs', include: ['*.jar'])    
             testCompile 'junit:junit:4.12'    
             compile 'com.android.support:appcompat-v7:23.2.0'    
             compile 'com.android.support:design:23.2.0'
     }

Gradle Android Plugin 工程目录结构约定如下：

![Android工程目录结构.png](http://upload-images.jianshu.io/upload_images/115071-d8948730fbd4c758.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
其中说明一下**jniLibs**目录，它主要存放.so库文件。在**jni**文件夹下存放ndk编译相关的c/c++文件，编译后生成的文件会在jniLibs中，Gradle默认读取jniLibs目录下的.so库文件，所以一般习惯于Eclipse的为了改变文件目录，会使用Android Plugin 的 DSL 语法 sourceSet来重新定义文件目录，如下：
  
    
    sourceSets {    
          main {     
              //将jniLibs下的文件/子文件目录 定义到与'src'同级的'libs'目录下 
              jniLibs.srcDirs = ['libs']   
          }
    }

对比Eclipse目录结构和Android Studio的目录结构

![Eclipse.png](http://upload-images.jianshu.io/upload_images/115071-0e37fc375774bbf0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![AndroidStudio.png](http://upload-images.jianshu.io/upload_images/115071-88b1d275aa368784.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
可以发现在Android Studio中，会默认识别Gradle Android Plugin 预先约定的工程目录结构，有些文件夹会变成蓝色，表示源码目录，而带有数据表示的文件夹，则表示存放资源文件。这点比起Eclipse要好很多。
那么问题来了，如何通过配置build.gralde来适配Eclipse工程导出的文件目录结构呢？其实不难，如下配置即可：

    sourceSets {    
            main {        
                  manifest.srcFile 'AndroidManifest.xml'        
                  java.srcDirs = ['src']        
                  resources.srcDirs = ['src']        
                  aidl.srcDirs = ['src']        
                  renderscript.srcDirs = ['src']        
                  res.srcDirs = ['res']        
                  assets.srcDirs = ['assets']    
            }    
            // Move the tests to tests/java, tests/res, etc...    instrumentTest.setRoot('tests')    
            // Move the build types to build-types/<type>    
            // For instance, build-types/debug/java, build-types/debug/AndroidManifest.xml, ...    
            // This moves them out of them default location under src/<type>/... which would    
            // conflict with src/ being used by the main source set.    
            // Adding new build types or product flavors should be accompanied    
            // by a similar customization.    
            debug.setRoot('build-types/debug')    
            release.setRoot('build-types/release')
            androidTest.setRoot('test')
      }

理解以上代码并不难，请参阅[《Android Plugin DSL参考手册》](http://google.github.io/android-gradle-dsl/current/index.html) 即可。

## Gradle的生命周期
通过分析Android Studio创建的项目结构或者IDEA创建的Java工程项目结构，不难发现，在每个Project的根目录下，一定会存在 ``setting.gradle``文件。
Gradle有一个初始化流程，这个时候``setting.gradle``会执行，通过``setting.gradle``可以获取当前整个工程的配置获取一个[Settings](https://docs.gradle.org/current/dsl/org.gradle.api.initialization.Settings.html)对象(查看[Gradle DSL参考手册](https://docs.gradle.org/current/dsl/org.gradle.api.initialization.Settings.html) 了解更多详情)
关于Gradle的Project相关的生命周期、任务、依赖等官方解释如下：

![Gradle Offical.png](http://upload-images.jianshu.io/upload_images/115071-9116e6bf5eb148ce.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
可以简单的总结一下，就是在初始化时，读取setting.gradle中的项目配置，来确定编译的工程，通过每个工程中的build.gradle来确定编译的任务，而这些任务是由对应的plugin来决定的，比如 'com.android.application'插件决定编译成apk，而'com.android.library'插件决定项目编译成.aar库文件等。
在每个插件中，都有相应的语法约定。通过[Android DSL 手册](http://google.github.io/android-gradle-dsl/current/index.html) 可以理解为啥要那么定义。以及每个定义中的变量所对应的类型和方法。
在博客[《Android中实用的Gradle配置》](http://mrljdx.com/2016/01/12/Android%E6%9C%80%E5%AE%9E%E7%94%A8%E7%9A%84Gradle%E9%85%8D%E7%BD%AE/) 中说明了一个应用在开发过程中的一些配置问题，其中对于每一个配置都有解释说明，感兴趣的可以参考一下。

## Gradle学习建议
关于Gradle的具体语法相关DSL细节，如果在这里讲可能要写很多。授人以鱼不如授人以渔，这里结合自身的学习过程给大家一点建议和参考：

* [Gradle 官方文档](https://docs.gradle.org/current/javadoc/org/gradle/api/Project.html) ：了解Project对应的章节，仔细阅读LifeCycle、Tasks、Dependencies、Multi-project Builds、Plugins、Properties、Extra Properties、Dynamic Methods相关内容。基本上可以理解为何一个工程项目的结构以及依赖是如何导入的。
* [深入理解Android（一）：Gradle详解](http://www.infoq.com/cn/articles/android-in-depth-gradle) ：详细的介绍了Groovy相关语法、Gradle构建细节等内容。其中关于Groovy的 List、Map、Range、闭包等内容值得细看，因为在Gradle的使用中，比如多版本多渠道的APK构建会用到这些内容。
* [Android DSL 参考手册](http://google.github.io/android-gradle-dsl/current/index.html) ：可以结合本片文章开头的图来进行系统学习，了解Android DSL的语法层级，了解构建的一些内容，理解即可。
* [Gralde In Action 中文版](https://lippiouyang.gitbooks.io/gradle-in-action-cn/content/index.html)：看名字就知道，这本书主要讲解一些Gradle常用的实例讲解，可以好好看看

## END
由于个人能力有限，在文中如有纰漏还请斧正，关于Gradle构建项目相关的问题留言欢迎交流，开卷有益~

-------------------------


