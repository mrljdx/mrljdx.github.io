# 连接同一wifi配置Charles代理的问题

关于Charles的使用网上有很多文章，这篇文章主要是由于博主最近在mac上使用Charles碰到了一个很蛋疼的问题，分享出来让一些遇到同样问题的人有个解决思路。
## 问题描述
### 环境描述
1. Mac + 连接Wifi + 打开Charles 设置好端口号，勾选代理。
2. App 连接同样的Wifi + 手动设置代理 + Mac的IP地址+设置好的端口号

### 出现的异常
1. 手机代理已经设置好了，但是Charles没有弹出提示框，告诉用户有ip地址连接了代理。
2. 手机连接代理后，无法连接网络，抓包窗口没有发现相关的网络请求。
<!--more-->
### 尝试的解决过程
首先是各种google搜索问题的答案，总结解决的过程如下：
1. 关闭Mac的防火墙，重新连接wifi，设置代理。测试【失败】
2. 重启charles，配置非8888的端口，手机端同时同步修改端口。测试【失败】
3. 重启电脑+ 关闭防火墙+重新卸载安装正版Charles（有试用期）+手机手动配置代理+重连wifi。 测试【失败】
4. 重复以上3各情况的各种排列组合。测试【失败】
5. 按照[wooyun的charles教程](http://drops.wooyun.org/tips/2423)手动配置手机的IP地址为静态IP地址。重新连接wifi + 重启charles+重复[1\2\3]步骤。测试【失败】

## 正确的解决姿势
最终发现，其实存在一个问题，就是一开始 Mac + Mobile 的手机连接的wifi属于同一个wifi，本来以为这样是没有任何问题的。但是事实证明，这样看似正确的网络环境其实有问题。
### 解决的方法步骤
1. Mac连接有线网络；
2. 手机连接Mac的有线网络ip地址，并配置端口号；

这时候神奇的一幕出现了，手机手动配置代理后，charles 出现弹框，选择【deny or accept 】。
泪奔~ 折腾了一天，终于可以连了。仅此记录mac下同一个wifi环境下，charles配置正确的前提下，手机抓包抓不到的问题。

## 问题的延展
在Win上如果也有这种问题，那么解决的思路也是大同小异，说不定fiddler也会碰到同样的情况。

## END
有些看似正确的配置，一开始就容易误导人，有时候如果一个问题无论如何都解决不了，到不如站在更高的维度来考虑，是不是一开始的环境就有问题。说不定就豁然开朗了~ 个人的一点浅见。


---------------------
