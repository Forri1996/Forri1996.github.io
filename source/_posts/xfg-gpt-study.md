---
title: 小傅哥gpt课程学习记录
date: 2023-09-03 21:44:43
tags: 星球学习
---
# 环境准备
- docker
- java
- mvn
- git

安装好以上环境依赖以后，可以部署一下 portainer ，这是一款docker管理平台。安装以后可以方便后续操作。

# 部署流程
为了将java程序部署在docker中，需要一下几个步骤：
1. 打jar包 

通过 mvn package 在target目录下生成jar包即可。

2. 生成镜像

为了生成镜像，我们首先需要一份Dockerfile文件：
```
# 基础镜像
FROM openjdk:8-jre-slim
# 作者
MAINTAINER forri
# 配置
ENV PARAMS=""
# 时区
ENV TZ=PRC
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone
# 添加应用
ADD target/chatgpt-api.jar /chatgpt-api.jar
## 在镜像运行为容器后执行的命令
ENTRYPOINT ["sh","-c","java -jar $JAVA_OPTS /chatgpt-api.jar $PARAMS"]
```
有了该文件以后，在项目根目录执行命令：
```
docker build -f ./Dockerfile -t forri/chatgpt-api:1.5 .
```
随即便可以在本地生成一个镜像，可以通过
```
docker images
```
命令查看

3. push到dockerhub仓库

为了push到远端仓库，首先需要在本地登录dockerhub账号：
```
docker login
```
登录完成以后，执行命令：
```
 docker push forri/chatgpt-api:1.5
```
即可将打包的镜像推送到远端，然后在任何一个包含docker环境的服务器上，随取随用。

4. 在需要用的地方，拉取镜像后启动

```
docker pull forri/chatgpt-api:1.5
docker run -p 8080:8080 --name chatgpt-api -d forri/chatgpt-api:1.5
```