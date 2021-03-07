---
title: hexo5.0+安装报错总结
mylink: hexo5.0+安装报错总结
date: 2021-03-05 16:33:10
tags:
	-
categories:
	- CS
---

## 部署与备份小总结

经过我三次的 `hexo` 部署经验，现在也算是摸清了它的逻辑，这次没照搬网上的教程，自己摸索出了如何通过设置分支来部署备份。

<!-- more -->

首先明确两点，`github` 展示的页面，不由自己掌管，也不用自己 `git push` 全部交由 `hexo g -d` 这个命令来完成，它会在 `/public` 文件夹中生成要部署的静态页面，然后复制到 `/.deploy_git` 文件夹中，再全部覆盖推到我们在 `/_config.yml`中设置的 分支，若设置的是 `master`，则文件内容是这样的：
```yml
deploy: 
  branch: master
```

我们要做的，是设置一个备份分支，如 `hexo` 分支，并在 `github` 中将其设置成默认分支，然后手动推到远程，`master` 和 `hexo`这两个分支没有关联，我们也只要维护 `hexo`分支来备份自己的文件。而 `master`分支下的静态文件的提交，由 `hexo g -d`来完成（是的这部分我说了两遍）

有一点需要注意，在仓库界面，如果你建立的第一个分支不是静态文件所在的那个分支，要手动改回 `master`，即 `hexo` 分支。（一般情况最先建立的都是主分支，即静态文件分支，不用进行这个操作）

