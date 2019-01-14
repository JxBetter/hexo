---
title: 从app.route装饰器引发对endpoint的思考
date: 2019-01-14 09:40:13
top: 1
tags: 
	- flask
	- python
categories: 
	- python
---

> 还是先来看看源码
> 
```
         def route(self, rule, **options):
             """A decorator that is used to register a view function for a
             given URL rule.  This does the same thing as :meth:`add_url_rule`
             but is intended for decorator usage::
     
                 @app.route('/')
                 def index():
                     return 'Hello World'
     
             For more information refer to :ref:`url-route-registrations`.
     
             :param rule: the URL rule as string
             :param endpoint: the endpoint for the registered URL rule.  Flask
                              itself assumes the name of the view function as
                              endpoint
             :param options: the options to be forwarded to the underlying
                             :class:`~werkzeug.routing.Rule` object.  A change
                             to Werkzeug is handling of method options.  methods
                             is a list of methods this rule should be limited
                             to (``GET``, ``POST`` etc.).  By default a rule
                             just listens for ``GET`` (and implicitly ``HEAD``).
                             Starting with Flask 0.6, ``OPTIONS`` is implicitly
                             added and handled by the standard request handling.
             """
             def decorator(f):
                 endpoint = options.pop('endpoint', None)
                 self.add_url_rule(rule, endpoint, f, **options)
                 return f
             return decorator
    
```
> route传入了**options这样一个字典，一般我们会传方法methods进去，GET、POST，如果不自己设置endpoint=....的话默认就是No。  
> 然后进入add\_url\_rule函数看一看：  
> 
```
     def add_url_rule(self, rule, endpoint=None, view_func=None, **options):
             if endpoint is None:
                 endpoint = _endpoint_from_view_func(view_func)
             options['endpoint'] = endpoint
             methods = options.pop('methods', None)
     ...
     ...
     ...
             self.url_map.add(rule)
             if view_func is not None:
                 old_func = self.view_functions.get(endpoint)
                 if old_func is not None and old_func != view_func:
                     raise AssertionError('View function mapping is overwriting an '
                                          'existing endpoint function: %s' % endpoint)
                 self.view_functions[endpoint] = view_func
```
> 
> 这里我截取了一些重点的，可以看到如果endpoint为None，会调用\_endpoint\_from\_view\_func函数来给endpoint赋值，  
> 看一下\_endpoint\_from\_view\_func的代码：  
> 
```
     def _endpoint_from_view_func(view_func):
         """Internal helper that returns the default endpoint for a given
         function.  This always is the function name.
         """
         assert view_func is not None, 'expected view func if endpoint ' \
                                       'is not provided.'
         return view_func.__name__
```
> 
> 可以看到将视图函数名赋值给了endpoint，所以如果我们创建视图函数时不在**options中指明endpoint的话，默认就是视图函数名，  
> 后半部分进行了判断，保证了endpoint的唯一，并将view\_func保存在view\_functions这个字典中，并且和endpoint形成映射关系，还将路径加入到当前应用中， 这样做的好处是，当我们用url\_for()从一个endpoint来找到一个URL时，可以顺着这个映射关系来找，而不用写URL， 常见的用法是url\_for(blueprint.endpoint,parm=val...)进行重定向，这样可以获取一个视图函数的路径，传给redirect()，  
> redirect(location,code=302)函数会把接收的参数作为响应body中的Location字段返回给客户端，并且默认是302临时重定向。  
> 
```
     def redirect(location, code=302, Response=None):
     ...
     ...
         #为Location字段赋值，返回给客户端进行重定向
         response.headers['Location'] = location
                 return response
```
> 
> 总结：  
> 
> URL<————>endpoint<————>view  
> *   一个视图函数的endpoint如果不设置那么就是视图函数名。
> *   为URL和view搭起桥梁，使得整个后端框架更加灵活。
> *   url_for(.endpoint)返回的是视图函数对应的URL，URL是对应视图函数装饰器传入的值。