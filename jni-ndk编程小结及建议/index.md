# JNI&NDK编程小结及建议


> 本文首发于简书：[JNI&NDK编程小结及建议](http://www.jianshu.com/p/a9fc9e41f952)，欢迎关注[我的简书](http://www.jianshu.com/users/ca00dd5e52a4/latest_articles)。

## 前言
由于网上关于JNI/NDK相关的知识点介绍的比较零散而且不具备参照性，所以写了这篇JNI/NDK笔记，便于作为随时查阅的工具类型的文章，本文主要的介绍了在平时项目中常用的命令、JNI数据类型、签名等，便于查阅相关资料。文末相关参考资料比较适合刚接触或者不熟悉Android NDK开发的朋友参阅。

<!--more-->
## 常用命令
### javac 编译java源文件生成.class文件

由于JNI对应的头文件由javah工具根据对应的.class文件生成，所以在进行JNI编程之前，写好Java代码后需要先编译，在使用javah生成对应的头文件

### javah -jni自动生成头文件

举例说明：

* 生成普通的JNI头文件

      javah -classpath path -jni -d outputdirpath com.mrljdx.JavaNativeCode

* 在Java函数中包含Android相关的参数代码，则需要在classpath中添加android.jar包的绝对路径地址

      javah -classpath path:$ANDROID_HOME/path/android.jar -jni -d outputdirpath com.mrljdx.JavaNativeCodeWithAndroid

### javap -s -p 查看函数签名

-s: 显示签名（只显示public类型的签名） -p:显示所有函数、成员变量的签名

举例说明：

    javap -classpath pacakage_path_dir -s -p com.mrljdx.JavaCode

## JNI数据类型和类型签名

### 数据类型

JNI的数据类型包括：**基本类型**和**引用类型**。这一点和Java的语言特性一致，基本类型包括jboolean、jchar、jint、jlong、jbyte、jshort、jfloat、jdouble、void，与Java类型的对应关系如下：

|   JNI类型 | Java类型 | 描述 |
| ---------- |------------| ------|
| jboolean | boolean | 无符号8位整型 |
| jbyte   | byte | 有符号8位整型 |
| jchar | char | 无符号16位整型 |
| jshort | short | 有符号16位整型 |
| jint | int | 32位整型 |
| jlong | long | 64位整型|
| jfloat | float | 32位整型 |
| jdouble | double | 64位整型 |
| void | void | 无类型 |

JNI中引用类型主要有类、对象和数组，这点也上符合Java的语法规范，对应的关系如下：

| JNI 类型 | Java引用类型 | 描述|
| -------| ----------| ----|
| jobject | Object | Object类型|
| jclass | Class | Class类型 |
| jstring | String | String类型 |
| jobjectArray| Object[]| 对象数组|
| jbooleanArray| boolean[]|boolean数组|
| jbyteArray| byte[] | byte数组|
| jcharArray | char[] | char数组|
| jshortArray | short[] | short数组 |
| jintArray | int[] | int数组 |
| jlongArray | long[] | long数组|
| jfloatArray | float[] | float数组 | 
| jdoubleArray | double[] | double数组|
| jthrowable | Throwable | Throwable |

### JNI类型签名
JNI的类型签名标识了一个特定的Java类型，这个类型可以是类和方法，也可以是数据类型。
* 类型签名
类的签名采用"L+包名+类名+;"标识，包名中将``.``替换为``/``即可。
比如String类的签名：
``Ljava/lang/String;`` 
注意末尾的``;``属于签名的一部分。
再比如Android中Context类的签名：
``Landroid/content/Context;``
* 基本数据类型签名
基本数据类型的签名采用一系列大写字母来标识，如下：

| Java类型 | 签名 | 
| -------| -----|
|boolean | Z | 
| byte | B |
| char | C |
| short | S|
| int | I |
| long | J |
| float | F |
| double | D |
| void | V |

可以发现除了 ``long``基本数据类型的签名为`` J``之外其他的都比较容易辨识，估计是由于之前的类类型的签名开头为``L+包名+类名+;``设计者为了区分所以签名为``J`` 
* 数组的类型签名
数组的类型签名比起类类型和基本数据类型的要稍微复杂一点，不过还是很好理解的。对于数组来说，它的签名为``[+类型签名``，举例说明：
String[] 数组类型对应的签名：
``[Ljava/lang/String;``
可以发现，就是在String的类签名前加了个``[``
同理基本数据类型签名int[]的签名：
``[I``
注意这里基本类型后面是不带分号的。
那么多维数组呢？可以类推，int[][] 的签名为``[[I`` ，而String[][]的签名为``[[Ljava/lang/String;``
* 方法的签名
在JNI中会经常需要在C/C++代码中调用Java的函数，这时候就会用到方法的签名。方法的签名为``(+参数类型签名+)+返回值类型签名``，比如：
方法： ``boolean login(String username,String password)``的方法签名如下：
``(Ljava/lang/String;Ljava/lang/String)B`` 如果这里不理解的话，请再去看看之前关于基本类型，类类型的签名部分内容。

> 小技巧:使用 类似于``javap -classpath pathdir -s -p com.sample.JavaCode `` 的 javap -s -p 命令也可以帮助查看一些类中各种方法和成员变量的签名。

## JNI相关命名解释
* 函数名的格式遵循规则：``Java_包名_类名_方法名``
* JNIEXPORT、JNICALL、JNIEnv和jobject 都是JNI标准中所定义的类型或者宏
*  JNIEnv \* : 指向JNI环境的指针，可以通过JNIEnv \* 访问JNI提供的接口方法
* JNIEXPORT、JNICALL：是jni.h中所定义的宏。

> 注：JNIEnv \* 可以简单的理解为Java和C/C++ 之间相互调用的桥梁，我们可以通过JNIEnv \* 调用C/C++定义的方法，也可以在C/C++中通过JNIEnv \* 来调用Java类中的方法。下面将会讲到C/C++中调用Java的方法，注意JNIEnv \*的作用。 

## 在C/C++中调用Java方法
首先说明一点，在Android开发过程中使用NDK主要是为了提高代码的安全性，有些游戏公司可能是为了方便利用已有的C/C++开源库来进行平台移植，其实在性能提升方面，NDK的作用并不是很明显。所以有时候一些在Java中实现起来非常简单的代码放在JNI里面做会显得吃力不讨好，所以干脆就直接在JNI中调用Java的方法，我们只把加密和验证的一些逻辑写到JNI层就行了。
在JNI中调用Java方法流程如下：
1. 在Java中定义一个**静态**方法供JNI调用，注意要是静态的。
2. 在JNI中利用env来调用Java中定义的静态方法
3. 调用声明好的静态方法

可能流程说的比较抽象，用代码简单说明一下：
1. 定义静态方法：
  
        //对应包名：com.mrljdx.jni.HelloJNI
        public static void helloJava() {    
            System.out.println("Hello JavaCode");
        }

2. JNI声明静态方法：
    
        static void static_helloJava(JNIEnv *env){
              jclass clazz = env->FindClass("com/mrljdx/jni/HelloJNI");
              jmethodID mid = env->GetStaticMethodID(clazz, "helloJava", "()V");
              env->CallStaticVoidMethod(clazz, mid);
        }

3. 调用声明好的静态方法:
    
        static_helloJava(env);
     

## 在AndroidStudio中NDK编程配置注意事项：

1. 在项目的``gradle.properties``中添加ndk支持：
    
       android.useDeprecatedNdk=true

2. 配置``build.gradle``看代码注释：
    
        defaultConfig {    
             minSdkVersion 9   
             targetSdkVersion 23   
             versionCode 1    
             versionName "1.0"    
            //配置ndk 支持   
            ndk {        
                  //编译的so库名称 libsecurity.so
                  moduleName "security"       
                  //指定编译后的库支持的平台
                  abiFilters "armeabi", "mips", "x86", "armeabi-v7a"  
                  //用于指定应用应该使用哪个标准库，此处添加c++库支持   
                  stl "stlport_static"    
              }
        }

3. 在AndroidStudio中写JNI代码有一个比较爽的地方，就是Android.mk系统会在编译时自动帮你生成，你只需要配置build.gradle就行了。注意jni相关代码需要放在``src/main/jni``目录下。如果对gradle配置不了解可以参考我的博客：[Gradle实战及学习建议](http://mrljdx.com/2016/03/17/Gradle%E5%AE%9E%E6%88%98%E5%8F%8A%E5%AD%A6%E4%B9%A0%E5%BB%BA%E8%AE%AE/)

## 小结

在我们做产品的时候，应该考虑该用JNI&NDK的时候就用，一切出发点是基于用户的体验和数据安全，我觉得在以下几种情况下建议使用NDK:
1. 重用现有的代码，比如C/C++的代码在Android中的重用。
2. 数据安全，比如将Http的请求加密和解密算法放在NDK中去实现，这样可以提高应用的安全。
3. 提升性能，由于Android设备制造商在手机中给每个应用分配了可用的最大RAM，有时候为了性能考虑，可以通过Native代码向系统来“借”一些内存，尽量少的使用系统分配给应用的内存。（参考Infoq：[Android内存优化](http://www.infoq.com/cn/presentations/android-memory-optimization))

## 参考文章及书籍
* [《Android性能优化》](http://item.jd.com/11104976.html) : 一本不厚的书，关于Android相关的性能优化建议都是满满的干货，适合有一定开发经验的人参考。
* [翻译:JNI Tips in Android Training Course](http://www.pedant.cn/2014/08/04/jni-tips-in-chinese/) : 关于google官方的培训课程提出了在使用NDK做本地开发时的一些优化建议。
* [[JNI/NDK开发指南](http://blog.csdn.net/xyang81/article/details/41759643)](http://blog.csdn.net/xyang81/article/details/41759643) ： 一个系列的文章，适合刚接触JNI/NDK开发的朋友学习。
* [《Android开发艺术探索》](http://item.jd.com/11760209.html) ：关于JNI的介绍比少，如果你对Android开发想深入理解一些View冲突、属性动画那么可以推荐一看。如果只想学习JNI相关内容，推荐看  [[JNI/NDK开发指南](http://blog.csdn.net/xyang81/article/details/41759643)](http://blog.csdn.net/xyang81/article/details/41759643) 的内容。

## END
由于本人能力有限，文中如有疏漏和错误之处还请斧正。开卷有益，多多交流~ O(∩_∩)O哈哈~ 
欢迎关注我的简书和[个人博客](http://mrljdx.com) 会不定期写一些在平时开发中遇到和解决问题和心得。


--------------------


