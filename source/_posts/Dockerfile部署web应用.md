---
title: Dockerfile部署web应用
date: 2019-01-19 15:39:02
top: 1
tags: 
	- docker
categories: 
	- docker
---
# Dockerfile部署web应用
> 在[记录一次web测试](https://easythink.top/2019/01/14/%E8%AE%B0%E5%BD%95%E4%B8%80%E6%AC%A1web%E6%B5%8B%E8%AF%95/#more)这一篇博文中，我把此次测试所产出的文件都放在了自己的服务器上，并提供了下载接口，正好最近在学习docker，今天我把这个web应用用docker跑起来。

## 编写Dockerfile
```
FROM python               # 拉取python基础镜像
COPY . /server/           # 将当前项目文件加下的所有文件拷贝到docker容器中的server文件夹
WORKDIR /server           # 容器内切换到server目录
RUN pip install flask	  # 在镜像安装flask
RUN pip install gunicorn  # 在镜像中安装gunicorn
EXPOSE 8081               # 暴露出8081端口
CMD gunicorn -b 0.0.0.0:9998 -w 4 server:app    # 最终运行服务器的命令
```

## 编译docker镜像
```
docker build -t test:v1    # test:v1 --> 自定义镜像名:标签
```

## 运行容器
```
docker run -d --name test_server -p 8081:8081 test:v1
```

## 访问
> 访问ip:port即可