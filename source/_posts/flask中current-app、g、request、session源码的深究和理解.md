---
title: flask中current_app、g、request、session源码的深究和理解
date: 2019-01-14 10:05:01
top: 1
tags: 
	- flask
	- python
categories: 
	- python
---

> 本文是我在学习flask中对上下文和几个类似全局变量的思考和研究，也有我自己的理解在内。  
>   
> 为了研究flask中的current_app、g、request、session，我找到定义在global.py的源码：
```
     `# context locals
     _request_ctx_stack = LocalStack()
     _app_ctx_stack = LocalStack()
     current_app = LocalProxy(_find_app)
     request = LocalProxy(partial(_lookup_req_object, 'request'))
     session = LocalProxy(partial(_lookup_req_object, 'session'))
     g = LocalProxy(partial(_lookup_app_object, 'g'))` 
```
> 
> 可以看到主要由\_lookup\_req\_object、\_lookup\_app\_object、\_find\_app等组成，我先来分析request和session  
> 其实request和session原理上是一样的，所以将其归为一类，称为请求上下文。  
>   
> 我们从最里面看起，partial(\_lookup\_req\_object, 'request')，最外层是一个偏函数，不过这不是重点，它主要是将'request'传给\_lookup\_req\_object，没有其他含义， 顺着\_lookup\_req_object找到它的源码
```
     `def _lookup_req_object(name):
     top = _request_ctx_stack.top
     if top is None:
         raise RuntimeError(_request_ctx_err_msg)
     return getattr(top, name)` 
``` 
> 
> 从最后的return可以看到，这个函数的主要功能是从top中取出键值为'request'的内容，top是一个字典，top从\_request\_ctx\_stack.top中来，在上面的源码中 \_request\_ctx\_stack = LocalStack()，从名字来看LocalStack应该是一个栈类，应该有pop,push,top方法，我继续找到源码：
```
     ``def __init__(self):
         self._local = Local()
     ...
     ...
     def push(self, obj):
         """Pushes a new item to the stack"""
         rv = getattr(self._local, 'stack', None)
         if rv is None:
             self._local.stack = rv = []
         rv.append(obj)
         return rv
 
     def pop(self):
         """Removes the topmost item from the stack, will return the
         old value or `None` if the stack was already empty.
         """
         stack = getattr(self._local, 'stack', None)
         if stack is None:
             return None
         elif len(stack) == 1:
             release_local(self._local)
             return stack[-1]
         else:
             return stack.pop()
 
     @property
     def top(self):
         """The topmost item on the stack.  If the stack is empty,
         `None` is returned.
         """
         try:
             return self._local.stack[-1]
         except (AttributeError, IndexError):
             return None`` 
```  
> 
> 可以看到LocalStack()这个类有一个属性self._local = Local()，对应另一个类，继续看源码：
```
     `
     def __init__(self):
         object.__setattr__(self, '__storage__', {})
         object.__setattr__(self, '__ident_func__', get_ident)
     ...
     ...
     def __getattr__(self, name):
         try:
             return self.__storage__[self.__ident_func__()][name]
         except KeyError:
             raise AttributeError(name)
 
     def __setattr__(self, name, value):
         ident = self.__ident_func__()
         storage = self.__storage__
         try:
             storage[ident][name] = value
         except KeyError:
             storage[ident] = {name: value}
     `         
```
> 
> 我截取了几个重要的函数，LocalStack()中的push，用到了Local()中的\_\_setattr\_\_();pop用到了\_\_getattr\_\_()，看到push和pop都是对'stack'这个键值进行查询和赋值，我们转到Local()这个类中，这个类有两个实例属性，\_\_storage\_\_和\_\_ident\_func__，前者是一个字典，后者是一个函数，我们看一下这个get_ident函数，查看源码：
```
     `def get_ident(): # real signature unknown; restored from __doc__
         """
         get_ident() -> integer
 
         Return a non-zero integer that uniquely identifies the current thread
         amongst other threads that exist simultaneously.
         This may be used to identify per-thread resources.
         Even though on some platforms threads identities may appear to be
         allocated consecutive numbers starting at 1, this behavior should not
         be relied upon, and the number should be seen purely as a magic cookie.
         A thread's identity may be reused for another thread after it exits.
         """
         return 0` 
```  
> 
> 显然这个函数不是python写的，[因为它来自_thread.py](http://因为它来自_thread.py)，是一个底层库，从名字可以猜到和线程有关，根据描述，它返回一个非零整数，代表了当前线程id，我们再看看\_\_setattr\_\_这个方法，它其实是一个字典嵌套列表再嵌套字典的数据，\_\_storage\_\_是一个字典，它里面的键值被赋值为当前线程id，这个键值对应的值是另一个字典:{'stack':\['request':r\_val,'session':s\_val\]}，这样和前面联系起来就很好理解了，Local()中的\_\_storage\_\_存储的格式为{thread\_id:{'stack':\['request':r\_val,'session':s_val\]}}，LocalStack()中的top方法，返回了'stack'中最后一个加入的元素，也就是最新的元素，我自己理解为服务器接受的最新的请求，在框架外看起来request和session是一个全局变量，其实内部已经由进程id将其分隔开了，即使同时有多个请求过来，进程间的数据也不会混乱。  
>   
> 同理current\_app和g也一样，唯一不同的是，current\_app、g和request、session是两个不同的实例，注意前面 \_request\_ctx\_stack = LocalStack()、\_app\_ctx\_stack = LocalStack()，所以'stack'中存的数据也不一样，current_app和g称为应用上下文，两者还是有区别的。  
> LocalProxy 则是一个典型的代理模式实现，它在构造时接受一个 callable 的参数（比如一个函数），这个参数被调用后的返回值本身应该是一个 Thread Local 对象。对一个 LocalProxy 对象的所有操作，包括属性访问、方法调用（当然方法调用就是属性访问）甚至是二元操作，都会转发到那个 callable 参数返回的 Thread Local 对象上。  
> LocalProxy 的一个使用场景是 LocalStack 的 \_\_call\_\_ 方法。比如 my\_local\_stack 是一个 LocalStack 实例，那么 my\_local\_stack() 能返回一个 LocalProxy 对象，这个对象始终指向 my\_local\_stack 的栈顶元素。如果栈顶元素不存在，访问这个 LocalProxy 的时候会抛出 RuntimeError。  
> 需要注意的是，如果需要离线编程，尤其在写测试代码时，需要将应用上下文push到栈中去，不然current\_app会指向空的\_app\_ctx\_stack栈顶，自然也就无法工作了。  
> 我们可以通过current\_app的值来判断是否进入应用上下文中，可以用app.app\_context().push()来进入应用上下文。