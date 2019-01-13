---
title: flask-login中的login_required装饰器
date: 2019-01-13 11:40:16
top: 1
tags: 
    - flask
    - web
categories: 
    - python
---
# flask-login中的login_required装饰器

```
def login_required(func):

    @wraps(func)

    def decorated_view(*args, **kwargs):

        if request.method in EXEMPT_METHODS:

            return func(*args, **kwargs)

        elif current_app.login_manager._login_disabled:

            return func(*args, **kwargs)

        elif not current_user.is_authenticated:

            return current_app.login_manager.unauthorized()

        return func(*args, **kwargs)

    return decorated_view
```
* 总结
	> EXEMPT_METHODS参数为EXEMPT_METHODS = set(['OPTIONS'])，
current_app.login_manager._login_disabled 默认是False，在login_manager.py第119行，
self._login_disabled = app.config.get('LOGIN_DISABLED', False)，如果没有设置的话，默认是False，
一般会进行到第三句判断，current_user.is_authenticated，判断当前用户是否认证，如果没有认证的话就执行unauthorized()，
unauthorized()会重定向到login_view参数设置的路由函数中去，所以在实例化LoginManager后要设置login_view属性，
当用户没有登陆时，会自动重定向到登陆界面，没有设置会返回401错误， 用户登陆后，这个装饰器就直接返回func(*args, **kwargs)，相当于没有包装一样，装饰器起到包装接口的作用。