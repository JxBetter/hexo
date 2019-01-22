---
title: 用appium测试app
date: 2019-01-22 13:49:55
top: 1
tags: 
	- appium
	- 测试
categories:
	- 软件测试
---
# appium
## 简介
> 关于appium的一些介绍已经在[selenium和appium内部原理总结](https://easythink.top/2019/01/21/selenium%E5%92%8Cappium%E5%86%85%E9%83%A8%E5%8E%9F%E7%90%86%E6%80%BB%E7%BB%93/)中总结了，
> appium是一个c/s架构的工具，server端是一个node.js启动的服务器，client端是使用对应开发包写的脚本，脚本发送请求给server，server操作设备端的app

## 环境搭建
### node.js
```
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
brew install node
node -v
npm -v
```
### java
在官网下载jdk并安装
安装完后可在/Library/Java/JavaVirtualMachines/目录下找到对应的jdk目录
在/etc/profile中添加环境变量
example:
```
export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.8.0_40.jdk/Contents/Home
export PATH=$JAVA_HOME/bin:$PATH:.
source /etc/profile
```

### android sdk
在[这里](https://www.androiddevtools.cn/)下载
下载完后解压进入tools目录，运行./android sdk
安装platform-tools、build-tools
在/etc/profile中添加环境变量
```
export ANDROID_HOME=mysdk_dir
export PATH=$PATH:$ANDROID_HOME/tools
export PATH=$PATH:$ANDROID_HOME/platform-tools
source /etc/profile
```

### appium server
Desktop版本：[下载地址](https://github.com/appium/appium-desktop/releases)
命令行版本： npm install appium -g

### appium client
python版本：pip install Appium-Python-Client

### appium-doctor
npm install appium-doctor -g
这个命令可以检查当前appium环境是否配置正确
可能出错的地方：jdk地址、android sdk地址、node.js

### android 模拟器
夜神：[下载地址](https://www.yeshen.com/)
打开模拟器后使用adb devices可能会找不到模拟器，把android sdk中platform-tool中的adb重命名成nox_adb覆盖掉夜神中的nox_adb
点击设置，关于平板电脑，连续点击 **安卓版本**，打开开发者模式
重启adb就能找到模拟器设备了

## 测试实例
等待更新