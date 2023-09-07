---
title: chatgpt工程部署
date: 2023-09-03 21:44:43
tags: 星球学习
---
# 环境准备
- docker
- java
- mvn
- git

安装好以上环境依赖以后，可以部署一下 portainer ，这是一款docker管理平台。安装以后可以方便后续操作。

# 打包java程序并部署
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


# 启动代理网关
执行命令
```
docker run --restart always --name Nginx -d -p 80:80 nginx
```

# 配置路由
现在我们有一个nginx container，有一个java container，

接下来，我们需要通过配置一个nginx server，实现反向代理。为了修改容器配置，我们需要将容器内的文件挂在到宿主机
```
docker run \
  --name nginx \
  -d -p 80:80 \
  -v /usr/local/nginx/nginx.conf:/etc/nginx/nginx.conf \
  -v /usr/local/nginx/conf.d:/etc/nginx/conf.d \
  -v /usr/local/nginx/log:/var/log/nginx \
  nginx
```

这样，可以实现在宿主机配置nginx文件。接下来，我们增加fotech.conf作为反向代理的配置，并把该域名下所有的流量转发到chatgpt容器，下面是一份nginx server配置模板
```
 server {
        listen       80;
        server_name  fotech.cn;

        location / {
            root   html;
            index  index.html index.htm;
	    proxy_pass  http://chatgpt-api:8080;
        }

        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
}
```

为了实现容器间的服务发现功能，需要创建一个自定义的network
```
docker network create nginx-net
```

并且在启动容器的时候，指定network

这里我们重启一下nginx服务和java服务
```
docker run -p 8080:8080 \
--name chatgpt-api \
--network nginx-net \
-d forri/chatgpt-api:1.5  


docker run \
  --name nginx \
  --network nginx-net \
  -d -p 80:80 \
  -v /usr/local/nginx/nginx.conf:/etc/nginx/nginx.conf \
  -v /usr/local/nginx/conf.d:/etc/nginx/conf.d \
  -v /usr/local/nginx/log:/var/log/nginx \
  nginx
```

注意，这里需要先启动chatgpt服务，再启动nginx，否则nginx会由于找不到服务而启动失败。

在宿主机配置host
```
127.0.0.1 fotech.cn
```
访问http://fotech.cn/success成功！



# 配置自动化部署
为了实现自动化的容器部署，使用github action能力实现：
- 提交代码到main分支后，自动触发docker build 打包镜像上传dockerhub
- ssh虚拟机，自动拉取最新镜像并部署

由于项目中依赖了chatgpt-sdk-java这个包，但是这个包没有上传到maven中央仓库，所以在打包的时候会提示包找不到的错误

本地可以通过手动install来解决，但是如果要用github action，这个方案就行不通了。

两个办法：
- 将sdk集成到data工程
- 上传sdk到中央仓库

这里使用第一个方案。

为了验证确实部署了新的代码上容器，新增一个接口用于部署后的验证：
```
127.0.0.1:8091/api/v1/auth/testcicd
```

准备好代码后，继续下面的操作（代码修改好后记得在本地 mvn package 并且能成功打包，再继续）

操作步骤
- 注册dockerhub并获得accesstoken，并配置到github action的variables中
- 获取部署服务器的ssh私钥(注意是私钥)，并配置到github action的variables中
- 将服务器公钥添加到keys文件(注意是公钥)
```
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
```
在代码根目录添加文件夹：.github/workflows/maven.yml
代码内容：
```
# This workflow will build a Java project with Maven, and cache/restore any dependencies to improve the workflow execution time
# For more information see: https://docs.github.com/en/actions/automating-builds-and-tests/building-and-testing-java-with-maven

# This workflow uses actions that are not certified by GitHub.
# They are provided by a third-party and are governed by
# separate terms of service, privacy policy, and support
# documentation.

name: Java CI with Maven

on:
  push:
    branches: [ "main" ]

env:
  REGISTRY: docker.io  # 默认为 docker.io，即去 Docker Hub 上找
  IMAGE_NAME: ${{ github.event.repository.name }}  # 使用 GitHub Actions 提供的能力，可以自动获取仓库名
  IMAGE_TAG: latest  # Docker Image 的 tag，为了方便我直接设置 latest
  PORT: 8091  # 服务要暴露的端口，可以改

jobs:
  build:

    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v3
      - name: Set up JDK 8  # 可以改版本
        uses: actions/setup-java@v3
        with:
          java-version: '8'  # 可以改版本
          distribution: 'temurin'
          cache: maven
      - name: Build with Maven
        run: mvn -B package --file pom.xml

      # Login against a Docker registry except on PR
      # https://github.com/docker/login-action
      - name: Log into registry
        if: github.event_name != 'pull_request'
        uses: docker/login-action@v2
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ secrets.DOCKER_HUB_USER }}
          password: ${{ secrets.DOCKER_HUB_TOKEN }}

      # Build and push Docker image with Buildx (don't push on PR)
      # https://github.com/docker/build-push-action
      - name: Build and push Docker image
        uses: docker/build-push-action@v3
        with:
          context: .
          push: ${{ github.event_name != 'pull_request' }}
          tags: ${{ secrets.DOCKER_HUB_USER }}/${{ env.IMAGE_NAME }}:${{ env.IMAGE_TAG }}

      # 连接到远程服务器
      - name: Connect to server
        uses: webfactory/ssh-agent@v0.4.1
        with:
          ssh-private-key: ${{ secrets.SSH_PRIVATE_KEY }}

      # 初始化 knownhosts
      - name: Setup knownhosts
        run: ssh-keyscan ${{ secrets.SERVER_HOST }} >> ~/.ssh/known_hosts

      # 触发服务器部署脚本
      - name: Trigger server deployment script
        run: |
          ssh root@${{ secrets.SERVER_HOST }} "docker stop ${{ env.IMAGE_NAME }} || true"
          ssh root@${{ secrets.SERVER_HOST }} "docker rm ${{ env.IMAGE_NAME }} || true"
          ssh root@${{ secrets.SERVER_HOST }} "docker pull ${{ secrets.DOCKER_HUB_USER }}/${{ env.IMAGE_NAME }}:${{ env.IMAGE_TAG }}"
          ssh root@${{ secrets.SERVER_HOST }} "docker run -p ${{ env.PORT }}:${{ env.PORT}} --name ${{ env.IMAGE_NAME }} -d ${{ secrets.DOCKER_HUB_USER }}/${{ env.IMAGE_NAME }}:${{ env.IMAGE_TAG }}"
```

提交代码，等待流水线执行完成后，到Portainer上检查部署情况，成功！
![image](https://github.com/Forri1996/blog-talk/assets/128824087/ca87ecae-d812-4e17-8697-cb3ffc92840c)
![image](https://github.com/Forri1996/blog-talk/assets/128824087/85942aca-9659-432c-a250-4cf0f96a7f57)
![image](https://github.com/Forri1996/blog-talk/assets/128824087/4265198c-3287-4453-947a-8b7fddb9c8a7)

这样，后续迭代开发只需要在dev分支开发测试完成，然后merge到main分支后提交，即可自动部署到云服务。

参考文档：[通过 GitHub Actions 完成 Spring Boot 项目的 CI/CD（基于 Docker）](https://zhuanlan.zhihu.com/p/590967429)