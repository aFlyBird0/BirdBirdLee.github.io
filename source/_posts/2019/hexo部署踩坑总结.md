---
title: hexo部署踩坑总结(文末有自动部署备份脚本)
mylink: hexo部署踩坑总结
date: 2019-08-04 15:50:32
tags:
    - hexo
categories:
    - CS
    - hexo
---

hexo安装，部署，主题，备份教程，备份为自己尝试很多次的总结，附自动部署备份脚本

<!--more-->

## 链接
* [官方教程](https://hexo.io/zh-cn/docs/) 官方教程肯定是最权威的最好的教程
* [使用hexo+github搭建免费个人博客详细教程](https://www.cnblogs.com/liuxianan/p/build-blog-website-by-hexo-github.html)
* [域名绑定](https://blog.enjoytoshare.club/article/hexo-do-domain.html) 实现用自己的域名访问博客
* [原文件备份](https://blog.csdn.net/u012195214/article/details/72721065) `master` 用于静态界面，另外新建一个分支，比如 `hexo` 用来备份原文件和配置等，[点我跳转到备份详细过程防踩坑](#deploy_backup)

##  `hexo` 更新

* 如果以前下载过 `hexo`，记得升级，我原来用的2， 现在已经 3.9 了，被坑死

## 主题

*  [更换主题](https://www.jianshu.com/p/33bc0a0a6e90) 注意文章的主题地址可能过时
* **next** 主题已经**更换地址**了， 如果看的是原来的教程，`clone` 时记得**更换 next 的 Github 地址**

<h3 id="deploy_backup"> 部署与备份</h3>
* 前期工作

  * 突然发现官网写的很清楚，[hexo部署](https://hexo.io/zh-cn/docs/deployment.html)

  * 记得在 `hexo-d` 前修改站点配置文件即 `_config.yml` 中的 `deploy` 配置， 示例， 注意替换具体仓库地址

    ```yaml
    deploy:
      type: git
      repo: <repository url> 
      branch: master
      # message: [message]
    ```

    

  * 如果 `hexo -d` 报错就看 [这个链接](https://blog.csdn.net/weixin_36401046/article/details/52940313)

* 备份说明

  * 命令： `hexo d`
  * 内部实现：其实是 `hexo-depolyer-git` 将 `hexo-g` 后生成的 `public` 文件打包。所以 `hexo-d` 的时候他会把 `public` 也就是静态文件自己上传到 `git`远程仓库的 `master` 分支（ git 地址和 master 分支在站点配置文件那边要提前配置好），我们不用管。换句话说也就是如果你不备份，只需要做到这里就好了，这个命令完成的是对静态文件的上传。
  * 备份：所以像前面说的，我们要备份的话，就在不影响 `master` 的情况下，新建一个分支，比如 `hexo` , 平时写文章就在 `hexo` 分支， 并且**不用切换回master**！！！，因为我前面讲到 master 分支也就是静态 `public` 文件的部署已经交给 `hexo-d` 全权负责了！我之前就是切回去所以错了
  * 附录一（`.gitignore`）

  ```sh
  .DS_Store
  Thumbs.db
  db.json
  *.log
  node_modules/
  public/
  .deploy*/
  auto_deploy.sh
  ```

  * 附录二（**自动部署备份脚本**）

  > **使用说明：每次写完文件保存后 bash 执行 `sh ./auto_deploy.sh "提交的message"` 即可， 文件名自定**

  ```sh
  # 切回hexo, 以防万一
  git checkout hexo
  # 提交本地的
  git add .
  git commit -m "$1"
  # 提交到远程
  git push origin hexo
  # 到页面发布分支
  # 清理原静态文件
  hexo clean
  # 生成，部署
  hexo g -d
  ```

  