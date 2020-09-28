# Hexo同时提交博客到github和gitcafe


本文记录自己在搭建好Hexo博客后deploy到github和gitcafe中的问题。
+ 介绍Hexo[官方文档](http://hexo.io/docs/deployment.html)里是如何进行配置。
+ 展示这个博客的配置项。
<!--more-->
## 根据Hexo官方文档配置
同步博客到Github和GitCafe
编辑博客根目录下的`_config.yml`文件。
```yml
  deploy:
     type: git
     message: [message]
     repo:
       github: <repository url>,[branch]
       gitcafe: <repository url>,[branch]
```

| 参数  |     描述     |
|-----  |------    |
| repo  | 存储库的URL和分支之间用`,`分开(分支不填的话默认为`master`)|
| message | 提交代码信息(默认Site updated:) |

## 展示博客配置项
举例说明：
```yml
  deploy:
     type: git
     message: “更新博客内容”
     repo:
       github: git@github.com:example/example.github.io.git,master  #将example改成你自己的
       gitcafe: git@gitcafe.com:example/example.git,gitcafe-pages  
```
然后 ** hexo clean **， ** hexo generate ** 最后 ** hexo deploy ** 即可。
在windows下运行的如上命令之后，要输入你ssh_key的密码。验证通过后才会分别提交到github和gitcafe上。
![运行截图](http://7u2j5d.com1.z0.glb.clouddn.com/hubcafe.png)
---**END**---
