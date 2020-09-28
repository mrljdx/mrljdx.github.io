# 资源混淆方案之微信vs美团

> 前言:目前在Android市场有很多应用在开发之后都不注重对代码和资源文件的混淆,而且大部分的小众应用在传输数据的时候,直接是Http明文传输,在这一点上比起大公司在安全方面的
考虑是明显不足的.关于资源文件混淆,目前国内有美团和微信都给出了相关的方案,作为一个没做过资源混淆的人来说,来比较一下这两个方案.

<!--more-->

## 微信资源混淆方案

Android资源混淆工具使用说明:[<font color="#aaaa">链接地址</font>](https://github.com/shwenzhang/AndResGuard/blob/master/README.zh-cn.md)
在实践过程中,基本上没有碰到任何问题,并且在使用apktool 进行反编译是获取不到资源文件的.有图不虚:

** 资源混淆后的解压文件:**
![extrafile](http://7u2j5d.com1.z0.glb.clouddn.com/weixinE.png)

** Apktool反编译后的文件: **
![apktool file](http://7u2j5d.com1.z0.glb.clouddn.com/weixin.png)

由于微信团队,对于资源混淆工具的使用说明比较详细,在此就不再说明具体的工具用法.
接下来看看美团团队的方案.

## 美团资源混淆方案

美团Android资源混淆保护实践:[<font color="#aaaa">原文地址</font>](http://tech.meituan.com/mt-android-resource-obfuscation.html)
美团的做法是,修改aapt来实现资源文件命名的混淆.
这种做法我没尝试过,感觉修改aapt不是那么方便,aapt位于 /sdk/build-tools/{android_version}/ 目录下,其中android_version为各个build_tools版本号.
这里纠正一下 **美团Android资源混淆保护实战中** 关于**aapt位置**的错误.
那么问题来了,如果仅仅修改aapt作为混淆的唯一方式,那么有些文件资源的名称是不能混淆的,比如Umeng的更新组件中一些资源文件的名称如果被混淆,就会导致apk在运行相关组件报错.

美团的资源混淆方案的最后一句话如下
    
> 这样通过修改AAPT，我们可以在代码零修改的基础下就能做到相对的资源安全，当然安全是相对的，没有绝对的安全。

对于普通的Android开发者来说,这种方案比较麻烦.直到看完这篇文章我也不知道怎么去修改aapt文件,醉了.

## 方案比较

微信的方案在这一点中就做的很好,在config.xml文件中,用户可以列举 资源文件混淆白名单,如下图
![Umeng file](http://7u2j5d.com1.z0.glb.clouddn.com/um.png)

在混淆方面微信团队的方案考虑的更加全面和周到,而且在打包的时候,可以在不影响之前编码的情况下做到资源文件保护.
微信这个方案可以说是完胜美团的方案.其他的好处我就不说了,看看微信团队的config.xml文件你就懂了.

## 总结

想做资源文件混淆的,可以直接考虑微信的方案.美团的基本可以不用考虑,talk is cheap , show you the code .
config.xml
    
    <?xml version="1.0" encoding="UTF-8"?>
    <resproguard>
        <!--defaut property to set  -->
        <issue id="property" > 
            <!--whether use 7zip to repackage the signed apk, you must install the 7z command line version in window -->
            <!--sudo apt-get install p7zip-full in linux -->
            <!--and you must write the sign data fist, and i found that if we use linux, we can get a better result -->  
            <seventzip value= "false" />
            <!--the sign data file name in your apk, default must be META-INF-->
            <!--generally, you do not need to change it if you dont change the meta file name in your apk-->
            <metaname value="META-INF" />
            <!--if keep root, res/drawable will be kept, it won't be changed to such as r/s-->
            <keeproot value="false" />
        </issue>
        
        <!--whitelist, some resource id you can not proguard, such as getIdentifier-->
        <!--isactive, whether to use whitelist, you can set false to close it simply-->
        <!--资源混淆白名单, isactive 设置为true生效-->
        <issue id="whitelist" isactive="false">
             <!--you must write the full package name, such as com.tencent.mm.R -->
             <!--for some reason, we should keep our icon better-->
             <!--and it support *, ?, such as com.tencent.mm.R.drawable.emoji_*, com.tencent.mm.R.drawable.emoji_?-->
            <path value ="com.tencent.mm.R.drawable.icon" />	
            <path value ="com.tencent.mm.R.drawable.emoji_*" />
        </issue>
        
        <!--keepmapping, sometimes if we need to support incremental upgrade, we should keep the old mapping-->
        <!--isactive, whether to use keepmapping, you can set false to close it simply-->
        <!--if you use -mapping to set keepmapping property in cammand line, these setting will be overlayed-->
        <issue id="keepmapping" isactive="false">
            <!--the old mapping path, in window use \, in linux use /, and the default path is the running location-->
            <!--填写签名后的 mapping 文件地址 -->
            <path value ="/home/shwenzhang/workspace/ResourceProguard/resource_mapping_weixin50android.txt" />
        </issue>
    
        <!--compress, if you want to compress the file, the name is relative path, such as resources.arsc, res/drawable-hdpi/welcome.png-->
        <!--what can you compress? generally, if your resources.arsc less than 1m, you can compress it. and i think compress .png, .jpg is ok-->
        <!--isactive, whether to use compress, you can set false to close it simply-->
        <issue id ="compress" isactive="false">
             <!--you must use / separation, and it support *, ?, such as *.png, *.jpg, res/drawable-hdpi/welcome_?.png-->
            <path value ="*.png" />
            <path value ="*.jpg" />
            <path value ="*.jpeg" />
            <path value ="*.gif" />
            <path value ="resources.arsc" />
        </issue>
    
        <!--sign, if you want to sign the apk, and if you want to use 7zip, you must fill in the following data-->
        <!--isactive, whether to use sign, you can set false to close it simply-->
        <!--if you use -signature to set sign property in cammand line, these setting will be overlayed-->
        <!--配置包的签名文件和秘钥,isactive 设置为true生效-->
        <issue id="sign" isactive="false">
            <!--the signature file path, in window use \, in linux use /, and the default path is the running location-->
            <path value ="/home/shwenzhang/workspace/ResourceProguard/signature.keystore" />
            <!--storepass-->
            <storepass value="storepassvalue" />
            <!--keypass-->
            <keypass value="keypassvalue" />
            <!--alias-->
            <alias value="aliasvalue" />
        </issue>
         
    </resproguard>

## 相关资料

(1) 微信资源混淆实现原理:[原文地址](http://mp.weixin.qq.com/s?__biz=MzAwNDY1ODY2OQ==&mid=208135658&idx=1&sn=ac9bd6b4927e9e82f9fa14e396183a8f#rd)
(2) 安装包立减1M--微信Android资源混淆打包工具:[原文地址](http://bugly.qq.com/bbs/forum.php?mod=viewthread&tid=42)
(3) AAPT介绍:[原文地址](https://github.com/recter/Anti-ADT/tree/master/aapt)
(4) 美团Android资源包保护实战:[原文地址](http://tech.meituan.com/mt-android-resource-obfuscation.html)
(5) 微信Android资源文件保护实战:[原文地址](https://github.com/shwenzhang/AndResGuard/blob/master/README.zh-cn.md)

--------------------
    

