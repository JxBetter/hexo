---
title: 应用服务器与WSGI协议以及flask后端框架总结(后端接收请求返回响应的整个流程)
date: 2019-01-14 10:29:59
top: 1
tags: 
	- flask
	- python
categories: 
	- python
---
> 1.  上次遗留了两个问题,先说一下自己的看法  
>     
>     * * *
>     
>     问题:  
>     1.明明一个线程只能处理一个请求,那么栈里的元素永远是在栈顶,那为什么需要用栈这个结构?用普通变量不行吗.  
>     2.\_request\_ctx\_stack和\_app\_ctx\_stack都是线程隔离的,那么为什么要分开?
>     
>     * * *
>     
>     我认为在web runtime的情况下是可以不需要栈这个结构的,即使是单线程下也不需要,原本我以为在单线程下,当前一个请求阻塞后,后一个请求还会被推入栈中,结果并不是这样,这也就说明了,栈的结构和是不是单线程没关系,为了验证这点,我写了个简单的接口验证这点:
>     
```
>         from flask import Flask,_request_ctx_stack
>         
>         app = Flask(__name__)
>         
>         
>         @app.route('/')
>         def index():
>             print(_request_ctx_stack._local.__storage__)
>             time.sleep(1)
>             return '<h1>hello</h1>'
>         
>         app.run(port=3000)
>         -------------------------------------------
>         def wsgi_app(self, environ, start_response):
>         
>             ctx = self.request_context(environ)
>             ctx.push()
>             print(_request_ctx_stack._local.__storage__)
```
>     
>     我在Flask类中的wsgi\_app()方法中加了这一句print(\_request\_ctx\_stack.\_local.\_\_storage__),wsgi_app()是后端接收应用服务器发来的包装好的WSGI请求的函数,后面会讲到,由于一个线程只能处理一个请求,所以结果应该是栈中永远只有一个请求对象,在路由接口中我延时了1秒,假设成阻塞,看一下结果:
>     
```
>          * Running on http://127.0.0.1:3000/ (Press CTRL+C to quit)
>         {139851542578944: {'stack': []}}
>         127.0.0.1 - - [14/Apr/2018 14:31:17] "GET / HTTP/1.1" 200 -
>         {139851542578944: {'stack': []}}
>         127.0.0.1 - - [14/Apr/2018 14:31:18] "GET / HTTP/1.1" 200 -
>         {139851542578944: {'stack': []}}
>         127.0.0.1 - - [14/Apr/2018 14:31:19] "GET / HTTP/1.1" 200 -
```  
>     
>     每次栈中只有一个请求对象,这也就说明了栈这个结构和web runtime下的单线程无关,那么就剩下非web runtime的情况了,最常见的是离线测试:
>     
```
>         from flask import Flask,_request_ctx_stack,_app_ctx_stack
>         
>         
>         app = Flask(__name__)
>         app2 = Flask(__name__)
>         
>         
>         def offline_test():
>             with app.app_context():
>                 print(_app_ctx_stack._local.__storage__)
>                 with app2.app_context():
>                     print(_app_ctx_stack._local.__storage__)
>         
>             with app.app_context():
>                 with app.test_request_context():
>                     print(_request_ctx_stack._local.__storage__)
>                     with app.test_request_context():
>                         print(_request_ctx_stack._local.__storage__)
```
>     
>     离线测试是单线程的,通过这个例子也能得到第二的问题的答案,为什么要将请求和应用分开,一个原因是flask支持多个app共存,这需要用到中间件,另一个原因是离线测试时,有可能只需要用到应用上下文,所以需要将两者分开,在离线测试时如果进行了嵌套则栈结构的特点就发挥了出来,看一下运行的结果:
>     
```
>         {140402410657536: {'stack': []}}
>         {140402410657536: {'stack': [, ]}}
>         {140402410657536: {'stack': []}}
>         {140402410657536: {'stack': [, ]}}
```  
>     
>     结果显而易见  
>     总结一下:栈结构和分离请求和应用是为了离线测试更加灵活
>     
>     * * *
>     
> 2.  web应用服务器 WSGI 后端之间的关系  
>     web应用服务器的作用是监听端口,当接收到客户端的请求后将请求转化成WSGI格式(environ)然后传给后端框架  
>     应用服务器<----WSGI协议---->后端框架  
>     WSGI是应用服务器和后端框架之间的桥梁,使得服务器和后端框架分离,各司其职,程序员也能专注于自己的逻辑  
>     在WSGI中规定了每个python web应用都需要是一个可调用的对象,即实现了\_\_call\_\_这个特殊方法,Flask就是一个可调用对象
> 
> * * *
> 
> 4.  web应用服务器从哪里将包装好的请求发送给后端  
>     在flask中使用了werkzeug这个工具包,在werkzeug.serving中有一个类,class WSGIRequestHandler(BaseHTTPRequestHandler, object)  
>     这个类提供了environ字典对象,定义了start\_response()和run\_wsgi()方法,在run_wsgi()中有一个execute(),看一下源码:
>     
```
>         def execute(app):
>             application_iter = app(environ, start_response)  #从这里发送到后端
>             try:
>                 for data in application_iter:
>                     write(data)
>                 if not headers_sent:
>                     write(b'')
>             finally:
>                 if hasattr(application_iter, 'close'):
>                     application_iter.close()
>                 application_iter = None
```
>     
>     第一句application\_iter = app(environ, start\_response)就调用了Flask.\_\_call\_\_(),并将environ, start\_response传入,而Flask.\_\_call__()就return了self.wsgi_app(),  
>     这个wsgi\_app(environ, start\_response)是一个标准的请求处理函数,所以它就是整个后端处理请求的入口函数,environ是一个包含所有HTTP请求信息的字典对象,start\_response是一个发送HTTP响应的函数,environ是从应用服务器传过来的,start\_response是定义好的,这些都不需要后端开发人员关心  
>     总结一下:  
>     1.WSGI规定了后端处理函数的格式,即需要接受environ,start_response这两个参数,这两个参数从应用服务器传给后端框架  
>     2.python web应用对象需要是可调用的,即实现了\_\_call\_\_方法,返回WSGI规定格式的后端处理函数来处理请求及返回响应  
>     3.应用服务器会使用werkzeug.serving中的WSGIRequestHandler类中的相应方法,将http请求转化成WSGI格式,所以说werkzeug是一个遵循WSGI协议的工具包,提供给应用服务器使用  
>     
> 
> * * *
> 
> 6.  后端处理请求返回响应整个流程  
>     之前说到,后端处理请求的入口函数是wsgi\_app(self,environ,start\_response),先看下源码:
>     
```
>             def wsgi_app(self, environ, start_response):
>                 ctx = self.request_context(environ)  #1
>                 ctx.push()  #2
>                 error = None
>                 try:
>                     try:
>                         response = self.full_dispatch_request()  #3
>                     except Exception as e:
>                         error = e
>                         response = self.handle_exception(e)
>                     except:
>                         error = sys.exc_info()[1]
>                         raise
>                     return response(environ, start_response)
>                 finally:
>                     if self.should_ignore_error(error):
>                         error = None
>                     ctx.auto_pop(error)
```
>     
>     其中有三句比较关键,我写了序号  
>     第一句:self.request\_context(environ),看下request\_context这个方法:
>     
```
>             def request_context(self, environ):
>                 return RequestContext(self, environ)
```
>     
>     简而言之,传入environ,初始化一个请求上下文对象并返回  
>     第二句:ctx.push(),看下源码:
>     
```
>             def push(self):
>                 top = _request_ctx_stack.top
>                 if top is not None and top.preserved:
>                     top.pop(top._preserved_exc)
>         
>                 app_ctx = _app_ctx_stack.top
>                 if app_ctx is None or app_ctx.app != self.app:
>                     app_ctx = self.app.app_context()
>                     app_ctx.push()
>                     self._implicit_app_ctx_stack.append(app_ctx)
>                 else:
>                     self._implicit_app_ctx_stack.append(None)
>         
>                 if hasattr(sys, 'exc_clear'):
>                     sys.exc_clear()
>         
>                 _request_ctx_stack.push(self)
>         
>                 self.session = self.app.open_session(self.request)
>                 if self.session is None:
>                     self.session = self.app.make_null_session()
```
>     
>     简而言之,推入应用上下文和请求上下文,如果设置了secret_key则开启一个session,关于flask的session放到后面说  
>     第三句:self.full\_dispatch\_request(),是处理请求的关键函数,看下源码:
>     
```
>             def full_dispatch_request(self):
>                 """Dispatches the request and on top of that performs request
>                 pre and postprocessing as well as HTTP exception catching and
>                 error handling.
>         
>                 .. versionadded:: 0.7
>                 """
>                 self.try_trigger_before_first_request_functions()
>                 try:
>                     request_started.send(self)
>                     rv = self.preprocess_request()  #function1
>                     if rv is None:
>                         rv = self.dispatch_request()  #function2
>                 except Exception as e:
>                     rv = self.handle_user_exception(e)
>                 return self.finalize_request(rv)  #function3
```
>     
>     这个函数中嵌套了另外三个函数,预处理函数preprocess\_request(),主处理函数dispatch\_request()和最终处理函数finalize_request(rv)  
>     1.preprocess\_request()是处理被before\_request装饰器装饰的函数  
>     2.dispatch_request()匹配请求的URL,并返回视图函数的返回值rv  
>     3.finalize\_request(rv)接受视图函数的返回值,并生成响应,这里有make\_response和process\_response这两个函数,make\_response生成响应对象,process_response对响应做一些处理,比如后面要讲到的session  
>     响应生成后,在wsgi\_app中return response,最后调用ctx.auto\_pop()将请求和应用上下文推出栈,return的response会通过start_response发送到应用服务器,并由其发送到客户端,这样一次请求就结束了.
> 
> * * *
> 
> 8.  最后说说session  
>     flask中的session是client side session,说白了就是session会封装在cookie中在最终响应时会发送给客户端,而在服务器本地不会存储,所以叫作client side session,要使用session需要设置secret\_key这个配置,通过app.secret\_key来设置,用来验证签名,等到下次客户端发来带有cookie的请求时,后端就能从生成对应的session中解析出带有的信息,写个简单的应用来看下session怎么用:
>     
```
>         from flask import Flask,session
>         
>         
>         app = flask.Flask(__name__)
>         app.secret_key = 'gjx'
>         
>         
>         @app.route('/')
>         def index():
>             if 'name' in session:
>                 print(session['name'])
>             else:
>                 print('stranger')
>             return '<h1>/</h1>'
>         
>         
>         @app.route('/')
>         def test(name):
>             session['name'] = name
>             print('session set successful')
>             return '<h1>test</h1>'
>         
>         
>         app.run(port=3000)
```
>     
>     跑起来后,在浏览器输入127.0.0.1:3000/,会打印出stranger,  
>     然后访问127.0.0.1:3000/jx后,后端打印出session set successful,并且浏览器会收到服务器发来的cookie,  
>     Set-Cookie:session=eyJuYW1lIjoiangifQ.DbNHuQ.MPZLWzoLdga2SPMg0plMYmKlJMc; HttpOnly; Path=/ 这是我测试时收到的,有三个字段,第一个是session的内容,第二个是时间戳,第三个是验证信息  
>     这时已经设置好了session,并且得到了cookie,再次访问127.0.0.1:3000/,后端打印出了jx,就是之前设置的值  
>     如果对session内的值更改,则返回的cookie也会更改,那么在那保存,在那创建session呢?  
>     之前在分析后端请求流程是提到了,在RequestContext的push方法最后:
>     
```
>                 self.session = self.app.open_session(self.request)
>                 if self.session is None:
>                     self.session = self.app.make_null_session()
```
>     
>     如果设置了secret\_key则会执行open\_session开启一个session,那如果更改了在哪里保存呢?  
>     在finalize\_request执行的self.process\_response中:
>     
```
>             def process_response(self, response):
>                 ctx = _request_ctx_stack.top
>                 bp = ctx.request.blueprint
>                 funcs = ctx._after_request_functions
>                 if bp is not None and bp in self.after_request_funcs:
>                     funcs = chain(funcs, reversed(self.after_request_funcs[bp]))
>                 if None in self.after_request_funcs:
>                     funcs = chain(funcs, reversed(self.after_request_funcs[None]))
>                 for handler in funcs:
>                     response = handler(response)
>                 if not self.session_interface.is_null_session(ctx.session):
>                     self.save_session(ctx.session, response)
>                 return response
```
>     
>     在最后判断如果session不是null session的话会执行self.save\_session来保存更新session,在self.save\_session中会调用response.set_cookie,flask中的session大概就是这样
> 
> 总结一下:  
> 1.分析了应用服务器封装好的environ从哪发送给后端  
> 2.分析了应用服务器 WSGI 后端之间的关系以及WSGI协议对接口的标准定义,使得后端人员只需要关心自己的逻辑  
> 3.分析了后端接收到应用服务器发来的WSGI请求之后的一系列处理流程,主要函数是wsgi\_app(environ,start\_response)  
> 4.最后简单分析了flask中的session机制,它是client side session的.