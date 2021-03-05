---
title: Docker安装与使用IDEA部署到远程服务器
mylink: Docker安装与使用IDEA部署到远程服务器
date: 2019-08-04 15:50:33
tags:
    - docker
categories:
    - CS
    - common
---
Docker教程链接与自己总结，涉及教程，安装，远程配置，IDEA调用

<!--more-->
> 如果是想学习可以跳过这段，首先我想学docker, 查了下好像docker for windows只能win10专业版安装，所以我招了教程利用各种帖子，后来安装失败，于是我在服务器上安装，很成功。之后发现可以用IDEA一键部署到服务器，就配置了一下，因为安全组关系又浪费不少时间，后来链接成功，但打包失败，我以为本地也要装，于是又试了一下，结果一下子就好了。可是IDEA打包还是失败，还来发现了什么，重启了一下IDEA，一切就都可以了，真是吐血，本地应该不需要装Docker，直接调用远程服务器的tcp的2375接口。



## Docker 教程

* [Docker 从入门到实践](https://yeasy.gitbooks.io/docker_practice/introduction/what.html)



## Docker 两个版本区分

* [dockerToolbox和docker for windows的区别](https://blog.csdn.net/JENREY/article/details/84493812)



## Docker 安装

### Docker for Windows 安装

* 安装
  * [Windows10家庭版安装Docker for Windows](https://www.cnblogs.com/samwu/p/10360943.html)
* 与虚拟机冲突解决办法
  * [【Hyper-V】与【VirtualBox】【VMware】冲突的解决方法](https://blog.csdn.net/qwsamxy/article/details/50533007)

### Docker Toolbox 安装

* [Docker ToolBox安装](https://www.cnblogs.com/jeshy/p/10518857.html) 无脑安装

### Docker Linux 安装

* 过于简单，无脑输命令就行， 第一条教程链接里面各个发行版本安装方法都有





## IDEA 连接远程 Docker 进行部署

### 远程服务器 Docker 设置(其实就是开放2375端口)

* docker 配置
  * [Ubuntu配置方法](https://blog.51cto.com/709151/2406150)
  * [CentOS配置方法](https://blog.csdn.net/u012946310/article/details/82315302)
* 服务器配置
  * 防火墙2375端口打开
  * 如果是阿里云要新建安全组规则加入2375
  * 如果是宝塔也要开放2375端口（注意这两个都要开放，我之前以为宝塔能直接改动阿里云的安全组就只开放了宝塔，结果一直连接超时）

### IDEA 设置

* [docker部署spring cloud项目](https://blog.csdn.net/forezp/article/details/70198649)
* 记得IDEA在maven引入docker之后重启一下，不然可能会出现我之前的无法连接到 "localhost:2375" 错误

## 部分错误文档

* [Dockerfile ADD路径不正确问题](https://blog.csdn.net/ii19910410/article/details/87882917)
* [这个也不错](https://blog.csdn.net/ChineseYoung/article/details/83107353)