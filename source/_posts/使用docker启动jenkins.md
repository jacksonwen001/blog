---
title: 使用docker启动jenkins
date: 2022-03-28 11:09:41
tags: docker
---
# 概述
使用 `docker` 启动 `jenkins` 

# 命令
```shell
docker run -d --name jenkins -v /Users/xxx/jenkins_home:/var/jenkins_home -p 8999:8080 -p 50000:50000 jenkins/jenkins:lts-jdk-11
cat ~/jenkins_home/secrets/initalAdminPassword 
得到相关密钥
```
需要注意的点：
1. 挂载文件夹的时候一定要使用绝对路径，不能使用相对路径，不然会报错
2. 密钥位于 `jenkins_home/secrets/initalAdminPassword` 

