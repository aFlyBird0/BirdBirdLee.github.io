---
title: 使用frpMgr快速搭建内网穿透
mylink: 使用frpMgr快速搭建内网穿透
date: 2020-04-21 13:28:07
tags:
categories:	
	- CS
---

## 概述

1. 安装基于 [frp](https://github.com/fatedier/frp) 的内网穿透管理面板 [frpMgr](https://github.com/Zo3i/frpMgr) 
2. 利用管理面板自动安装 frp server
3. 利用管理面板生成 frp client 安装语句，在待被穿透内网设备上安装
4. 使用 `ssh` 访问

<!--more-->

## 安装基于 [frp](https://github.com/fatedier/frp) 的内网穿透管理面板 [frpMgr](https://github.com/Zo3i/frpMgr) 

* **安装**
  
  1. **安装 docker 与 docker compose**
  
  	* 若无 docker :
  	
  	   ```
  	   wget -O-https://raw.githubusercontent.com/Zo3i/OCS/master/docker/docker-all2.sh | SH wget -O-
  	  ```
  	
  	* 若有 docker 无 docker compose : 
  	
  	  ```shell
  	  curl -L https://get.daocloud.io/docker/compose/releases/download/1.22.0/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
  	  chmod +x /usr/local/bin/docker-compose
  	  ```
  	
  	* 若 docker 和 docker compose都有， 跳过此步`
  	
  2. **安装 frpMgr**
  
     * `https://raw.githubusercontent.com/Zo3i/frpMgr/master/web/src/main/docker/final/run.sh | SH  `
     * 约 500 M，若下载太慢可以修改 host，或者用 [Github代下载](https://g.widora.cn/) 再传到服务器
  
  3. **开放防火墙 8999 端口**

## 利用管理面板自动安装 frp server

* 访问 `服务器域名（ip）:8999`   如 `xx.com:8999`，账户密码 admin / 12345678

* 准备一个域名，解析 *.xxx.com （二级通配）到 frp_server 的服务器 IP

* 配置 frp_server (图来自 frpMgr github)

  * 服务器名称，自定义名称，不解释
  * 服务器 IP：待安装 frp_server 的服务器IP
  * 域名：刚刚的一级域名
  * 访问端口：web 访问端口，如果配 web 访问要用到（和ssh无关），记得安全组/防火墙打开。
  * 服务器用户名：服务器用户名。
  * 依次点击 【保存】【远程安装】，输入服务器密码，即可看到安装成功提示
  * 在服务器执行 `netstat -lnp|grep 7000` 查看服务是否成功启动

  ![](https://camo.githubusercontent.com/778ec45220f7dc5fae657beb12f3f21016985174/68747470733a2f2f7a78782e6f6e652f696d67732f323031392f31312f623965373761363035663330396231362e706e67)

## 利用管理面板生成 frp client 安装语句，在待被穿透内网设备上安装

* 选择 `SSH客户端配置`， 点击右上角 `新增`
* ![](http://qiniu.tcualhp.cn/frp.png)
* 填写 client 配置
  * 项目名称随意
  * 远程端口为之后访问外网的端口，如你之后想通过 `*.xx.com:8090` 访问内网，就填8090， 同样防火墙和安全组要打开
  * 服务器选上一步添加的服务器
* 填写成功后点击右侧 `linux`，复制安装命令，在 client 安装即可。（client 需安装 ssh, 并开放22端口）

## 使用ssl访问

* ip 为 frp_server域名/IP，端口为 client 配置的端口