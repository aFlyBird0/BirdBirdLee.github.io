---
title: hexo+next+valine配置博客评论邮件通知（阿里云企业邮箱）
mylink: hexo+next+valine配置博客评论邮件通知（阿里云企业邮箱）
date: 2021-03-06 15:13:10
tags:
	- hexo
categories:
---

## 参考博客

> 不喜欢重复造轮子和复制粘贴别人的内容，这里在Valine-Admin官方教程和某博客的基础上，增加**阿里云企业邮箱**的配置部分，以及指出原博客中的一些错误

某博客：[Hexo主题使用Valine-Admin管理评论和评论提醒 - SegmentFault 思否](https://segmentfault.com/a/1190000021474516?utm_source=tag-newest)

Valine-Admin 官方教程：[zhaojun1998/Valine-Admin: 一个 Valine 的拓展应用，用来增强 Valine 的邮件通知。](https://github.com/zhaojun1998/Valine-Admin)

<!--more-->

## 邮件通知部署流程

完全按上面的「某博客」来即可，下面指出博客中的一些不清晰的地方以及错误

### 阿里云企业邮箱

用管理员账号，一般为 `postmaster@域名`，依次点击【组织与用户】-【员工账号管理】-【新建账号】

![image-20210306152129677](https://tcualhp-notes.oss-cn-hangzhou.aliyuncs.com/img/image-20210306152129677.png)

基本信息填写略，注意下图红框部分勾选

![image-20210306152223749](https://tcualhp-notes.oss-cn-hangzhou.aliyuncs.com/img/image-20210306152223749.png)

值得注意的是，我搜了半天，甚至登录了员工邮箱，都没找到 `SMTP` 授权码。经尝试，**授权码就是上图中给员工邮箱分配的密码**。

也就是说，`leancloud` 中要填写的环境变量有这些（如果用阿里云企业邮箱）

```
SITE_NAME : 网站名称。
SITE_URL : 网站地址, 最后不要加 / 。
SMTP_USER : 填写创建的员工邮箱地址（SMTP 服务用户名，一般为邮箱地址。）
SMTP_PASS : 阿里云为创建员工邮箱时分配的密码（SMTP 密码，一般为授权码，而不是邮箱的登陆密码，请自行查询对应邮件服务商的获取方式）
SENDER_NAME : 寄件人名称。
TO_EMAIL：填写自己的收件邮箱，所以的评论都会发给这个邮箱
TEMPLATE_NAME：设置提醒邮件的主题，目前内置了两款主题，分别为 default 与 rainbow。默认为 default
SMTP_HOST : 阿里云为：smtp.mxhichina.com（邮件服务提供商 SMTP 地址，如 qq : smtp.qq.com，此项需要自行查询或询问其服务商。）
SMTP_PORT : 阿里云为：25 （邮件服务提供商 SMTP 端口, 此项需要自行查询或询问其服务商。）
```

### 定时任务自动唤醒

还有个环境变量是 `ADMIN_URL`，是原博客中设置定时任务自动唤醒要用到的，设置方法如下：

进入应用，依次点击：【云引擎】-【web】-【设置】，下拉（其实和环境变量在同一页），找到【云引擎域名】，绑定一下，下图红框内的文字即为 `ADMIN_URL` 项应填入的值

![image-20210306153102967](https://tcualhp-notes.oss-cn-hangzhou.aliyuncs.com/img/image-20210306153102967.png)

### 原博客捡漏定时任务错误

博客中关于捡漏定时任务的描述中，截图和给出的代码不一致，（截图是正确的），这里我给出正确代码以供复制

```
0 0 8 * * ?
```



![image-20210306153610390](https://tcualhp-notes.oss-cn-hangzhou.aliyuncs.com/img/image-20210306153610390.png)

### next主题配置文件修改

还有一点博客没提到，原作者应该是默认大家已经会开启评论但是没设置邮件通知。

应该这样修改，`enable` 和 `notify` 改为 `true`，以及 `appid` 与 `appkey` 填完整

![image-20210306153258422](https://tcualhp-notes.oss-cn-hangzhou.aliyuncs.com/img/image-20210306153258422.png)