![image-20210305214426331](https://tcualhp-notes.oss-cn-hangzhou.aliyuncs.com/img/image-20210305214426331.png)

我的操作流程是这样的：

1. 先在本地把整个博客都搭好了，文件夹名字随意
2. 在远程仓库建立 `用户名.github.io` 仓库，注意 `readme` 和 `.gitignore` 文件都不要
3. 在本地博客目录下， `git init`，然后 `git remote add origin git@github.com:用户名/用户名.github.io.git` 与远程仓库建立联系
4. 然后再把本地文件提交，推上去，再检出 `hexo` 分支，推上去，将 `hexo` 分支设置为默认分支
5. 后面需要写文章的时候，用 `hexo new "文件名"`
6. 部署和备份直接运行我去年写的脚本，将脚本复制到博客根目录下，`git bash` 键入：`sh auto_deploy.sh "github提交的commit"` 即可完成部署+备份

```shell
# 切回hexo, 以防万一
git checkout hexo
# 提交本地的
git add .
git commit -m "$1"
# 提交到远程
git push origin hexo
# 到页面发布分支
# git checkout master
# 清理原html
hexo clean
# 生成，部署
hexo g -d
# 返回原文件分支
# git checkout hexo		
```



## 参考链接

写到这里，下面的链接早已不局限于安装部署了，各种杂七杂八的关于自建 `hexo` 的链接都有收录

`hexo` 官方文档：[文档 | Hexo](https://hexo.io/zh-cn/docs/)

* `hexo` 中 `_posts` 文件夹分类：
* [如何在Hexo中对文章md文件分类_一个码农的博客-CSDN博客](https://blog.csdn.net/maosidiaoxian/article/details/85220394)

* [如何对hexo中的文章进行分类管理 | 大专栏](https://www.dazhuanlan.com/2020/03/28/5e7e34ca352b3/)

* 添加看板娘：[Hexo添加Live2D看板娘最新教程_enchanted的博客-CSDN博客](https://blog.csdn.net/qq_36239569/article/details/104104894)

* https://tang.su/2020/09/upgrade-hexo-to-5-0/#more)

* [Hexo NexT 主题优化：显示文章阅读次数 | 九月枫林](http://www.yangyong.xyz/2018/01/03/add-hexo-next-post-views/)

* [hexo博客next主题添加 评论功能_Mumu's Blogs-CSDN博客](https://blog.csdn.net/zhu_1997/article/details/87554975)

* [zhaojun1998/Valine-Admin: 一个 Valine 的拓展应用，用来增强 Valine 的邮件通知。](https://github.com/zhaojun1998/Valine-Admin)

* [[转]Hexo主题使用Valine-Admin管理评论和评论提醒_MelodyJerry-CSDN博客](https://blog.csdn.net/weixin_43438052/article/details/106617739?utm_medium=distribute.pc_relevant.none-task-blog-baidujs_title-0&spm=1001.2101.3001.4242)

* [阿里企业邮箱POP\SMTP\IMAP地址和端口信息_凌飞安-CSDN博客_阿里邮箱pop服务器](https://blog.csdn.net/lingfeian/article/details/100030808?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.control&dist_request_id=1328603.27235.16150118713713009&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.control)

* [Hexo NexT 7谷歌收录 必应收录 百度收录 | 蓝蓝博客](https://lanlan2017.github.io/blog/242f5d55/)

* [Hexo博客：十一、文章置顶 - 简书](https://www.jianshu.com/p/a94422c0dc48/)

* [Hexo安装Next主题 Rss 侧边栏 | 漂自己的移，让别人都撞墙去吧](https://www.gagahappy.com/use-next-theme/)

* [给 Hexo 中的 Next 主题添加 RSS 功能 | 苏寅 Blog](https://suyin-blog.club/2020/2M3YWE7/)

* [给 Hexo Next 网站评论框配置炫酷的打字效果 | 苏寅 Blog](https://suyin-blog.club/2020/EBJEZ6/)

* 打字特效（一定要先看第一个链接，涉及到 next 主题配置分离思想）
  * [Hexo-NexT 版本更新记录 | 小丁的个人博客](https://tding.top/archives/2bd6d82.html)
  * [Hexo-NexT 添加打字特效、鼠标点击特效 | 小丁的个人博客](https://tding.top/archives/58cff12b.html) 
* [hexo-submit-urls-to-search-engine 中文文档 | 峡州仙士之页](https://cjh0613.com/20200603HexoSubmitUrlsToSearchEngine.html)



## 报错

### 报错一：

```
FATAL {
  err: YAMLException: end of the stream or a document separator is expected (1:1)

   1 | ```
  -----^
      at generateError (D:\Desktop\blog\node_modules\js-yaml\lib\loader.js:183:10)asap.js:40:19)
      at flush 
      
      ---------------中间省略---------------------
      (internal/process/task_queues.js:75:11) {
    reason: 'end of the stream or a document separator is expected',
    mark: {
      name: null,
      buffer: '```\n',
      position: 0,
      line: 0,
      column: 0,
      snippet: ' 1 | ```\n-----^'
    }
  }
} Something's wrong. Maybe you can find the solution here: %s https://hexo.io/docs/troubleshooting.html

```

查阅资料得知，大部分情况为，`_config.yml` 文件下所有的 `:` 后，必须有空格，且只能有一个空格。于是我开始检查，利用编辑器搜索 `: `，高亮出符合内容的冒号，这样就能快速找出不符合要求的冒号。但是试了好几遍，没有找出不符合要求的，缓存清除一类的也不行。



在这个时候，我仔细阅读错误，可以看到，错误中指示了 「buffer:'```\n」

我一直认为这是乱码，没有什么指导性作用。直到后来我看别人的报错的 `buffer` 行是这样的 `buffer: '# Hexo Configuration\n## Docs: http://hexo.io/docs/configuration.html`。

也就是说，`buffer` 后的内容真是出现错误的信息！但我的报错怎么这么奇怪啊！等下，这个怎么那么像 `markdown` 的多行代码？我在哪里写了呢，于是我想到了 `scaffolds/post.md` 中我定义的模板，

```
---
title: {{ title }}
permalink: {{ title }}
date: {{ date }}
tags:
categories:
---
```

因为去年的时候我部署过 `hexo`，所以这次是直接粘贴上去的，粘贴的时候， `typora` 把 `---` 转化成了 「```」，问题解决！

### 报错二

hexo 的链接点击不跳转，变成下载文件

[升级Hexo到5.0 | TS' Blog](https://tang.su/2020/09/upgrade-hexo-to-5-0/#more)

### 错误三

我发现用 `npm` 安装 `next` 很不靠谱，`/themes` 文件夹中什么都没，所以涉及到编辑js的那些操作都做不了。于是检出了个 `bug_fix` 分支，删除了 `.gitkeep` 文件，`git clone` 了 `next` 的代码。后来又发现，主题内部自带了一份 `_config.yml` 文件，而且结构和之前的那份完全不一样！！！得知，根目录下的 `_config.yml` 的优先级比主题文件夹里面的优先级高，所以把配置复制到根目录下的配置文件，并且新的配置文件中是含有 `valine` 的默认配置的，很轻松就配置好了。

### 错误四

无法评论，需要在 `leancloud` 中【设置】-【安全中心】中设置安全域名，即自己的域名，形式为`https://你在cname中填写的域名`

### 错误五

访客显示错误，原因是域名映射的时候，`github` 给我们映射成了 `https`，但其实我是`http`，做如下设置即可

```yml
leancloud_visitors:
	security: false
```

