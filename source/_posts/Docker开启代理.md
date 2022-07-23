---
title: docker开启代理
id: '32'
tags: ['docker','proxy']
categories:
  - - deployment
date: 2022-07-22 15:59:16
url: /dockerproxy
---
# 前言
在国内网路情况下docker 拉取DockerHub仓库的镜像是非常慢的，这时候我们经常使用镜像加速器来解决这一问题，但是镜像加速器并不是始终生效的，在一些情况下可能会失效。
[docker rigster test](https://github.com/docker-practice/docker-registry-cn-mirror-test/actions)每天提供了镜像的测试功能。如果你经常使用代理的话，这边可以配置Docker代理。

# 安装
> The Docker daemon uses the HTTP_PROXY, HTTPS_PROXY, and NO_PROXY environmental  variables in its start-up environment to configure HTTP or HTTPS proxy behavior. You  cannot configure these environment variables using the daemon.json file.  

## 1. 创建`docker.service.d`文件夹
```
sudo mkdir -p /etc/systemd/system/docker.service.d
```
## 2. 创建`http-proxy.conf`文件

```
touch http-proxy.conf

```
## 3. 编写文件内容
```
[Service]
Environment="HTTPS_PROXY=127.0.0.1:7890"
```
## 4.重启docker
```
 sudo systemctl daemon-reload
 sudo systemctl restart docker
```
## 5. 验证
```
sudo systemctl show --property=Environment docker
```

# FAQ
## 拉取镜像报错
> Using default tag: latest
Error response from daemon: Get "https://registry-1.docker.io/v2/": proxyconnect tcp: EOF

- 重置dns解析服务器
https://www.cxyzjd.com/article/a460467324/117607753
- 检查`/etc/systemd/system/docker.service.dhttp-proxy.conf` 配置文件
# 参考

> https://docs.docker.com/config/daemon/systemd/#httphttps-proxy