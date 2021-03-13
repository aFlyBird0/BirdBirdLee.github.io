---
title: 解决hexo博客leancloud访问量永远是1的错误
mylink: 解决hexo博客leancloud访问量永远是1的错误
date: 2021-03-13 16:22:27
tags:
	- hexo
categories:
	- CS
    - hexo
---

## 错误定位

定位到错误：这条请求标红，并且含有 `Counter`，我猜是 `leancloud` 修改访问量的接口

![image-20210313162321026](https://tcualhp-notes.oss-cn-hangzhou.aliyuncs.com/img/image-20210313162321026.png)

返回是这样的：![image-20210313162351656](https://tcualhp-notes.oss-cn-hangzhou.aliyuncs.com/img/image-20210313162351656.png)

应该是`Couter`类的权限设置问题，默认设置是创建所有人可创建，但访问者没有写（更新）的权限，导致阅读量一直是1。

<!--more-->

## 错误解决

进入`leancloud`控制台，【存储】-【结构化数据】-【Counter】，删除 Class

![image-20210313162937093](https://tcualhp-notes.oss-cn-hangzhou.aliyuncs.com/img/image-20210313162937093.png)

再创建一个 `Counter` Class，注意下面选【无限制】

![image-20210313163038465](https://tcualhp-notes.oss-cn-hangzhou.aliyuncs.com/img/image-20210313163038465.png)

## 参考

[Forbidden writing by object's ACL. - Yanting Ou](https://www.ouyanting.com/archives/2018/12/ebe3ded2.html)