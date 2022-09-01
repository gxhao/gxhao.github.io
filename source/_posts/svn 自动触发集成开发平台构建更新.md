---
title: 利用Svn Hooks触发自动部署流水线
id: '32'
tags: ['DevOps','Gitlab','CI/CD']
categories:
  - - deployment
date: 2022-01-31 15:59:16
url: /svn-cicd
---
## 缘起

项目上使用[集成开发平台](https://dev.epoint.com.cn/clouddev-web/frame/fui/pages/themes/clouddev/clouddev)构建部署已经有一段时间了，因为项目上使用的是svn，所以在更新系统的时候仍然需要**提交代码**->**集成开发平台更新**->**服务器远程构建更新**。由于提交代码和集成开发平台构建更新都是在公司内网进行的，那么我们可不可以将前两步合并为一步？也就是提交完代码**自动**进行集成开发平台更新，接着只需要在服务器端远程构建更新就可以了。


## 思路

1. gitlab 有个配置项[Webhooks](http://xxx/mtjt/custom/-/hooks)可以触发集成开发平台构建。 
> Webhooks enable you to send notifications to web applications in response to events in a group or project. 

2. svn应该有相关触发事件
3. 编写个后台Restful接口，svn触发这个接口，这个接口触发集成开发平台更新

## 调研与设计

### svn监听如何实现？
#### 1. svn本身仓库自带了hooks功能   
在svn仓库的hooks文件夹也自带了相关的模板。配置相关shell命令即可使用
![](https://gxhao.aiur.site/svnhook1.png)
_没有访问项目svn仓库的权限，此实现方法不考虑。_(否决)
#### 2. svn 开发人员开发了基于apache开源的svn提交监听工具-[CommitMonitor](https://tools.stefankueng.com/CommitMonitor.html)  
_这个工具可以监听svn提交操作，然后发送通知在桌面右下角。但是于我而言，我不需要他在右下角提醒我，我需要他提供一个通知的api，我可以利用这个api去做一些别的事情。很遗憾没有找到_（否决）

#### 3. TortoiseSVN 提供的[Hooks Script](https://tortoisesvn.net/docs/release/TortoiseSVN_en/tsvn-dug-settings.html#tsvn-dug-settings-hooks)脚本
查看官方介绍，貌似此方法是一种比较可行的方法，且貌似官方文档上没有限定必须是什么类型的脚本。

### 脚本如何选择？

#### 1.shell 脚本

直接编写shell执行访问Rest接口或者相关jar包，但是这样在执行后会有一堆提交报错（实际是执行成功的），类似这样：
![](https://gxhao.aiur.site/svnhook2.png)
这可能导致体验不太好，~~明明我提交了完了，你却说我提交错了~~

#### 2.JavaScript

js可以编写http请求的代码，然后利用`node`去执行脚本。考虑环境兼容性，大部分人电脑应该没有安装node相关工具包。

#### 3.[Wscript](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/wscript)

这是微软在`Windows98`上就推出的Windows系统自带的脚本，可以理解为`Windows js`,这样环境就不用担心了，开始编写`Wscript`脚本代码。


## 开发与测试

### 编写代码
Wscript有个很方便的对象（`WinHttp.WinHttpRequest.5.1`），可以直接访问http请求。
编写Wscript代码如下
```
var http = WScript.CreateObject("WinHttp.WinHttpRequest.5.1"); 
http.Open("POST", "http://xxx/start", false);
http.SetRequestHeader("Accept", "*/*");
http.SetRequestHeader("Accept-Language", "zh-cn,zh");
http.SetRequestHeader("User-Agent", "Mozilla/6.0");
http.SetRequestHeader("Content-Type", "application/json");
// http.SetRequestHeader("Referer", "http://www.baidu.com/");
// http.SetRequestHeader("Connection", "Close ");
// http.SetRequestHeader("Content-Length", "100");
JSONString = "{\"pipelineGuid\" : \"xxxxx\", \"loginId\": \"gxhao\"}";
http.Send(JSONString);
http.WaitForResponse(1000);
WScript.Echo(http.ResponseText);
```
### 配置TortoiseSvn

![](https://gxhao.aiur.site/svnhook.png)

`Hook Type`选择Post-Commit-Hook，即在提交完成时触发。`Working Copy Path` 选择你要提交的代码目录（可配置多个）。`Command Line To Execute`选择你要执行的Wscript脚本(注意双引号)。

### 测试提交

结果大概时这样：  

![](https://gxhao.aiur.site/svnhook3.png)




## 目前存在的一些问题

1. 只能在TortoiseSvn的工具中提交才能触发，ide提交不会触发。
2. 集成平台构建完成后没有相关通知到msg/邮件。


## 结语

引用：  

> https://tortoisesvn.net/docs/release/TortoiseSVN_en/tsvn-dug-settings.html#tsvn-dug-settings-hooks  

> https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/wscript

感谢：

**非常感谢中研院庄晟琪，顾疆飞两位大佬的帮助和支持**，希望此脚本能够对大家在开发时有所帮助，提高开发效率，减少更新构建时间。

