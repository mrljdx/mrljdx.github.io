# Mac下配置React Native的一些问题

Facebook推出的React Native开发框架最初只支持Mac上的开发环境，现在也可以通过在Win上简单的配置支持React Native的应用开发。不过个人觉得React Native拿来玩玩可以，但是不适合做一些复杂的商业应用开发。
2015 年 9 月 15 号，React Native for Android 发布。至此，React 基本完成了对多端的支持。基于 React / React Native 可以：

* H5, Android, iOS 多端代码复用
* 实时热部署

<!--more -->

由于React Native的这些特性，解决了Android/IOS开发中的一些问题：

* 线上修复BUG（目前Android原生解决办法是使用插件更新应用，而IOS还是需要更新应用；部分混编依靠H5更新应用部分模块等）
* 发布应用到市场需要审核，特别是IOS应用审核周期普遍要一周。

如果做到移动应用像Web应用一样在服务器部署就能让用户用到，这就完美了。不过目前原生应用很难做到这一点。所以React Native的前景还是看起来不错的，特别是有Facebook的支持发展应该没啥问题。

按照[秋百万](http://weibo.com/liaohuqiu?sudaref=passport.weibo.com&is_all=1)的教程[React Native: 配置和起步](http://www.liaohuqiu.net/cn/posts/react-native-1/) 既可以在Mac上配置好React Native开发环境，并开始开发。

不过在配置完环境后，发现 nvm 和 node.js 等命令会在重启终端后失效。不科学，最终发现在个人的环境变量中需要添加 nvm 才能在重启终端后依然有效。
步骤如下：

* 打开bash_profile配置环境变量
	> vim ~/.bash_profile
* 添加如下内容到 ~/.bash_profile 中
	> export NVM_DIR=~/.nvm
* 配置brew nvm命令生效
	>source $(brew --prefix nvm)/nvm.sh
* 让配置的环境变量立即生效
	> $ source ~/.bash_profile

由于之前安装node js 以及 watchman 等工具都是基于 nvm 安装的。所以，只要nvm配置的环境生效，node 也会生效。
安装参照的网址：
http://www.liaohuqiu.net/cn/posts/react-native-1/
for ReactNative Android 的学习，目前这种技术在国内来讲还是不太普遍，因为编译起来太慢了，编译成功后还需要调试配置服务器选项。

小结：

目前ReactNative暂时只适合一些资讯类的小应用开发（这点适合外包团队的外包项目，类似于[PhoneGap](http://phonegap.com/about/)但是强于PhoneGap），也适合一些具有JS基础的开发人员转移动开发过度。
复杂的商业应用还是不适合。考虑到如下几个原因：

1. 应用加密保护
2. 相关开发调试工具目前ReactNative还不够完善
3. 不能有效利用原生的优势，使用ReactNative开发的应用还是不如原生流畅，在包的体积上还是有点大。
4. 调试定位错误不方便

以上几点原因只是个人对于React Native的一点浅薄的看法，如有纰漏还请指正。

END

-----------------------
