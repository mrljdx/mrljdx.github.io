# AndroidStudio添加三方lib库


## 环境说明
如今AndroidStudio(简称AS)已经比较成熟了，如今windows的版本已经是1.0.2了。现在Github上的项目也基本是AS的了，所以Eclipse现在已经不符合时代发展的潮流了。废话不多说，关于使用AS的便捷之处请自行Google脑补。
### 系统环境：
>+ windows 7 64bit system
+ jdk 1.7
+ AS version : 1.0.2
+ Gradle version: 2.2.1

<!--more-->
## Eclipse和AS的比较
习惯使用Eclipse的童鞋在刚转到AS的时候需要明白AS中的一些特性和Eclipse的区别
![Eclipse vs AS](http://7u2j5d.com1.z0.glb.clouddn.com/eclipsevs.png)
### workspace vs project
在Eclipse中workspace和AS中的project类似，也就是说，AS中新建一个project就相当于eclipse新建了一个工作区间。比如Eclipse中一个workspace可以新建多个project，`java project`, `android project`等。其中不同的project可以相互依赖。
### Project vs Module
根据对比图可以看出，Eclipse中的project相当于AS中的Module。你可以像新建一个Eclipse的Project一样来新建一个Module。如图所示(AS1.0.2新建一个Module的界面)
![module](http://7u2j5d.com1.z0.glb.clouddn.com/newmodule.png)
![project](http://7u2j5d.com1.z0.glb.clouddn.com/asmodule.png)
其中MyDao模块为`JavaProject`，而app模块为`AndroidProject`，最下面的Gradle Scripts就是AS中整个Project中所有的项目和模块Gradle的配置。

### Proerties vs Module Setting
Eclipse中的Properties和AS的Module Setting比较类似，以上面的工程项目配置为例，如图所示：
`MyDao` JavaModule的配置：
![mydao setting](http://7u2j5d.com1.z0.glb.clouddn.com/mydao.png)
`app` Android Module的配置:
![app setting](http://7u2j5d.com1.z0.glb.clouddn.com/app.png)
可以看出，这里的Module Setting和Eclipse中的Properties一样，比如添加依赖的jar包，在Module Setting里选择Dependencies，然后跟在Eclipse里面一样添加就行了。

## AS中添加jar包,library项目和so库文件
明白了Eclipse和AS异同后，使用AndroidStudio就可以很顺心应手了。下面分别介绍一下如何导入jar包，为工程添加library项目和加入.so库文件。
### 导入jar包
在AS中，导入jar包提供两种方法
1.本地导入：将jar包放到module lib目录下
2.在线导入： 在Gradle中配置 complie "xxxx.xxxx.xxx" 然后运行gradle即可。

关于导入jar包在此不再赘述。
### 导入library项目
明确一点，就是在AS中 `:`表示project根目录，然后根据你所在的根目录来配置Android项目所依赖的library即可。

> compile project(':library')

###导入so库文件
现在这个版本.so导入比较简单，只需要把so文件放到libs文件夹里的对应cpu文件夹里，最后在module的build.gradle里加上jni的sourceSets配置：jniLibs.srcDirs = ['libs']，完整代码代码片如下。
```yml
android{
    //添加.so库依赖
    sourceSets {
        main {
            jniLibs.srcDirs = ['libs']
        }
    }
}
```


---**END**---




