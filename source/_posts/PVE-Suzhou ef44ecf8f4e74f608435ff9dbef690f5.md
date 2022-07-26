---
title: pve安装openwrt、nas简单教程
id: '32'
tags: ['pve','openwrt','nas']
categories:
  - - deployment
date: 2022-11-22 15:59:16
url: /pveinstall
---

## 基础配置

### 镜像配置

```bash
## 进入系统软件源配置文件目录
cd /etc/apt

## 将默认软件源配置文件进行备份
cp sources.list sources.list.bak 

## 替换系统软件仓库

sed -i 's|^deb http://ftp.debian.org|deb https://mirrors.ustc.edu.cn|g' /etc/apt/sources.list

sed -i 's|^deb http://security.debian.org|deb https://mirrors.ustc.edu.cn/debian-security|g' /etc/apt/sources.list 

# 订阅源替换

## 创建 PVE 免费源

source /etc/os-release

echo "deb https://mirrors.ustc.edu.cn/proxmox/debian/pve $VERSION_CODENAME pve-no-subscription" > /etc/apt/sources.list.d/pve-no-subscription.list 
```

### 桌面配置

- 更新配置源

```bash
## 更新配置源
# 將enterprise ppa 移出套件庫
mv /etc/apt/sources.list.d/pve-enterprise.list /etc/apt/sources.list.d/pve-enterprise.list.bk
# 將 non-enterprise ppa 加入套件庫
echo "deb http://download.proxmox.com/debian/pve buster pve-no-subscription" > /etc/apt/sources.list.d/pve-no-enterprise.list

# 更新套件清單及系統
apt update -qq &&  apt upgrade -y -qq

# 重開機
reboot

```

- 安装桌面

```bash

#  安裝sudo等工具 (Option)
apt install -y -qq sudo tmux vim htop

# 安裝Gnome桌面環境
tasksel install desktop gnome-desktop

#  移除 network-manager (Option)
apt remove network-manager-gnome  network-manager

# 啟用桌面環境
systemctl set-default graphical.target

# 加入使用者
adduser <USERNAME>

# 加入sudo群組(option)
usermod -a -G sudo <USERNAME> 

# 重開機
reboot
```

## 安装OpenWrt

- 上传img2kvm、 openwrt.img
- 执行镜像转换

```jsx
chmod +x img2kvm

./img2kvm Openwrt.img 100 vm-100-disk-1
```

- pve中添加这个磁盘并且修改启动顺序
- openwrt开机，并修改ip

```jsx
vi etc/config/network 

reboot
```

![Untitled](PVE-Suzhou%20ef44ecf8f4e74f608435ff9dbef690f5/Untitled.png)

## NAS安装

### 上传镜像

```bash
./img2kvm synoboot.img 101 vm-101-disk-1
```

### pve配置

```bash
网络选择e1000、内存2G，磁盘类型为sata

```

### 安装

打开地址：[http://192.168.1.201:5000/](http://192.168.1.201:5000/)

![Untitled](PVE-Suzhou%20ef44ecf8f4e74f608435ff9dbef690f5/Untitled%201.png)

- 选择手动安装，选择`DSM_DS3615xs_23739.pat`

![Untitled](PVE-Suzhou%20ef44ecf8f4e74f608435ff9dbef690f5/Untitled%202.png)

- 配置Storage Pool 和Volume

![Untitled](PVE-Suzhou%20ef44ecf8f4e74f608435ff9dbef690f5/Untitled%203.png)

- Windows挂载NAS

![Untitled](PVE-Suzhou%20ef44ecf8f4e74f608435ff9dbef690f5/Untitled%204.png)

## 链接

### u盘处理

[U盘启动盘怎么复原为普通U盘](https://zhuanlan.zhihu.com/p/37772825)

### pve初始化

[[ Proxmox 折腾手记 ] PVE初始化配置](https://www.bilibili.com/read/cv17670431)

[[ Proxmox 折腾手记 ] PVE系统调整](https://www.bilibili.com/read/cv17671660?spm_id_from=333.999.0.0)

### 安装桌面环境

[Proxmox 安裝 Gnome](https://blog.404nofound.com/post/install-on-proxmox/)

### OpenWrt作为旁路网关

[Openwrt 作为旁路网关（不是旁路由、单臂路由）的终极设置方法，破解迷思 - 少数派](https://sspai.com/post/68511)