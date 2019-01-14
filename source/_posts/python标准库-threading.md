---
title: python标准库--threading
date: 2019-01-14 12:45:43
top: 1
tags: 
	- python library
categories: 
	- python
---
> 1.  threading.active_count()  
>     返回当前存活的线程数
> 2.  threading.current_thread()  
>     返回当前线程对象，如果当前没有线程，则返回一个功能有限的虚拟线程对象dummy thread object
> 3.  threading.get_ident()  
>     返回一个非零线程标识符，没有实际意义，作为类似魔数cookie来使用，比如索引一个包含特定线程数据的字典，  
>     当一个线程退出后，其标识符可以循环使用
> 4.  threading.enumerate() 返回当前所有存活的线程列表，包括后台线程，dummy线程和主线程，不包括终止的和未开启的线程
> 5.  threading.main_thread() 返回主线程，通常是从解释器启动的
> 6.  threading.settrace(func)  
>     为所有线程设置一个跟踪函数，这个函数会传递给sys.settrace()，会正对于每个线程，要在线程run()之前使用
> 7.  threading.setprofile(func)  
>     与settrace类似，为每个线程设置一个配置文件函数
> 8.  threading.stack_size(\[size\])  
>     返回创建新线程时堆栈的大小，可以指定size用于随后创建的线程堆栈大小，size要么是0要么是大于32kb的正整数，所以size最小为32*1024bytes  
>     32kb目前支持的最小的堆栈大小值，为解释器本身保证足够的堆栈空间
> 9.  threading.TIMEOUT_MAX  
>     返回获取锁或者等待函数的最大超时时间，超过会返回OverflowError

Class
-----

> 1.  threading.local  
>     为线程保存特有的数据，  
>     threading.local()  
>     mydata = threading.local()  
>     mydata.x = 1
> 2.  Thread Objects  
>     有两种方法来制定线程的活动，第一种是在创建线程时指定traget，它默认会被run()方法调用，第二种方法是创建一个子类，并重写\_\_init\_\_和run()方法  
>     一旦一个线程对象被创建，它必须通过调用线程的start()方法启动，并且会调用run()方法  
>     一旦线程的活动启动，线程就被认为是“alive”状态，当run()方法终止时或者接受了一个未处理的异常，线程就会停止alive状态  
>     is_alive()方法可以检测线程是否存活
>     *   class threading.Thread(group=None, target=None, name=None, args=(), kwargs={}, *, daemon=None)  
>         线程的构造函数:参数group是保留的参数，为将来实现线程组的功能，默认为None；参数target是将在函数run()调用的函数，默认为None；参数name是线程名称；参数args是目标调用的函数参数；参数kwargs是目标调用的字典参数，如果显示设置了daemon，则会将线程设置为后台  
>         如果子类重写了构造函数，它必须确保调用基类的构造函数(Thread.\_\_init\_\_())在做其他事情之前
>     *   start()  
>         启动线程，每个线程对象最多必须调用一次，否则会产生RuntimeError
>     *   run()  
>         线程运行的主函数，可以继承线程类重载这个函数，有参数的话需要传参
>     *   join(timeout=None)  
>         等待线程终止，这会阻塞线程直到终止、产生异常或者超时  
>         timeout单位为秒，join总是返回None，如果想知道一个线程是否发生超时，你要在join()后调用is_alive()，如果还存活则超时  
>         如果timeout没有被设置，则阻塞到所有线程终止  
>         一个线程可以多次调用join()，join会引发RuntimeError，如果试图加入当前线程，并会造成死锁，join会引发相同的错误，如果加入一个未启动的线程
>     *   name  
>         仅用于识别的字符串，它没有实际意义，多个线程可以被赋予相同的名称，初始名称由构造函数设置
>     *   getName()  
>         setName(name)  
>         老的API，可以直接用name属性代替
>     *   ident  
>         获取标识
>     *   is_alive()  
>         线程是否存活，存活的条件是线程已经调用了run()方法，并且还未终止
>     *   daemon  
>         isDaemon()  
>         setDaemon()  
>         线程是否为守护线程或者叫后台线程，可直接调用属性，老的API
> 3.  Lock Objects
> 原生锁有两种状态，locked或者unlocked，当它是locked时不由特定的线程拥有  
> 有两种方法acquire() 和 release()，当状态是unlocked时，调用acquire()获得锁，状态有unlocked变成locked，当状态是locked时，acquire()会阻塞  
> 直到release()，release()只能在locked状态下使用，如果尝试去release()一个unlocked的锁，会引发RunTimeError  
> 当多个线程阻塞等待release()时，只有一个线程会获得锁
> 
> *   class threading.Lock
> *   acquire(blocking=True, timeout=-1)  
>     获取线程锁。当参数blocking设置为True时，直到获取到锁成功才返回；当参数blocking设置为False时，不进行阻塞，调用之后立即返回。如果当设置锁成功，返回True，设置锁不成功返回False；参数timeout是设置阻塞超时间，以秒为单位的小数
> *   release()  
>     释放锁，可以从任何线程来释放之前锁住的锁；然后设置锁为无锁状态，允许之前等待之中的一个线程再次获取锁；如果调用此函数之前没有执行任何获取锁的动作，就立即抛出异常RuntimeError  
>     不会返回任何值
> 
> 8.  RLock Objects
> 可重用锁，可以由同一线程多次获取，在内部，它使用了“拥有线程”和“递归级别”的概念，以及原始锁所使用的锁定/解锁状态，在锁定状态下，某些线程拥有锁；在未锁定状态下，没有线程拥有它  
> 为了加锁，线程调用acquire()，一旦线程获得了锁就返回，为了解锁，线程调用release()  
> acquire()/release()总是成对出现，release()在最后使用
> 
> *   acquire()同上
> *   release()同上
> 
> 12.  Condition Objects
> 条件对象，有wait() notify()等方法，主要用来判断线程条件，使其做出正确行为，wait()会释放锁，并且会阻塞线程，直到这个线程被notify()唤醒  
> notify()并不会释放锁，需要用release()释放 消费者-生产者模型很适合这个例子，简述一下：  
> 消费者wait()等待某样特定物品，生产者生产许多物品，当生产者生产出特定物品后notify()唤醒消费者达到目的
> 
> *   class threading.Condition(lock=None)
> *   acquire(*args)
> *   release()
> *   wait(timeout=None)
> *   wait_for(predicate, timeout=None)
> *   notify(n=1)
> *   notify_all()
> 
> 16.  Semaphore Objects
> 信号量有一个内部计数器，初始化时会赋予信号量值，这个值不能小于零，当信号量为零时，调用acquire()会阻塞线程，直到release()后信号量不为零 class threading.Semaphore(value=1)，信号量默认值为1，这时和锁差不多  
> 
> *   acquire(blocking=True, timeout=None)  
>     获取一个信号量
> *   release()  
>     释放一个信号量