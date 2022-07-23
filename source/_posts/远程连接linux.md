---
title: 远程连接linux桌面的几种方式
id: '32'
tags: ['linux','remote']
categories:
  - - deployment
date: 2022-07-22 15:59:16
url: /remote-linux
---
# 前言
家里有个老电脑想使用起来，最适合的就是linux系统了，安装了Ubuntu。除了远程命令执行外还需要远程窗口帮助家里长辈处理一些问题。问题描述如下:  
v2ex类似问题：[远程访问 Linux 桌面最快速最高效的方式是什么 VNC,RDP?](https://v2ex.com/t/462243)
# 使用花生壳做端口映射
无公网ip的首选方案，但是限制网速1m，网络总流量1gb。有点太少了。一天基本就要用500m。
## xrdp
在linux安装xrdp并在花生壳开启3389端口映射。事实体验并不太好。画面卡断。操作不流畅
## teamview
```shell
下载teamView
wget https://download.teamviewer.com/download/linux/teamviewer_amd64.deb
安装 teamView
sudo apt install ./teamviewer_amd64.deb
``` 
在安装过程中，官方 TeamViewer 软件源已经被添加到你的系统中。你可以使用 cat 命令来验证文件内容：
``` 
cat /etc/apt/sources.list.d/teamviewer.list

deb http://linux.teamviewer.com/deb stable main
```
缺点teamViewer在Linux端经常掉线
## VNC远程链接
[官方文档](https://help.realvnc.com/hc/en-us/articles/360002253198-Installing-and-Removing-VNC-Connect#downloading-vnc-server-0-20)  

很详细的一个帖子
> https://bytexd.com/how-to-install-configure-vnc-server-on-ubuntu-20-04/

开启vnc服务
> vncserver -localhost no :1  

关闭vnc服务
> vncserver -kill :1

编辑桌面配置文件

> sudo apt install ubuntu-budgie-desktop

> sudo vim ~/.vnc/xstartup

```
#!/bin/sh
xrdb $HOME/.Xresources
exec budgie-desktop &
```

--- 

# 使用ZeroTier做内部局域网

官网：[ZeroTier](https://announce.zerotier.com/en)
- 支持linux、ios、Widows、nas
- 操作简单

## Linux端使用

```shell 
curl -s https://install.zerotier.com | sudo bash
# 查看相关信息
zerotier-cli info
# 加入局域网
zerotier-cli join 83048a0632fdcd69
# 卸载
yum erase zerotier-one
```
## Windows下载

> https://download.zerotier.com/dist/ZeroTier%20One.msi	

# 使用openssh for Windows 作为代理转发内网(未成功)

[如何使用windows远程控制一台服务器](https://cloud.tencent.com/developer/article/1420930)

[如何通过 SSH 在家里访问公司的内网网站？](https://www.v2ex.com/t/493717)

--- 

# 使用Widows端口转发工具（NATBypass）

[NATBypass_github](https://github.com/cw1997/NATBypass)

相关教程 ：  
> https://cloud.tencent.com/developer/article/1365274
https://apt404.github.io/2016/09/11/htran-portforward/
https://cloud.tencent.com/developer/article/1365274  
https://xz.aliyun.com/t/6349#toc-22

