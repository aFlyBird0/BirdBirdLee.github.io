---
title: hexo5.0+安装报错总结
mylink: hexo5.0+安装报错总结
date: 2021-03-05 16:33:10
tags:
categories:
	- CS
---

## 参考链接

`hexo` 官方文档：[文档 | Hexo](https://hexo.io/zh-cn/docs/)

`hexo` 中 `_posts` 文件夹分类：

[如何在Hexo中对文章md文件分类_一个码农的博客-CSDN博客](https://blog.csdn.net/maosidiaoxian/article/details/85220394)

[如何对hexo中的文章进行分类管理 | 大专栏](https://www.dazhuanlan.com/2020/03/28/5e7e34ca352b3/)

添加看板娘：[Hexo添加Live2D看板娘最新教程_enchanted的博客-CSDN博客](https://blog.csdn.net/qq_36239569/article/details/104104894)

https://tang.su/2020/09/upgrade-hexo-to-5-0/#more)

[Hexo NexT 主题优化：显示文章阅读次数 | 九月枫林](http://www.yangyong.xyz/2018/01/03/add-hexo-next-post-views/)

[hexo博客next主题添加 评论功能_Mumu's Blogs-CSDN博客](https://blog.csdn.net/zhu_1997/article/details/87554975)

<!-- more -->

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