---
title: 使用docker启动jenkins
date: 2022-03-28 11:09:41
tags: docker
---
# 概述
使用 `docker` 启动 `jenkins` 

# 命令

需要注意的点：
1. 挂载文件夹的时候一定要使用绝对路径，不能使用相对路径，不然会报错
2. 密钥位于 `jenkins_home/secrets/initalAdminPassword` 

## 创建网络
```shell
docker network create jenkins
```

## 挂载 docker
```shell
docker run --name jenkins-docker --restart=always --detach \
  --privileged --network jenkins --network-alias docker \
  --env DOCKER_TLS_CERTDIR=/certs \
  --volume jenkins-docker-certs:/certs/client \
  --volume /Users/jackson/software/jenkins/jenkins_home:/var/jenkins_home \
  --publish 2376:2376 \
  docker:dind --storage-driver overlay2
```
## 创建 docker file
```
FROM jenkins/jenkins:2.332.1-jdk11
USER root
RUN apt-get update && apt-get install -y lsb-release
RUN curl -fsSLo /usr/share/keyrings/docker-archive-keyring.asc \
  https://download.docker.com/linux/debian/gpg
RUN echo "deb [arch=$(dpkg --print-architecture) \
  signed-by=/usr/share/keyrings/docker-archive-keyring.asc] \
  https://download.docker.com/linux/debian \
  $(lsb_release -cs) stable" > /etc/apt/sources.list.d/docker.list
RUN apt-get update && apt-get install -y docker-ce-cli
USER jenkins
RUN jenkins-plugin-cli --plugins "blueocean:1.25.3 docker-workflow:1.28"
```
## 运行

```shell
  docker run \
  --name jenkins-blueocean \
  --restart=always \
  --detach \
  --network jenkins \
  --env DOCKER_HOST=tcp://docker:2376 \
  --env DOCKER_CERT_PATH=/certs/client \
  --env DOCKER_TLS_VERIFY=1 \
  --publish 8888:8080 \
  --publish 50000:50000 \
  --volume /Users/jackson/software/jenkins/jenkins_home:/var/jenkins_home \
  --volume jenkins-docker-certs:/certs/client:ro \
    myjenkins-blueocean:2.332.1-1 
```

注意： 参考以下链接， 避免浪费时间
# 参考：
https://www.jenkins.io/doc/book/installing/docker/
