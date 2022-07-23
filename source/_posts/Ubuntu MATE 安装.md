---
title: Ubuntu MATE Install
id: '32'
tags: ['ubuntu','linux']
categories:
  - - deployment
date: 2022-07-22 15:59:16
url: /ubuntu-install
---

# Ubuntu MATE安装及初始配置
## 前言
家里有个老电脑、还是在我上初中的时候买的，到现在他的使用率也很低了。考虑配置为amd双核、4G内存、250G硬盘，拿来搭一个小型的Linux服务器，用来学习和跑一些常用的软件还是很不错的。以下是操作详情。

## 系统下载及安装
> Ubuntu MATE is an impressive lightweight Linux distro that runs fast enough on older computers. It features the MATE desktop – so the user interface might seem a little different at first but it’s easy to use as well.

此次使用的是**Ubuntu MATE**,如果你需要使用其他系统，以下操作方式类似
### 1. 下载iso镜像文件
[官方地址](https://ubuntu-mate.org/download/amd64/)
### 2. 刻录镜像文件到u盘
使用[rufus](https://rufus.ie/en/)刻录工具进行镜像刻录，使用默认配置。
### 3. 系统安装
开机按`F12`选择u盘进行默认安装，**图形化安装**基本使用默认配置安装就可以。  
界面大概是这样
![](https://note.youdao.com/yws/api/personal/file/129741D0E59A44EB9969ADFDDB1A010F?method=download&shareKey=c36fcbdab60d0ee03411abcd6efe0778)
## 软件安装
系统安装完之后需要一些必要的软件安装及配置
### 1、SSH服务端
初始系统中是没有ssh软件的，我们也就无法通过ssh远程访问，需要安装ssh。
```
# 安装ssh
sudo apt install ssh
# 启用ssh服务
sudo service ssh enable
```
### 2、zeroTier安装（可选）
因为家里是内网，如果不在家连接到电脑，需要安装[ZeroTIer](https://www.zerotier.com/download/)。
```
# 下载curl工具
sudo apt install curl
# 下载zerotier
curl -s https://install.zerotier.com | sudo bash
# 查看相关信息
zerotier-cli info
# 加入局域网，xxx为zero中心的16位网络标识码
sudo zerotier-cli join xxx
# 卸载
yum erase zerotier-one
```
### 3、speedtest（可选）
下载完第一件事当然是测个速了
```
curl -s https://install.speedtest.net/app/cli/install.deb.sh | sudo bash
# 安装
sudo apt-get install speedtest
# 使用
speedtest
``` 
测完我们发现网速一般情况下没跑到宽带网速上限，以下需要设置一下。
```
## 安装ethtool工具
sudo apt install ethtool
## 设置网速
sudo ethtool -s enp2s5 speed 100 duplex full autoneg on
```
设置开机自启动(Ubuntu)
```
## 创建文件
/etc/systemd/system/foo.service
## 添加脚本
[Unit]
Description=Job that runs your user script

[Service]
ExecStart=/some/command
Type=oneshot
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target
## 执行脚本
sudo systemctl daemon-reload
sudo systemctl enable foo.service
```

### 4、docker 安装
docker作为容器，部署使用常用应用非常方便，安装一下
```
# 添加软件源密钥
curl -fsSL https://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
# 添加docker软件源
echo  "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://mirrors.aliyun.com/docker-ce/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
# 更新缓存
sudo apt-get update
# 安装docker
sudo apt-get install docker-ce docker-ce-cli containerd.io

```
### 5、安装clash for linux
```
wget -P /home/gxhao/Downloads https://github.com/Dreamacro/clash/releases/download/v1.9.0/clash-linux-amd64-v1.9.0.gz
``` 
### 6、关闭系统休眠
```
#查看休眠状态
systemctl status sleep.target
# 执行关闭休眠功能
sudo systemctl mask sleep.target suspend.target hibernate.target hybrid-sleep.target
```


## 引用
Ubuntu MATE介绍
> https://itsfoss.com/ubuntu-mate-20-04-review/  

网速为10mb/s设置
> https://phoenixnap.com/kb/ethtool-command-change-speed-duplex-ethernet-card-linux

网速设置为自启动
> https://portal.whitesandshosting.com/index.php/knowledgebase/6/Linux---change-the-speed-and-duplex-settings-of-an-Ethernet-card.html

如何设置命令行开机自启动
> https://askubuntu.com/questions/814/how-to-run-scripts-on-start-up