# Clash for Linux 部署
## 前言

Clash for Windows 是一款非常好用的代理软件，它通过简单的配置能实现通过代理的网路自动切换直连或者代理。也提供了允许局域网其他设备连接和游戏模式等非常好i用的功能。在Windows的图形化界面操作非常友好。由于我家里有台设备是Linux Ubuntu。想通过这个软件走代理。以下是配置三种方法

## linux常规安装
### 1. 下载文件

  ```
  wget  https://github.com/Dreamacro/clash/releases/download/v1.8.0/clash-freebsd-amd64-v1.8.0.gz
  ```
### 2. 解压
```
  gunzip
```
### 3. 配置
  将订阅的配置文件移动到解压目录并重命名为`config.yaml`
### 4、启动
```
chmod +x ./clash-linux-XXXX
./clash-linux-XXXX -d .
```
### 5、自启动
```
#创建service文件
touch /etc/systemd/system/clash.service

#编辑如下文本： 
[Unit] 
Description=clash daemon  
[Service] 
Type=simple 
User=root 
ExecStart=/home/username/下载/Clash/clash -d /home/username/下载/Clash/ 
Restart=on-failure  
[Install] 
WantedBy=multi-user.target

## 刷新系统服务并设置为自启动
sudo systemctl daemon-reload 
sudo systemctl enable clash 
sudo systemctl start clash 
```


--- 
##  docker部署
需要先在本地创建config.yaml文件
### 1. 编辑docker-compose配置文件
```
version: '3'
services:
  clash:
    # ghcr.io/dreamacro/clash
    # ghcr.io/dreamacro/clash-premium
    # dreamacro/clash
    # dreamacro/clash-premium
    image: dreamacro/clash
    container_name: clash
    volumes:
      - ./config.yaml:/root/.config/clash/config.yaml
      # - ./ui:/ui # dashboard volume
    ports:
      - "7890:7890"
      - "7891:7891"
      # - "8080:8080" # external controller (Restful API)
    restart: unless-stopped
    network_mode: "bridge" # or "host" on Linux
```
### 2. 启动
```
$ docker-compose up -d
```

### 3. 访问

http://clash.razord.top/#/proxies

## 通过[flatpak](https://flatpak.org/setup/Ubuntu)安装clash for windows
 
```
flatpak install clash 
```


## 参考
> https://www.cnblogs.com/rogunt/p/15127947.html #普通方式部署clash

> https://github.com/Dreamacro/clash/wiki/clash-as-a-daemon#docker #docekr 部署clash