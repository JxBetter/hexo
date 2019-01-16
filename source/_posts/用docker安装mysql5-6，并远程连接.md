---
title: 用docker安装mysql5.6，并远程连接
date: 2019-01-16 10:31:54
top: 1
tags: 
	- docker
	- mysql
categories: 
	- docker
---
# 用docker安装mysql:5.6镜像
> 运行命令: docker pull mysql:5.6

# 查看下载的docker镜像
> 运行命令: docker images
> REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
> mysql                  5.6                 27e29668a08a        2 weeks ago         256MB

# 创建docker容器
> tips: MySQL(5.7.19)的默认配置文件是 /etc/mysql/my.cnf 文件。如果想要自定义配置，建议向 /etc/mysql/conf.d 目录中创建 .cnf文件。新建的文件可以任意起名，只要保证后缀名是 cnf 即可。新建的文件中的配置项可以覆盖 /etc/mysql/my.cnf 中的配置项。
> 1. 创建要映射到容器中的.cnf文件
>> mkdir -p docker_v/mysql/conf
>> cd docker_v/mysql/conf
>> touch my.cnf
> 2. 启动容器
>> docker run -p 3306:3306 --name mysql -v /opt/docker_v/mysql/conf:/etc/mysql/conf.d -e MYSQL_ROOT_PASSWORD=123456 -d imageID
>>> -p 3306:3306：将容器的3306端口映射到主机的3306端口
>>> -v /opt/docker_v/mysql/conf:/etc/mysql/conf.d：将主机/opt/docker_v/mysql/conf目录挂载到容器的/etc/mysql/conf.d
>>> -e MYSQL_ROOT_PASSWORD=123456：初始化root用户的密码
>>> -d: 后台运行容器，并返回容器ID
>>> imageID: mysql镜像ID

# 查看启动的mysql容器
> 运行命令: docker ps

# navicat远程连接mysql
## 新建连接
> 点击新建连接，选择mysql
## 更改配置
> 在常规选项中，输入主机名，mysql的端口号，mysql的用户名和密码
> 在SSH选项中，输入主机ip，主机端口号，主机用户名和密码
> 点击测试连接
