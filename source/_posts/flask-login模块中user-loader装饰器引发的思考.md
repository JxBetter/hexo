---
title: flask_login模块中user_loader装饰器引发的思考
date: 2019-01-13 17:15:13
top: 1
tags: 
	- flask
	- web
categories: 
	- python
---
# flask_login模块中user_loader装饰器引发的思考

今天看书遇到了flask login模块中的信号机制，看到user_loader这个装饰器时有些疑惑，为什么需要这个装饰器呢，先看一下源码：

```
def user_loader(self, callback):
        '''
        This sets the callback for reloading a user from the session. The
        function you set should take a user ID (a ``unicode``) and return a
        user object, or ``None`` if the user does not exist.
        :param callback: The callback for retrieving a user object.
        :type callback: callable
        '''
        self.user_callback = callback
        return callback
```
看到这不禁疑惑，它的作用只是将被它包装的函数存到self.user_callback这个属性中去，我们先到login_user这个登陆函数中去看看：

```
    def login_user(user, remember=False, duration=None, force=False, fresh=True):
        if not force and not user.is_active:
            return False
 
        user_id = getattr(user, current_app.login_manager.id_attribute)()
        session['user_id'] = user_id
        session['_fresh'] = fresh
        session['_id'] = current_app.login_manager._session_identifier_generator()
 
        if remember:
            session['remember'] = 'set'
            if duration is not None:
                try:
                    # equal to timedelta.total_seconds() but works with Python 2.6
                    session['remember_seconds'] = (duration.microseconds +
                                                   (duration.seconds +
                                                    duration.days * 24 * 3600) *
                                                   10**6) / 10.0**6
                except AttributeError:
                    raise Exception('duration must be a datetime.timedelta, '
                                    'instead got: {0}'.format(duration))
 
        _request_ctx_stack.top.user = user
        user_logged_in.send(current_app._get_current_object(), user=_get_user())
        return True
```
可以看到，login_user这个函数接受user这个主要的参数，getattr(user, current_app.login_manager.id_attribute)()这句是为了调用user中的get_id方法
```

self.id_attribute = ID_ATTRIBUTE
ID_ATTRIBUTE = 'get_id'
```
注意在getattr后面还有个()所以会调用对应的方法，所以user_id中就存放了登陆用户的id号，并写入到session中去，如果设置了remember为True的话，关掉浏览器重新打开后，用户不会退出，函数的最后_request_ctx_stack.top.user = user，将当前user加入到请求上下文的栈顶，就能用current_user获取了。
上面说到self.user_callback已经存了被user_loader装饰的函数，那么在哪里用到了它呢，我在login_manager.py中查找，发现只有一个方法使用到了这个熟悉，这个方法是reload_user()：
```
    def reload_user(self, user=None):
        '''
        This set the ctx.user with the user object loaded by your customized
        user_loader callback function, which should retrieved the user object
        with the user_id got from session.
        Syntax example:
        from flask_login import LoginManager
        @login_manager.user_loader
        def any_valid_func_name(user_id):
            # get your user object using the given user_id,
            # if you use SQLAlchemy, for example:
            user_obj = User.query.get(int(user_id))
            return user_obj
        Reason to let YOU define this self.user_callback:
            Because we won't know how/where you will load you user object.
        '''
        ctx = _request_ctx_stack.top
 
        if user is None:
            user_id = session.get('user_id')
            if user_id is None:
                ctx.user = self.anonymous_user()
            else:
                if self.user_callback is None:
                    raise Exception(
                        "No user_loader has been installed for this "
                        "LoginManager. Refer to"
                        "https://flask-login.readthedocs.io/"
                        "en/latest/#how-it-works for more info.")
                user = self.user_callback(user_id)
                if user is None:
                    ctx.user = self.anonymous_user()
                else:
                    ctx.user = user
        else:
            ctx.user = user
```
它先从请求上下文中取出最新的请求，如果没有传入user，那么会从session中试图取出对应的user_id，这是一种保护机制，不使用cookie，而使用session，user_id在login时会写入session，如果登陆时remember参数传入了True，那么关闭浏览器重新打开后session['user_id']将不会被清除，这时候也就可以获取到了，如果登陆时没有设置remember为True，那么关闭浏览器后user_id会被设为None，则ctx.user = self.anonymous_user()，栈顶的用户为匿名用户，也就需要重新登陆了;取出了user_id，并且self.user_callback不为空，则会调用被user_loader装饰的函数，并传入user_id，在被装饰的函数中我们要根据这个user_id来查找并返回对应的用户实例，如果成功返回，那么当前请求上下文栈顶的用户就设置为返回的用户。
你可能会问，为什么要重载用户呢？因为http协议是无状态的，每次都会发送一个新的请求，请求上下文的栈顶会被新的请求覆盖，对应的user属性也就没了，所以需要通过reload_user重载上一次记录在session中并且未被清除的用户，重载失败则需要重新登陆，这也就是这个装饰器的作用了。
最后我们看下logout_user()这个方法：
```
def logout_user():
    '''
    Logs a user out. (You do not need to pass the actual user.) This will
    also clean up the remember me cookie if it exists.
    '''
 
    user = _get_user()
 
    if 'user_id' in session:
        session.pop('user_id')
 
    if '_fresh' in session:
        session.pop('_fresh')
 
    cookie_name = current_app.config.get('REMEMBER_COOKIE_NAME', COOKIE_NAME)
    if cookie_name in request.cookies:
        session['remember'] = 'clear'
        if 'remember_seconds' in session:
            session.pop('remember_seconds')
 
    user_logged_out.send(current_app._get_current_object(), user=user)
 
    current_app.login_manager.reload_user()
    return True

```
logout主要是清除了session和cookie中的关键参数，比如login时设置的user_id以及remember等，清除后又调用了reload_user()，根据之前的逻辑，当然不可能重载成功，因为user_id已经为None了，执行到ctx.user = self.anonymous_user()就已经结束了，其实reload_user算是这个模块中很关键的一个函数，login_manager这个类也是这个模块的核心所在，以后有时间继续研究。