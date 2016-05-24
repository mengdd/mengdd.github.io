---
title: 2016-05-23-set-up-github-pages-with-hexo.md
date: 2016-05-23 18:12:02
tags:
---

# Set up Github Pages with Hexo, migrating from Jekyll.

本文介绍用Hexo建立github pages, 其中包含了从Jekyll迁移过来的过程.
Migrate github pages from Jekyll to [Hexo](https://hexo.io/docs/index.html).
Set up github pages using Hexo.
不光是迁移哇, 直接用Hexo setup github pages 看这个也有用哇.

<!-- more -->

为什么要把github pages 从Jekyll实现迁移到Hexo?
前阵子用Jekyll建了github pages(官方推荐), 但是发现添加代码段比较痛苦, markdown的前后三个点并不能标记一个代码块, 需要在代码块前后加上两句特定的语句.
如果一篇文章有很多代码块, 这样一个一个加下来比较费劲, 而且加完了之后代码段的样式也不是很好看.

为此很苦恼的我问了phodal大神, 大神回复: Hexo.

下文记录了我的操作过程.

## 安装Hexo
必要条件:
[Node.js](https://nodejs.org/en/)
可以选个pkg,下载后点击安装, 装完之后告诉你路径.

也可以用nvm装的
[nvm](https://github.com/creationix/nvm)
Git
这个一般大家都有哒~

上面两个都有之后, 安装Hexo:

`$ npm install -g hero-cli `
装得也很快.
之后试一下hexo命令,如果有命令介绍(而不是command not found)就代表装好了.

## 设置和迁移
[Setup](https://hexo.io/docs/setup.html)

### 备份
首先, 从原先的_posts/目录下将原来的文章都拷贝出来.
另外将.git目录也拷贝出来(这是为了保持github上的历史).
这些文件另外保存.

### 建立新的目录
在准备好的空目录运行命令:
`hexo init .`
就建立好了hexo的目录,相关介绍可以去网站看.
`npm install` 下载依赖包.

Hexo会自动忽略下划线开头的目录和文件名,但是_posts目录除外.
这时候可以运行
`$ hexo server`
然后访问http://localhost:4000/预览一下.

### 配置
网站的设置文件是`_config.yml`
打开可以配置一些新东东, 比如title url之类的.
具体设置参照这个: [configuration](https://hexo.io/docs/configuration.html)

### 内容迁移
这是内容迁移的介绍:
[migration](https://hexo.io/docs/migration.html)

所以首先在`_config.yml`文件里把 new_post_name字段改为:
`new_post_name: :year-:month-:day-:title.md`
否则就要修改之前每一个文章的文件名,太麻烦,而且我觉得加个日期也比较好.

然后把原来备份的博客文章移到source/_posts/目录下.

比较bug的是以前jekyll文章里的代码段前后加的那两句还得手动移除.
用Hexo后 前后各加三个点即可标记代码段.
如果想要代码高亮, 比如是java, 代码段首的三个点后加个java.
这里可以查看代码高亮的各种语言: [highlightjs](http://highlightjs.readthedocs.io/en/latest/css-classes-reference.html)

完成之后可以运行hexo server 命令在本地看一下样子.

然后把.git目录拷贝回来放在根目录. 可以看到repo地址啊, 历史记录啊还在.
最后提交, 本次提交即为迁移提交, push.

## Deployment
[部署Deployment](https://hexo.io/docs/deployment.html)

我本来以为跟Jekyll一样本地运行好了, push上去就生效了, 结果并没有.
访问原地址, github pages并不生效,居然还是原来的那个样子.

查了一下是因为deploy没有设置.

打开`_config.yml`文件,找到deploy字段, 设置一下.
我的是这样写的:

```
deploy:
  type: git
  repo: https://github.com/mengdd/mengdd.github.io
  branch: master
```

然后执行一下这条命令:
`npm install hexo-deployer-git --save`
这条命令执行后package.json会有一条改动, 新添加了一个插件.
把这个提交了.

然后generate和deploy:
`$ hexo generate --deploy`
`$ hexo deploy --generate`
这两条命令是一样的.
运行这条命令的时候可能会要求你输入github的账户名和密码(如果你没有配置SSH key的话).
这个需要等待比较长的时间, 实际上最后它是给master分支做了一个`git push -f`
所以它实际上是不用你自己push的.

之后就可以访问啦: http://mengdd.github.io/
哒哒!

## 网站代码和静态网页管理
在知乎上看到这个问题: [使用hexo, 如果换了电脑怎么更新博客?](https://www.zhihu.com/question/21193762)
乍一看感觉很奇怪, 换一台电脑难道不是clone下来就行了吗? 仔细看了一下恍然大悟.
前面提到刚刚的部署命令执行了一次强制push, 可是当我们查看github上的repo(https://github.com/mengdd/mengdd.github.io),
发现push上去的文件和我们本地的这些完全不同.
现在在origin master上的全是一些静态html文件.
而我们本地master分支上是一些配置, 还有source文件等.
也就是等于我们把母鸡留在了本地电脑, 执行deploy之后只把鸡蛋push到了origin. (我这个形象的比喻).

那么怎么解决呢? 该问题下排名第一的 [CrazyMilk] 大哥已经给出了答案.
[他的博文](http://crazymilk.github.io/2015/12/28/GitHub-Pages-Hexo%E6%90%AD%E5%BB%BA%E5%8D%9A%E5%AE%A2/)
所以解决办法就是新建一个分支把网站代码(母鸡)放上去咯~哈~

首先在本地, 基于当前本地的master新建一个hexo分支:
`git checkout -b hexo`
然后push到origin上去:
`git push origin hexo`
耗时比较久, 请耐心等待.

最后在github的settings页面把hexo分支设置为default.
DONE!

## Theme
之后想设置一个好看的主题, 知乎上居然还有这么个问题:
[有哪些好看的Hexo主题?](https://www.zhihu.com/question/24422335)
我打算选这个试试: https://github.com/wuchong/jacman
在本地hexo分支根目录下运行:
`git clone https://github.com/wuchong/jacman.git themes/jacman`
就会clone到themes目录下面叫jacman的目录下.
然后在根目录下的`_config.yml`中把theme名改为jacman.

然后用命令hexo server就可以在本地查看效果.
改主题大概就酱, 两步就可以完成, 可以多试几个选一选.
选完了在hexo分支提交保存,然后运行`hexo d -g`生成部署即可.

## Future
需要用到的命令: [commands](https://hexo.io/docs/commands.html) 随时查看哇.




