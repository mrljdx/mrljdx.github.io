# Hexo博客的搭建及同步更新


## 摘要
  最近开始使用Hexo来搭建个人博客，这篇文章主要记录在使用Hexo搭建个人博客的时候出现的一些问题。

<!--more-->
## Hexo博客的搭建
关于Hexo博客搭建的文章请参考：[Hexo你的博客](http://ibruce.info/2013/11/22/hexo-your-blog/)

## Hexo博客的同步更新
关于Hexo博客的同步问题，之前在网上查了一些大神的方法，看起来不明觉厉。
对高端备份法感兴趣的可以参考：

> 1. [Hexo-git-backup](https://github.com/coneycode/hexo-git-backup)
> 2. [Hexo 服务器端布署及 Dropbox 同步](http://lucifr.com/2013/06/02/hexo-on-cloud-with-dropbox-and-vps/)
> 3. [知乎-使用hexo，如果换了电脑怎么更新博客？](http://www.zhihu.com/question/21193762?sort=created)

下面介绍一下我使用的方法：
> - 在OSChina中创建私有项目，存放博客工程目录；在GitCafe/Github上部署博客内容。
> - Git Push 整个工程
> - Hexo deploy 部署博客内容到[Github](http://github.com/)或[GitCafe](http://gitcafe.com/)

### 修改.gitiignore文件
为了在不同机器中的同步顺利，所以要保证除了.deploy之外的文件都同步到Git服务器上。
修改后的.gitignore文件如下：
```xml
.deploy
```
配置完成后，Git push博客工程目录文件到你的代码仓库
```txt
git remote add origin "git@git.oschina.net:xxx/xxxx.git"
git push -u origin master
```

### 修改_config.xml文件
修改Hexo根目录下的_config.xml文件
> - 部署到Github的deploy配置为：

```txt
deploy:
    type: github
    repository: https://github.com/xxxx/xxxx.github.io.git #GitHub
    branch: master
```
> - 部署到GitCafe的deploy配置为：
```txt
deploy:
    type: github
    repository: git@gitcafe.com:xxxx/xxxx.git  #GitCafe 
    branch: gitcafe-pages
```
在配置_config.xml的deploy选项之后
```txt
hexo deploy ##即可将博客部署到Github/GitCafe上了
```

### 在不同电脑中更新博客

 1. 首先，电脑中需要安装Node.js环境、安装Git
 2. 其次，通过Git clone你的博客项目（以oschina的私人项目为例）
 
 ```txt
  git clone git@git.oschina.net:xxx/xxxx.git
 ```
 3. 进入到同步好的项目中，执行命令
 
```txt
npm install ##执行完该命令后，执行如下命令
hexo clean ##非必要(个人习惯)
hexo g 
hexo s ##(运行hexo博客本地服务打开locahost:4000即可预览博客)
```
如果以上三步都没有问题，那么你就可以在这台机器愉快的写文章和更新博客了。
> PS:文中的步骤都是在自己的电脑上总结出来的。如果在你的机器上同步遇到问题，可以留言，一般情况下会当天晚上回复留言内容。


