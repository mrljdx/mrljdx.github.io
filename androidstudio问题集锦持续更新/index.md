# AndroidStudio问题集锦（持续更新中）

在这里记录平时在使用AndroidStudio开发，遇到的问题。以及解决办法，如果大家也有相应的问题，可以在这里留言
解决的问题都会集中展现在这里以供查阅。这篇文章会不定期更新，来汇总大家所遇到的问题。方便以后进行查阅。
<!--more-->
### Gradle: 错误: 编码GBK的不可映射字符
> 解决办法：
> 在所在项目之下的build.gradle中添加：
> **tasks.withType(JavaCompile) { options.encoding = "UTF-8" }**
> 注：由于Gradlev2.2.1的升级，语法不向下兼容。之前解决这个问题的方法是添加：
>  **tasks.withType(Compile) { options.encoding = "UTF-8" }**

### Could not find property 'processResources' 
> 解决办法：
> 修改build.gradle中的**androidManifestFile variant.processResources.manifestFile** 改成 **androidManifestFile variant.outputs[0].processResources.manifestFile**即可。
> 原因：Gradle升级导致，新的目录指向于**variant.outputs[0]**

### Android Studio: Gradle - build fails — Execution failed for task ':dexDebug'
> 原因:
> 你有一些jar文件没有进行编译。应该到你的项目文件build.gradle修改jar包的依赖关系，如果你只是导入一些jar文件，你可以尝试将其删除，并添加一次，看看是哪一个jar包让你的工程出现错误。就我而言，我就是这样做的，比如你的build.gradle中有**compile 'com.android.support:appcompat-v7:21.0.3'**同时libs目录下有**android-support-v4.jar**包就出现了上述的错误。所以我想v7的包和v4的包，可能重复导入多次同一个jar，当删掉v4的包之后，问题就解决了。
> 注：这个问题也存在于导入一些三方library时，该library中也有v4的包，然后你的工程也有一个v4的包，这样相对于项目同时导入v4的包两次，所以就报错了。

### Gradle警告: Not recognizing known sRGB profile that has been edited
> 原因:
> 就我遇到的这个问题，是因为我在工程目录libs中添加了android-support-v4的包。导致在编译的时候出现如上错误。在删除该包之后，gradle build就没有这个警告了。
> 注意：
> 如果你碰到的问题和我的不一样，请看这里[链接地址](http://poetengineer.postach.io/libpng-warning-iccp-not-recognizing-known-srgb-profile-that-has-been-edited)
> 由于墙的存在，怕各位看不到，就在这里贴一下原文：
> <font size="5">libpng warning: iCCP: Not recognizing known sRGB profile that has been edited</font>
If you're seeing this warning ("libpng warning: iCCP: Not recognizing known sRGB profile that has been edited") in your gradle build output, you have three options:
+ Remove iCCP chunks from your PNGs, using ImageMagick: http://tex.stackexchange.com/questions/125612/warning-pdflatex-libpng-warning-iccp-known-incorrect-srgb-profile
+ Downgrade to an earlier gradle version, which will also cause a libpng downgrade
+ Wait for the next version, promised in comment #9 here: https://code.google.com/p/android/issues/detail?can=2&start=0&num=100&q=&colspec=ID%20Type%20Status%20Owner%20Summary%20Stars&groupby=&sort=&id=77704


未完，待续。

