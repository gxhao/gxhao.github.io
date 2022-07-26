---
title: shell proxy via proxychains-ng
id: '32'
tags: ['shell','deploy','proxy']
categories:
  - - deployment
date: 2022-05-22 15:59:16
url: /shell-proxy
---


## Introduce


Usually when we use `GNU/Linux` we visit some external websites, and in many cases the relevant content is downloaded through a shell, such as `git`,`wget`,etc.

At this point we need to speed up access by setting up a proxy .[proxychains-ng](https://github.com/rofl0r/proxychains-ng) is very simple and convenient.

Although we can enable the current shell agent via export `export https_proxy=http://127.0.0.1:7890 http_proxy=http://127.0.0.1:7890 all_proxy=socks5://127.0.0.1:7890`,but he is not permanent solution.

## Download compile and install 

```shell
# download
git clone https://github.com/rofl0r/proxychains-ng

# enter folder
cd proxychains-ng

# complie
$ ./configure --prefix=/usr --sysconfdir=/etc
$ make
$ make install
$ make install-config (安装proxychains.conf配置文件)
```

## configration 


```shell
#View profile location
proxychains4 printenv` 

#edit profile
[ProxyList] 
socks5  127.0.0.1 7890
http    127.0.0.1 7890

# modify alias(zsh)
vi .zshrc
alias pc="proxychains4"
source .zshrc 
```
Congratulations,you now have fast access through a proxy 


Thanks
> https://www.hi-linux.com/posts/48321.html#vip-container


