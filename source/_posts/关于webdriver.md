---
title: 关于webdriver
date: 2019-01-24 14:56:30
top: 1
tags: 
	- 测试
categories: 
	- 软件测试
---
## webdriver协议
> webdriver协议是一套json格式的规范，本身是基于http协议的
> 这个协议规定了每个操作对应的数据格式，webdriver作为一个服务端，需要实现协议中的每一个操作
> 作为客户端的库文件需要封装好给用户使用的api，每个api对应着协议中不同的数据格式，这些数据封装在http中的body中，数据格式内容和具体的操作一一对应
> selenium中的webdriver就是浏览器驱动，比如ChromeDriver等，驱动实现了webdriver协议
> appium中的webdriver是appium server
通俗地说：
> 由于客户端脚本(java, python, ruby)不能直接与浏览器/手机通信，这时候可以把webdriver server当做一个翻译器，它可以把客户端代码翻译成浏览器/手机可以识别的代码(比如js)，客户端通过http请求向webdriver server发送restful的请求，webdriver server翻译成浏览器/手机懂得脚本传给浏览器/手机，浏览器/手机把执行的结果返回给webdriver server,webdriver server把返回的结果做了一些封装(JSON Wire protocol)，然后返回给客户端脚本，客户端根据返回值就能判断对浏览器/手机的操作是否执行成功
> 协议就像是一个抽象类，规定了方法，以及触发方法所需要的数据格式和内容，但是没有具体实现；服务端需要具体去实现这些方法；客户端则需要按照协议规定的数据格式和内容去封装提供给用户的api

## selenium中的WebDriver类
> selenium作为一个客户端，提供给用户的接口基本都在selenium/webdriver/remote/webdriver.py中的WebDriver类中实现
> 这个类是selenium中所有关于浏览器driver类的基类
截取Chrome webdriver类中初始化的一段代码：
```
try:
    RemoteWebDriver.__init__(
        self,
        command_executor=ChromeRemoteConnection(
            remote_server_addr=self.service.service_url,
            keep_alive=keep_alive),
        desired_capabilities=desired_capabilities)
except Exception:
    self.quit()
    raise
self._is_remote = False
```
可以看到调用了基类的初始化方法，来连接到webdriver server，在此之前会寻找浏览器驱动，并自动启动服务：
```
self.service = Service(
    executable_path, # 默认为chromedriver
    port=port,
    service_args=service_args,
    log_path=service_log_path)
self.service.start() #启动server
```
之后，脚本调用对应的api，就会向这个server发送符合webdriver协议规范的http请求，server接收请求来操作浏览器

## appium中的WebDriver类
> appium中的WebDriver继承自很多类，功能更加丰富，当然它也继承了selenium中的webdriver基类
```
class WebDriver(
    ActionHelpers,
    Activities,
    Applications,
    Clipboard,
    Context,
    DeviceTime,
    HardwareActions,
    ImagesComparison,
    IME,
    Keyboard,
    Location,
    Network,
    RemoteFS,
    ScreenRecord
):
```
> 在初始化中，command_executor参数填写的是appium server的地址+端口号，这个appium server我们在运行脚本前需要手动启动
> 之后脚本调用对应的api，就会向这个appium server发送符合webdriver协议规范的http请求，appium server接收请求来操作手机

## 总结
### webdriver中的三个角色
1. 测试脚本——作为客户端
2. 浏览器驱动/appium server——作为服务端
3. 浏览器/手机——作为服务端操作的对象

### 客户端包的作用
> 屏蔽有关协议的内容，让用户不必关心这些细节，只需使用提供给用户的api即可完成相应的操作