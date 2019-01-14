---
title: python标准库--queue
date: 2019-01-14 12:52:41
top: 1
tags: 
	- python library
categories: 
	- python
---

> 队列模块实现了多消费者、多生产者队列，当数据在多线程之间需要安全交互时非常有用，队列模块实现了所有必要的锁，它依赖于python可靠的线程支持  
> 模块实现了三种类型的队列，区别在于检索出条目的顺序。  
> FIFO队列，先进先出；LIFO队列，和栈类似，后进的先出；优先队列，将队列中的元素排序呢，最小的先出。  
> 队列模块实现的类和异常：*   class queue.Queue(maxsize=0)
> *   class queue.LifoQueue(maxsize=0)
> *   class queue.PriorityQueue(maxsize=0)  
>     注意：当maxsize设置为0或者负数时，队列的大小变为无限，如果队列大小有限，当队列满时，插入会被阻塞
> *   exception queue.Empty  
>     队列为空时，如果使用了未阻塞的get()或者是get_nowait()则会引发异常
> *   exception queue.Full  
>     队列满时，如果使用了未阻塞的put()或者是put_nowait()也会引发异常
> 
> Queue Objects
> -------------
> 
> 1.  Queue.qsize()  
>     返回队列的大致大小，qsize()大于0并不能保证get()不会被阻塞，同样qsize()小于maxsize也不会保证put()不会被阻塞
> 2.  Queue.empty()  
>     返回队列是否为空，为空不能保证put()不会被阻塞，不为空不能保证get()不会被阻塞
> 3.  Queue.full()  
>     返回队列是否已满，如果已经满了不能保证get()不会被阻塞，如果未满也不能保证put()不会被阻塞
> 4.  Queue.put(item, block=True, timeout=None)  
>     将数据放入队列，如果block=True,timeout=None，也就是默认情况下，如果队列已满则会阻塞直到队列有余量，  
>     如果timeout为正数，到达timeout指定时间，队列还是满的话会引发full异常  
>     如果block设置为false，队列未满则放入数据成功，队列已满会立即引发异常，这种情况下timeout会被忽略
> 5.  Queue.put_nowait(item)  
>     等价于put(item, False)
> 6.  Queue.get(block=True, timeout=None)  
>     从队列中移走并返回一个数据，如果block=True,timeout=None，也就是默认情况下，如果队列为空则会阻塞知道队列中有元素可以被取出，  
>     如果timeout为正数，到达timeout指定时间，队列还是空的话会引发empty异常，  
>     如果block设置为false，队列不为空则返回数据，为空则立即引发异常，这种情况下timeout会被忽略
> 7.  Queue.get_nowait()  
>     等价于get(False)
> 8.  Queue.task_done()  
>     申明之前的队列任务已经完成，被用在消费线程中，用get()来获取数据，获取完毕后通过 task_done()告诉队列，任务完毕  
>     如果有一个join()当前依旧阻塞，那么它知道队列中所有元素都被取出后才会恢复(即所有队列中元素都被task_done()调用申明已经处理完毕)  
>     如果task_done()调用次数大于队列中的元素数量，则会引发ValueError异常
> 9.  Queue.join()  
>     阻塞，知道所有队列中的元素被取出和调用task_done申明处理完毕  
>     当有元素加入到队列中时，未完成的任务数会增加，当有元素从队列中取出并且调用task_done()时，未完成任务数会减少，当未完成任务数为0时，join()变成不阻塞状态