---
title: flask如何开启多线程详解
date: 2019-01-14 10:11:25
top: 1
tags: 
	- flask
	- python
categories: 
	- python
---
> flask开启多线程
> ==========
> 
> * * *
> 
> 在我之前写的'flask中current_app、g、request、session源码的深究和理解'一文中解释了flask如何支持多线程  
> 主要通过两个类来实现,LocalStack和Local,在Local中有两个属性,\_\_storage\_\_和\_\_ident\_func__,后者用来获取线程id,从而区分不同线程发来的请求
> 
> * * *
> 
> 这次要说的是flask如何开启多线程
> 
> 先从app.run()这个方法看起
> 
```
         def run(self, host=None, port=None, debug=None, **options):
             from werkzeug.serving import run_simple
             if host is None:
                 host = '127.0.0.1'
             if port is None:
                 server_name = self.config['SERVER_NAME']
                 if server_name and ':' in server_name:
                     port = int(server_name.rsplit(':', 1)[1])
                 else:
                     port = 5000
             if debug is not None:
                 self.debug = bool(debug)
             options.setdefault('use_reloader', self.debug)
             options.setdefault('use_debugger', self.debug)
             try:
                 run_simple(host, port, self, **options)  #会进入这个函数
             finally:
                 # reset the first request information if the development server
                 # reset normally.  This makes it possible to restart the server
                 # without reloader and that stuff from an interactive shell.
                 self._got_first_request = False
``` 
> 
> 经过判断和设置后进入run_simple()这个函数,看下源码
> 
```  
     def run_simple(hostname, port, application, use_reloader=False,
                    use_debugger=False, use_evalex=True,
                    extra_files=None, reloader_interval=1,
                    reloader_type='auto', threaded=False,
                    processes=1, request_handler=None, static_files=None,
                    passthrough_errors=False, ssl_context=None):
         """Start a WSGI application. Optional features include a reloader,
         multithreading and fork support.
     
         This function has a command-line interface too::
     
             python -m werkzeug.serving --help
     
         .. versionadded:: 0.5
            `static_files` was added to simplify serving of static files as well
            as `passthrough_errors`.
     
         .. versionadded:: 0.6
            support for SSL was added.
     
         .. versionadded:: 0.8
            Added support for automatically loading a SSL context from certificate
            file and private key.
     
         .. versionadded:: 0.9
            Added command-line interface.
     
         .. versionadded:: 0.10
            Improved the reloader and added support for changing the backend
            through the `reloader_type` parameter.  See :ref:`reloader`
            for more information.
     
         :param hostname: The host for the application.  eg: ``'localhost'``
         :param port: The port for the server.  eg: ``8080``
         :param application: the WSGI application to execute
         :param use_reloader: should the server automatically restart the python
                              process if modules were changed?
         :param use_debugger: should the werkzeug debugging system be used?
         :param use_evalex: should the exception evaluation feature be enabled?
         :param extra_files: a list of files the reloader should watch
                             additionally to the modules.  For example configuration
                             files.
         :param reloader_interval: the interval for the reloader in seconds.
         :param reloader_type: the type of reloader to use.  The default is
                               auto detection.  Valid values are ``'stat'`` and
                               ``'watchdog'``. See :ref:`reloader` for more
                               information.
         :param threaded: should the process handle each request in a separate
                          thread?
         :param processes: if greater than 1 then handle each request in a new process
                           up to this maximum number of concurrent processes.
         :param request_handler: optional parameter that can be used to replace
                                 the default one.  You can use this to replace it
                                 with a different
                                 :class:`~BaseHTTPServer.BaseHTTPRequestHandler`
                                 subclass.
         :param static_files: a list or dict of paths for static files.  This works
                              exactly like :class:`SharedDataMiddleware`, it's actually
                              just wrapping the application in that middleware before
                              serving.
         :param passthrough_errors: set this to `True` to disable the error catching.
                                    This means that the server will die on errors but
                                    it can be useful to hook debuggers in (pdb etc.)
         :param ssl_context: an SSL context for the connection. Either an
                             :class:`ssl.SSLContext`, a tuple in the form
                             ``(cert_file, pkey_file)``, the string ``'adhoc'`` if
                             the server should automatically create one, or ``None``
                             to disable SSL (which is the default).
         """
         if not isinstance(port, int):
             raise TypeError('port must be an integer')
         if use_debugger:
             from werkzeug.debug import DebuggedApplication
             application = DebuggedApplication(application, use_evalex)
         if static_files:
             from werkzeug.wsgi import SharedDataMiddleware
             application = SharedDataMiddleware(application, static_files)
     
         def log_startup(sock):
             display_hostname = hostname not in ('', '*') and hostname or 'localhost'
             if ':' in display_hostname:
                 display_hostname = '[%s]' % display_hostname
             quit_msg = '(Press CTRL+C to quit)'
             port = sock.getsockname()[1]
             _log('info', ' * Running on %s://%s:%d/ %s',
                  ssl_context is None and 'http' or 'https',
                  display_hostname, port, quit_msg)
     
         def inner():
             try:
                 fd = int(os.environ['WERKZEUG_SERVER_FD'])
             except (LookupError, ValueError):
                 fd = None
             srv = make_server(hostname, port, application, threaded,
                               processes, request_handler,
                               passthrough_errors, ssl_context,
                               fd=fd)
             if fd is None:
                 log_startup(srv.socket)
             srv.serve_forever()
     
         if use_reloader:
             # If we're not running already in the subprocess that is the
             # reloader we want to open up a socket early to make sure the
             # port is actually available.
             if os.environ.get('WERKZEUG_RUN_MAIN') != 'true':
                 if port == 0 and not can_open_by_fd:
                     raise ValueError('Cannot bind to a random port with enabled '
                                      'reloader if the Python interpreter does '
                                      'not support socket opening by fd.')
     
                 # Create and destroy a socket so that any exceptions are
                 # raised before we spawn a separate Python interpreter and
                 # lose this ability.
                 address_family = select_ip_version(hostname, port)
                 s = socket.socket(address_family, socket.SOCK_STREAM)
                 s.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
                 s.bind(get_sockaddr(hostname, port, address_family))
                 if hasattr(s, 'set_inheritable'):
                     s.set_inheritable(True)
     
                 # If we can open the socket by file descriptor, then we can just
                 # reuse this one and our socket will survive the restarts.
                 if can_open_by_fd:
                     os.environ['WERKZEUG_SERVER_FD'] = str(s.fileno())
                     s.listen(LISTEN_QUEUE)
                     log_startup(s)
                 else:
                     s.close()
     
             # Do not use relative imports, otherwise "python -m werkzeug.serving"
             # breaks.
             from werkzeug._reloader import run_with_reloader
             run_with_reloader(inner, extra_files, reloader_interval,
                               reloader_type)
         else:
             inner()  #默认会执行
     
 
 还是经过一系列判断后默认会进入inner()函数,这个函数定义在run\_simple()内,属于闭包,inner()中会执行make\_server()这个函数,看下源码:
 
     
     def make_server(host=None, port=None, app=None, threaded=False, processes=1,
                     request_handler=None, passthrough_errors=False,
                     ssl_context=None, fd=None):
         """Create a new server instance that is either threaded, or forks
         or just processes one request after another.
         """
         if threaded and processes > 1:
             raise ValueError("cannot have a multithreaded and "
                              "multi process server.")
         elif threaded:
             return ThreadedWSGIServer(host, port, app, request_handler,
                                       passthrough_errors, ssl_context, fd=fd)
         elif processes > 1:
             return ForkingWSGIServer(host, port, app, processes, request_handler,
                                      passthrough_errors, ssl_context, fd=fd)
         else:
             return BaseWSGIServer(host, port, app, request_handler,
                                   passthrough_errors, ssl_context, fd=fd)
     
 
 看到这也很明白了,想要配置多线程或者多进程,则需要设置threaded或processes这两个参数,而这两个参数是从app.run()中传递过来的:  
 app.run(**options) ---> run\_simple(threaded,processes) ---> make\_server(threaded,processes)  
 默认情况下flask是单线程,单进程的,想要开启只需要在run中传入对应的参数:app.run(threaded=True)即可.  
 从make_server中可知,flask提供了三种server:ThreadedWSGIServer,ForkingWSGIServer,BaseWSGIServer,默认情况下是BaseWSGIServer  
 以线程为例,看下ThreadedWSGIServer这个类:
```
```
     class ThreadedWSGIServer(ThreadingMixIn, BaseWSGIServer):  #继承自ThreadingMixIn, BaseWSGIServer
     
         """A WSGI server that does threading."""
         multithread = True
         daemon_threads = True
     
 
     
     ThreadingMixIn = socketserver.ThreadingMixIn
     
     class ThreadingMixIn:
         """Mix-in class to handle each request in a new thread."""
    
         # Decides how threads will act upon termination of the
         # main process
         daemon_threads = False
     
         def process_request_thread(self, request, client_address):
             """Same as in BaseServer but as a thread.
     
             In addition, exception handling is done here.
     
             """
             try:
                 self.finish_request(request, client_address)
                 self.shutdown_request(request)
             except:
                 self.handle_error(request, client_address)
                 self.shutdown_request(request)
     
         def process_request(self, request, client_address):
             """Start a new thread to process the request."""
             t = threading.Thread(target = self.process_request_thread,
                                  args = (request, client_address))
             t.daemon = self.daemon_threads
             t.start()
```
> 
> process_request就是对每个请求产生一个新的线程来处理  
> 最后写一个非常简单的应用来验证以上说法:
> 
```
     from flask import Flask
     from flask import _request_ctx_stack
     
     app = Flask(__name__)
     
     
     @app.route('/')
     def index():
         print(_request_ctx_stack._local.__ident_func__())
         while True:
             pass
         return '<h1>hello</h1>'
     
     app.run()  #如果需要开启多线程则app.run(threaded=True)
```
> 
> \_request\_ctx\_stack.\_local.\_\_ident\_func__()对应这get\_ident()这个函数,返回当前线程id,为什么要在后面加上while True这句呢,我们看下get\_ident()这个函数的说明:  
> 
> * * *
> 
> Return a non-zero integer that uniquely identifies the current thread amongst other threads that exist simultaneously. This may be used to identify per-thread resources. Even though on some platforms threads identities may appear to be allocated consecutive numbers starting at 1, this behavior should not be relied upon, and the number should be seen purely as a magic cookie. **A thread's identity may be reused for another thread after it exits.**
> 
> * * *
> 
> 关键字我已经加粗了,线程id会在线程结束后重复利用,所以我在路由函数中加了这个死循环来阻塞请求以便于观察到不同的id,这就会产生两种情况:  
> 1.没开启多线程的情况下,一次请求过来,服务器直接阻塞,并且之后的其他请求也都阻塞  
> 2.开启多线程情况下,每次都会打印出不同的线程id  
> 
>     
>     结果:
>     ---------------------
>     第一种情况
>     >>> * Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)
>     >>>139623180527360
>     ---------------------
>     第二种情况
>     >>> * Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)
>     >>>140315469436672
>     >>>140315477829376
>     >>>140315486222080
>     >>>140315316901632
>     >>>140315105163008
>     >>>140315096770304
>     >>>140315088377600
>     
> 
> 结果显而易见  
> 综上所述:flask支持多线程,但默认没开启,其次app.run()只适用于开发环境,生产环境下可以使用uWSGI,Gunicorn等web服务器