---
title: flask接收请求并推入栈
date: 2019-01-13 17:36:58
top: 1
tags: 
	- flask
	- python
categories: 
	- python
---
# flask接收请求并推入栈
前面两篇讲明了flask怎么支持多线程以及怎么开启多线程的,这篇来讲讲当后端接收到请求后是怎么一步步封装的

* 1.Flask类中的wsgi_app()
	> 当应用启动后WSGI Server会通过Flask.__call__()接收http请求,Flask.__call__()中返回的是wsgi_app()方法,

```
    def wsgi_app(self, environ, start_response):
        ctx = self.request_context(environ)
        ctx.push()
        error = None
        try:
            try:
                response = self.full_dispatch_request()
            except Exception as e:
                error = e
                response = self.handle_exception(e)
            except:
                error = sys.exc_info()[1]
                raise
            return response(environ, start_response)
        finally:
            if self.should_ignore_error(error):
                error = None
            ctx.auto_pop(error)

```
	wsgi_app()主要做了两件事情:
第一件事是通过Flask的另一个方法request_context()返回得到了一个封装好的RequsetContext对象,
ctx = self.request_context(environ),然后调用RequestContext中的push(),
```
class RequestContext(object):
 
    def push(self):
        app_ctx = _app_ctx_stack.top
        if app_ctx is None or app_ctx.app != self.app:
            app_ctx = self.app.app_context()
            app_ctx.push()
            self._implicit_app_ctx_stack.append(app_ctx)
        else:
            self._implicit_app_ctx_stack.append(None)
 
        if hasattr(sys, 'exc_clear'):
            sys.exc_clear()
 
        _request_ctx_stack.push(self)
 
        self.session = self.app.open_session(self.request)
        if self.session is None:
            self.session = self.app.make_null_session()

```

在最后调用了_request_ctx_stack.push(self),将请求对象推入请求上下文栈中
第二件事是在RequestContext的push()中调用app_ctx = self.app.app_context(),app_ctx.push(),将app推入应用上下文栈,
深究下去时可以发现,在RequestContext中有个app属性,它在Flask中的request_context(),也就是在wasi_app()中调用self.request_context(environ)时被赋值,

```
def request_context(self, environ):
    return RequestContext(self, environ)

```

	可以看到每次都传入了self,也就是Flask对象,它在RequestContext中赋值给了self.app,所以在RequestContext push()中每次推入应用上下文的app都是同一个
	有两点需要注意:
	1.在web runtime情况下,请求上下文和应用上下文,同时存在,同时消亡
	2.创建完应用后不会立即生成应用上下文


* 2.RequestContext中放了request和session
AppContext中放了g
current_app就是AppContext对象


* 3.Local()中的__storage__这个字典格式是{thread_id:{'stack':[<RequestContext>,...]}}
所以LocalStack中的top方法返回的结果有两个:一个是RequestContext对象,一个是AppContext对象


> 最后抛两个问题,下次写
1.明明一个线程只能处理一个请求,那么栈里的元素永远是在栈顶,那为什么需要用栈这个结构?用普通变量不行吗.
2._request_ctx_stack和_app_ctx_stack都是线程隔离的,那么为什么要分开?
