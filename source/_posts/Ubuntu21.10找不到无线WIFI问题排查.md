---
title: Ubuntu 21.10 安装后找不到无线wifi问题排查
id: '32'
tags: ['ubuntu','linux']
categories:
  - - deployment
date: 2022-07-22 15:59:16
url: /ubuntu-wifi
---

## 起因

家用笔记本在更新win11 最新的更新后频繁出现系统黑屏进入休眠状态的问题，但是我系统设置`休眠状态`为永不休眠。一气之下选择更换系统为 `Ubuntu（21.10）`（听说gnome40挺好用的

装好后出现了找不到wifi模块的问题，因为笔记本没有网线接口，有线网络也没法使用了。经过网上的搜索排查，最终解决，以下记录一下解决的思路。

## 解决

### 确定网卡型号

通过厂商的下载中心，[华硕全球支持中心](https://www.asus.com/support/Download-Center/)查询到我这台电脑的网卡型号为**MT7921**。

![图片1]()

### 是否是驱动问题

查看相关问题说MT7921在Linux内核为5.12以下不会自动识别,但是Ubuntu21.10内核为lunux kernel 5.13.应该不是这个问题

### linux防火墙

看[这个帖子](https://miloserdov.org/?p=6899)说即使`linux`内核高于5.12也会因为防火墙问题导致无法查找wifi模块，安装linux防火墙
```
sudo apt install linux-firmware
```
### 关闭快速启动

在[archlinux](https://bbs.archlinux.org/viewtopic.php?id=267229) 的论坛上翻到一个安装了防火墙仍然不起作用的回答。
> There is a bug report with not much more information in the upstream repository:
https://github.com/openwrt/mt76/issues/548  
If you dual-boot with windows, disable its fast-startup mode. https://wiki.archlinux.org/title/Dual_b … ibernation

定位到github issue,通过完全关闭电脑，并且拔掉电源线再启动解决。幸运的是我也是通过这种方法解决的。

## 参考

> https://github.com/openwrt/mt76/issues/548

> https://bbs.archlinux.org/viewtopic.php?id=267229

> https://askubuntu.com/questions/1346617/ubuntu-20-04-does-not-have-mediatek-driver-mt7921-for-wifi-bluetooth

> https://miloserdov.org/?p=6899

