---
title: 为hexo-next安装侧边rss
mylink: 为hexo-next安装侧边rss
date: 2021-03-06 21:25:23
tags:
	- hexo
categories:
	- CS
	- hexo
---

## 步骤

### 1. 在博客根目录下安装[ hexo-generator-feed](https://github.com/hexojs/hexo-generator-feed)

```sh
npm install hexo-generator-feed --save
```

<!--more-->

### 2. 在博客根目录下的 `_config.yml` 文件中添加以下代码

```yml
feed:
  type: atom
  path: atom.xml
  limit: 20
  hub:
  content:
  content_limit: 140
  content_limit_delim: " "
  order_by: -date
  icon: icon.png
```

### 3. 配置博客根目录下的 `_config.yml` 文件

```yml
url: 自己域名
```

### 4. 修改 `Next` 主题的配置文件 `_config.yml`

首先，这里的主题配置文件最好是根目录下的 `_config_next.yml`，如果没有的话，建议把 `/thems/next/_config.yml` 文件复制到根目录下，并改名成 `_config_next.yml`。

不直接修改`/themes/next/_config.yml` 的好处是，当拉取新的更新时，不会因修改了`_config.yml`而发生冲突。（每个主题都是一个独立的 `git`项目）

`rss` 有两种形式，一种是侧边栏，一种是每个文章底部都有个大大的`rss`区域，推荐侧边烂。(缩进都是两格，我这里代码可能缩进格数不对)

侧边栏形式：

```yml
social:
	RSS: /atom.xml || rss
```

文章底部形式：

```yml
follow_me:
	RSS: /atom.xml || rss
```

## 参考博客

[Hexo安装Next主题 Rss 侧边栏 | 漂自己的移，让别人都撞墙去吧](https://www.gagahappy.com/use-next-theme/)

[给 Hexo 中的 Next 主题添加 RSS 功能 | 苏寅 Blog](https://suyin-blog.club/2020/2M3YWE7/)