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