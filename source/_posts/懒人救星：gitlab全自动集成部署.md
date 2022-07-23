# 懒人救星：gitlab ci/cd 全自动部署实践

## 背景
由于项目上需要，上周申请了公司的服务器作为演示临时使用。我使用常规方式，将部署包上传到服务器，编写dockerfile文件，然后使用docker容器部署。做完这一套大概要半个小时，并且我还得全程监控服务器的状态，命令一个一个手敲，之后的更新还有重复上面的步骤，这对我这种懒人来说实在是太麻烦了。突然想到公司在大力推进集成开发平台CI/CD持续集成、持续部署，那我是不是也可以用这种方式快速部署开发，提升工作效率。

## 引言
受顾佳文[新点易维背后的故事（一）：“终端妙囊，百宝锦囊”的从 0 到 1](https://oa.epoint.com.cn/EpointCommunity/EpointCommunity/Dis/ShowTopic.aspx?TopicID=29303)这篇帖子的启发，再加上公司目前已经部署了gitlab代码托管服务器，利用周末的时间研究了下gitlab的ci/cd流程。内心os：那我是不是可以通过gitlab来实现**全自动**构建部署?
## 实战
1、先写一个最简单的web服务，可以参考这个
> http://192.168.0.200/mtjt/java-servlet-hello/-/tree/master  

2、部署gitlab Runer 
### 安装Runner执行器
```
# 下载执行器
sudo curl -L --output /usr/local/bin/gitlab-runner https://gitlab-runner-downloads.s3.amazonaws.com/latest/binaries/gitlab-runner-linux-amd64
# 赋予文件夹权限
chmod +x /usr/local/bin/gitlab-runner
# 新建操作用户
useradd --comment 'gitlab-runner' --create-home gitlab-runner --shell /bin/bash
# 添加 docker 用户组
usermod -a -G docker gitlab-runner
# gitlab-runner 作为管理员执行
sudo visudo -f /etc/sudoers.d/gitlab-runner
gitlab-runner ALL=(ALL) NOPASSWD: ALL
# 安装
gitlab-runner install --user=gitlab-runner --working-directory=/home/gitlab-runner
# 启动
gitlab-runner start
# 验证
gitlab-runner verify 
```
### 注册Runner执行器
```
gitlab-runner register 

选择shell 
It supposes you have configured a shell Runner and have installed Docker.
```
### 效果
- 整体效果
![gitlab-ci1](https://note.youdao.com/yws/api/personal/file/A732675291764B7FA6EF496DDAFB8DE4?method=download&shareKey=cf18868bf11347e64d0e8410f748c777)
- 局部管道
![gitlab-ci2](https://note.youdao.com/yws/api/personal/file/DD0B7E1588E047C7BE4250FB5157F611?method=download&shareKey=1ba1ed9773f99d68801fc49d1695fbda)
- 实际结果
![gitlab-cd1](https://note.youdao.com/yws/api/personal/file/DAE56F52D63D4EA19E7579EC5F948F35?method=download&shareKey=eed42c83b5ce17eb9d2d644d2abc5787)

## 他有什么作用？
###  提高开发人员效率
当开发人员将代码合并到gitlab-ci.yaml文件所在的分支，会**自动触发**持续集成、持续部署流程、不必将代码手动上传至服务器，然后一步步等待服务器执行。开发人员能够专注与系统设计和开发，加快客户响应速度。如果管道构建失败，会通过oa发送邮件提醒，类似这样：
![gitlab-cd2](https://note.youdao.com/yws/api/personal/file/8703BFE3C0B441AA958F0EAA18B1B0BE?method=download&shareKey=f041327e21086d384d276d0691b6865d)
### 公司条线整改能够及时更新
目前整改基本上是一个月一次，但是实际情况是开发负责人并不能在发布整改后立即更新，尤其是当开发负责人负责8、9个项目的时候、整改对于开发负责人造成了额外的负担。如果可能的话，公司可以添加一系列安全验证，开放maven仓库，gitlab通过运行maven命令实现从公司仓库拉取最近的发布包，及时完成整改。
![gitlabcd3](https://note.youdao.com/yws/api/personal/file/91107AE83B4E4437B776CE40B7AB0A7A?method=download&shareKey=22783e09eee20e8e886d3781b370f108)
### 满足条线在更新服务器之前先进行单元测试的要求
通过在gitlab-ci.yaml 中引入test管道，可以实现在更新服务器之前，先通过test管道访问云测平台接口，自行编写测试用例等方式保证更新到服务器的代码是通过完整测试的代码。
![gitlab-test](https://note.youdao.com/yws/api/personal/file/A1861E8959B5402E94BD265C16EE382B?method=download&shareKey=6a4d7ea7a1abb507a4362423fce79f9d)
### 一键部署到生产、仿真系统
在测试系统验证完成后，配置手动触发部署正式服务器管道，可以一键完成正式服务器部署，避免二次打包。

## 下一个目标规划
- 部署harbor镜像仓库，允许版本回退
- docker-compose，多容器部署。
- 利用ssh部署到远程服务器 
> https://docs.gitlab.com/ee/ci/ssh_keys/
