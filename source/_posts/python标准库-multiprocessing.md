---
title: python标准库--multiprocessing
date: 2019-01-14 12:57:52
top: 1
tags: 
	- python library
categories: 
	- python
---
基于并行的多进程模块
==========

> 介绍
> --
> 
> *   multiprocessing模块提供了大量进程接口类似于线程模块，并提供了本地和远程的并发，高效的是用子进程代替了线程，避免了GIL全局解释锁的影响，正因为这样，multiprocessing模块允许程序员充分利用多核处理器，可以在unix和windows上使用。
> *   multiprocessing模块也包括了线程模块不具备的接口，一个只要的例子就是Pool对象，它为一个函数传入多个值从而并发执行提供了方便，因为数据的分配在进程中实现。
> 
> ### Process Class
> 
> 进程大量的产出是通过创建Process对象，然后调用start()方法来实现
> 
>     
>     from multiprocessing import Process
>     
>     def f(name):
>         print('hello', name)
>     
>     if __name__ == '__main__':
>         p = Process(target=f, args=('bob',))
>         p.start()
>         p.join()
>     
> 
> ### Contexts and start methods
> 
> > multiprocessing模块提供了三种开启进程的方式 spawn,fork,forkserver  
> > 2.  spawn  
> >     父进程开启一个解释器进程，子进程将只从父进程继承run()时必要的资源，没必要的资源将不会继承，这个开启方法比fork,forkserver慢。  
> >     在unix和windows上都可用，在windows是默认方法
> > 3.  fork  
> >     父进程使用os.fork()，fork一个python解释器，子进程进行时和父进程完全相同，所有资源全部继承，fork一个有多个线程的进程会存在问题。  
> >     只能在unix上可用，并是默认方法
> > 4.  forkserver  
> >     当使用这种方式启动进程，一个服务器进程会被开启，当需要一个新的进程时，父进程连接服务器并请求fork一个新进程，服务器进程是单线程的，所以它用os.fork()是安全的，不会继承不必要的资源。  
> >     在支持将文件描述符通过管道传递给unix的系统中可用
> > 
> > _Changed in version 3.4:_spawn在所有unix平台都可用，在windows中子进程不再继承父进程所有资源。  
> > 通过set\_start\_method()来设置启动方法，在一个程序中只能设置一次  
> > set\_start\_method('spawn')  
> > 或者你可以使用get_context()来选择一个上下文对象，和multiprocessing模块有着同样的接口，允许你在同一个程序中使用多个start方法  
> > ctx = mp.get_context('spawn')  
> > ctx.Process()  
> 
> ### Exchanging objects between processes
> 
> > 多进程支持进程间两种通信方式
> > 
> > 1.  Queue
> > 和queue.Queue类似
> > 
> >     
> >     from multiprocessing import Process, Queue
> >     
> >     def f(q):
> >         q.put([42, None, 'hello'])
> >     
> >     if __name__ == '__main__':
> >         q = Queue()
> >         p = Process(target=f, args=(q,))
> >         p.start()
> >         print(q.get())    # prints "[42, None, 'hello']"
> >         p.join()
> >     
> > 
> > 队列是线程和进程安全的，不用自己用锁  
> > 4.  Pipes
> > Pipe()返回一对通过管道连接的对象，默认是全双工的  
> > 
> >     
> >     from multiprocessing import Process, Pipe
> >     
> >     def f(conn):
> >         conn.send([42, None, 'hello'])
> >         conn.close()
> >     
> >     if __name__ == '__main__':
> >         parent_conn, child_conn = Pipe()
> >         p = Process(target=f, args=(child_conn,))
> >         p.start()
> >         print(parent_conn.recv())   # prints "[42, None, 'hello']"
> >         p.join()
> >     
> > 
> > 每一个连接对象都有send(),recv()方法，如果两个进程或者线程试图同时读取或写入管道的同一端，那么管道中的数据可能会损坏，如果同时读取写入的是管道的不同端则不会有损害的危险。
> 
> ### Synchronization between processes
> 
> > multiprocessing包括了所有类似于线程的同步机制，比如可以使用锁来使得同一时刻只有一个进程输出  
> > 
> >     
> >     from multiprocessing import Process, Lock
> >     
> >     def f(l, i):
> >         l.acquire()
> >         try:
> >             print('hello world', i)
> >         finally:
> >             l.release()
> >     
> >     if __name__ == '__main__':
> >         lock = Lock()
> >     
> >         for num in range(10):
> >             Process(target=f, args=(lock, num)).start()
> >     
> 
> ### Sharing state between processes
> 
> > 在多进程程序中应该避免状态的共享，如果你一定要这么做multiprocessing提供了几种方法  
> > 
> > 1.  Shared memory
> > 数据可以存储在共享内存中，通过Value或者Array映射  
> > 
> >     
> >     from multiprocessing import Process, Value, Array
> >     
> >     def f(n, a):
> >         n.value = 3.1415927
> >         for i in range(len(a)):
> >             a[i] = -a[i]
> >     
> >     if __name__ == '__main__':
> >         num = Value('d', 0.0)
> >         arr = Array('i', range(10))
> >     
> >         p = Process(target=f, args=(num, arr))
> >         p.start()
> >         p.join()
> >     
> >         print(num.value)
> >         print(arr[:])
> >     
> >     output:
> >     3.1415927
> >     [0, -1, -2, -3, -4, -5, -6, -7, -8, -9]
> >     
> > 
> > 'd'表示double双精度浮点型，'i'代表int型  
> > 你可以使用multiprocessing.sharedctypes模块，它支持创建任意类型并分配共享内存。5.  Server process
> > 通过Manger()创建manger对象，支持的类型有list, dict, Namespace, Lock, RLock, Semaphore, BoundedSemaphore, Condition, Event, Barrier, Queue, Value and Array  
> > 
> >     
> >     from multiprocessing import Process, Manager
> >     
> >     def f(d, l):
> >         d[1] = '1'
> >         d['2'] = 2
> >         d[0.25] = None
> >         l.reverse()
> >     
> >     if __name__ == '__main__':
> >         with Manager() as manager:
> >             d = manager.dict()
> >             l = manager.list(range(10))
> >     
> >             p = Process(target=f, args=(d, l))
> >             p.start()
> >             p.join()
> >     
> >             print(d)
> >             print(l)
> >     
> >     output:
> >     {0.25: None, 1: '1', '2': 2}
> >     [9, 8, 7, 6, 5, 4, 3, 2, 1, 0]
> >     
> > 
> > server process 管理器比内存共享对象更加灵活，因为它支持任意数据类型，一个管理器可以共享给在同一个网络中的进程，但是比共享内存运行要慢。
> 
> ### Using a pool of workers
> 
> > Pool类代表了工作进程池，允许将任务卸载到进程中去  
> > 
> >     
> >     from multiprocessing import Pool, TimeoutError
> >     import time
> >     import os
> >     
> >     def f(x):
> >         return x*x
> >     
> >     if __name__ == '__main__':
> >         # start 4 worker processes
> >         with Pool(processes=4) as pool:
> >     
> >             # print "[0, 1, 4,..., 81]"
> >             print(pool.map(f, range(10)))
> >     
> >             # print same numbers in arbitrary order
> >             for i in pool.imap_unordered(f, range(10)):
> >                 print(i)
> >     
> >             # evaluate "f(20)" asynchronously
> >             res = pool.apply_async(f, (20,))      # runs in *only* one process
> >             print(res.get(timeout=1))             # prints "400"
> >     
> >             # evaluate "os.getpid()" asynchronously
> >             res = pool.apply_async(os.getpid, ()) # runs in *only* one process
> >             print(res.get(timeout=1))             # prints the PID of that process
> >     
> >             # launching multiple evaluations asynchronously *may* use more processes
> >             multiple_results = [pool.apply_async(os.getpid, ()) for i in range(4)]
> >             print([res.get(timeout=1) for res in multiple_results])
> >     
> >             # make a single worker sleep for 10 secs
> >             res = pool.apply_async(time.sleep, (10,))
> >             try:
> >                 print(res.get(timeout=1))
> >             except TimeoutError:
> >                 print("We lacked patience and got a multiprocessing.TimeoutError")
> >     
> >             print("For the moment, the pool remains available for more work")
> >     
> >         # exiting the 'with'-block has stopped the pool
> >         print("Now the pool is closed and no longer available")
> >     
> > 
> > 注意，Pool的方法只应该被创建它的进程所使用。
> 
> Reference
> ---------
> 
> ### Process and exceptions
> 
> > class multiprocessing.Process(group=None, target=None, name=None, args=(), kwargs={}, *, daemon=None)
> > 
> > 1.  run()  
> >     调用traget传入的方法
> > 2.  start()  
> >     开启激活进程，每个进程只能开启一次
> > 3.  join(\[timeout\])  
> >     阻塞直到所有进程都执行完毕或者到达设定的timeout时间  
> >     试图在进程启动之前加入进程是错误的。
> > 4.  name  
> >     没有实际意义，多进程间可以使用相同的名字，但不建议这么做
> > 5.  is_alive()  
> >     返回进程是否是存活状态，当进程调用start()后存活
> > 6.  daemon  
> >     是否为守护进程，是布尔值，必须在start()前设置，初始值会设定成父进程对应的值  
> >     当进程退出时，它所有创建的守护进程都将终止  
> >     守护进程不允许创建子进程
> > 7.  pid  
> >     返回进程id
> > 8.  exitcode  
> >     返回退出状态码
> > 9.  authkey  
> >     进程的身份验证密钥(字节字符串)，当multiprocessing被主进程初始化，会被os.urandom()分配一个随机的字符串，当一个进程对象被创建，它将继承父进程的认证密钥，虽然这可以通过设置authkey另一字节串改。
> > 10.  sentinel  
> >     一个系统对象的数字句柄，当进程结束时，它将变成“ready”状态，你可以使用这个值来等待多个事件完成，multiprocessing.connection.wait()，否则使用join()更简单
> > 11.  terminate()  
> >     终止进程，进程的后代进程不会终止——它们将成为孤立的进程  
> >     _Warning:_  
> >     如果在关联进程使用管道或队列时使用此方法，则管道或队列容易损坏并可能被其他进程不可用。类似地，如果进程获取了锁或信号量等，则终止它很容易导致其他进程死锁。__
> > __start(),join(),is_alive(),terminate(),exitcode只能被进程对象(object)调用  
> >   
> > *   exception multiprocessing.ProcessError  
> >     multiprocessing exceptions 基类
> > *   exception multiprocessing.BufferTooShort  
> >     Connection.recv\_bytes\_into()，当提供的缓冲区对象太小而无法读取消息时。
> > *   exception multiprocessing.AuthenticationError  
> >     认证错误
> > *   exception multiprocessing.TimeoutError  
> >     超时错误__