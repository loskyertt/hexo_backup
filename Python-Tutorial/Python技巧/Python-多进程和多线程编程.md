---
title: Python 下的多进程和多线程编程
date: 2024-09-16 20:38:40
tags:
    - "python"
excerpt: false
categories: "Python技巧"
---


# 1.多进程

## 1. 1 使用多进程完成任务

用到的包：`multiprocessing`，以下面的代码为例：

运行的函数示例：
```Python
import multiprocessing
import time

def sing():
    for i in range(3):
        print('sing ...')
        time.sleep(0.5)

def dance():
    for i in range(3):
        print('dance ...')
        time.sleep(0.5)
```

- 采用常规方式：
```Python
if __name__ == '__main__':

    start = time.time()
    # 常规方式
    sing()
    dance()
    end = time.time()
    print(f"花费的时间：{end - start}")
```
在运行时可以发现，是先运行`sing`函数，直到`sing`函数完全执行完后，再运行的`dance`函数，输出结果为：
``` txt
sing ...
sing ...
sing ...
dance ...
dance ...
dance ...
花费的时间：3.001246929168701
```
和预想的时间差不多。

- 采用多进程方式：
```Python
if __name__ == '__main__':

    start = time.time()
    # 多进程方式
    process1 = multiprocessing.Process(target=sing)
    process2 = multiprocessing.Process(target=dance)
    process1.start()
    process2.start()
    process1.join()
    process2.join()
    end = time.time()
    print(f"花费的时间：{end - start}")
```
这里的`Process`类用于创建一个新的子进程。在运行时可以发现，`sing`和`dance`函数在同时进行，同时观察到输出结果为：
```txt
sing ...
dance ...
sing ...
dance ...
sing ...
dance ...
花费的时间：1.5049564838409424
```
基本上是在 `1.5s`左右，差不多就是一个函数执行完所需要的时间，多进程执行让两个函数同时开始执行。

## 1.2 join 方法

注意：在多线程编程中也可以用这个方法！

当我们启动多个进程时，主进程默认不会等待这些子进程结束，而是继续执行自己的代码。通过调用子进程的`join()`方法，主进程会暂停执行，直到该子进程运行结束后再继续。

**主要作用如下：**

**确保子进程结束后再继续执行：** 主进程在调用`join()`时会等待相应的子进程执行完毕，再继续执行后续代码。
**同步进程：** 如果不使用`join()`，主进程可能会在子进程执行完之前就结束，从而导致子进程也被强行终止。

比如说这个例子：
```Python
if __name__ == '__main__':

    process1 = multiprocessing.Process(target=sing)
    process2 = multiprocessing.Process(target=dance)
    process1.start()
    process2.start()

    time.sleep(1)
    print("这是一个测试 ... ...")
```

输出结果是：
```txt
sing ...
dance ...
sing ...
dance ...
这是一个测试 ... ...
sing ...
dance ...
```
要输出主进程中的`print("这是一个测试 ... ...")`，需要等`1s`的时间，正好是在`sing`和`dance`函数中的循环体打印两次所需要的时间，可以发现，在到运行的时间时，主进程就直接开始执行了。

现在换成用`join()`方法：
```txt
if __name__ == '__main__':

    process1 = multiprocessing.Process(target=sing)
    process2 = multiprocessing.Process(target=dance)
    process1.start()
    process2.start()
    process1.join()
    process2.join()

    start = time.time()
    time.sleep(1)
    end = time.time()
    print(f"这是一个测试 ... ...，所花费的时间为：{end - start}")
```

输出结果：
```txt
sing ...
dance ...
sing ...
dance ...
sing ...
dance ...
这是一个测试 ... ...，所花费的时间为：1.0003576278686523
```
可以看出，是在子进程中的函数执行完后，再执行主进程中的内容。

## 1.3 进程执行带有参数的任务

运行的函数示例：
```Python
import multiprocessing
import time

def sing(num):
    for i in range(num):
        print('sing ...')
        time.sleep(0.5)

def dance(num):
    for i in range(num):
        print('dance ...')
        time.sleep(0.5)

if __name__ == '__main__':

    process1 = multiprocessing.Process(target=sing, args=(3, ))
    process2 = multiprocessing.Process(target=dance, kwargs={'num': 3})
    process1.start()
    process2.start()
```

一共有两种传参方式：
- **args:** 元组传参，需要注意只有一个参数时，要用逗号结尾，以及传如参数的顺序。
- **kwargs:** 字典传参，注意`key`要对应。

## 1.4 获取子/父进程 PID

```Python
import multiprocessing
import time
import os

def sing(num):
    print('sing 的父进程 pid：', os.getppid())
    print('sing 的子进程 pid：', os.getpid())
    for i in range(num):
        print('sing ...')
        time.sleep(0.5)

def dance(num):
    print('dance 的父进程 pid：', os.getppid())
    print('dance 的子进程 pid：', os.getpid())
    for i in range(num):
        print('dance ...')
        time.sleep(0.5)

if __name__ == '__main__':

    print('程序的父进程 pid（终端/Shell）：', os.getppid())
    print('当前 Python 解释器的 pid：', os.getpid())

    process1 = multiprocessing.Process(target=sing, args=(3, ))
    process2 = multiprocessing.Process(target=dance, kwargs={'num': 3})
    process1.start()
    process2.start()
```
    
输出结果：
```txt
程序的父进程 pid（终端/Shell）： 13583
当前 Python 解释器的 pid： 23492
sing 的父进程 pid： 23492
sing 的子进程 pid： 23493
sing ...
dance 的父进程 pid： 23492
dance 的子进程 pid： 23494
dance ...
sing ...
dance ...
sing ...
dance ...
```

## 1.5 守护进程

运行的函数示例：
```Python
import multiprocessing
import time

def work():

    for i in range(10):
        time.sleep(0.5)
        print('working ...')
```

在默认情况下，主进程会等待子进程完全执行结束才会退出。
比如这样：
```Python
if __name__ == '__main__':

    sub_process = multiprocessing.Process(target=work)
    sub_process.start()

    time.sleep(2)
    print('任务执行结束！')
```
输出结果：
```txt
working ...
working ...
working ...
任务执行结束！
working ...
working ...
working ...
```
主进程执行到`print('任务执行结束！')`这里的时候，已经是执行完了的，但是主进程并没有退出，而是在执行完子进程后才退出的。

现在是设置守护进程的效果：
```Python
if __name__ == '__main__':

    sub_process = multiprocessing.Process(target=work)
    sub_process.daemon = True
    sub_process.start()

    time.sleep(2)
    print('任务执行结束！')
```
让`daemon = True`来设置守护进程，主进程退出后，子进程会被直接销毁，不再执行子进程中的代码。
除了用这种方式，还可以通过像`Process`传入参数来设置：
```Python
if __name__ == '__main__':

    sub_process = multiprocessing.Process(target=work, daemon=True)
    sub_process.start()

    time.sleep(2)
    print('任务执行结束！')
```
两种方式的效果都是一样的。

# 2.多线程

## 2.1 使用多线程完成任务

用到的包：`threading`，以下面的代码为例：

运行的函数示例：
```Python
import threading
import time

def sing():

    for i  in range(3):
        print('singing ...')
        time.sleep(1)

def dance():

    for i in range(3):
        print('dancing ...')
        time.sleep(1)
```

- 采用多线程方式：
```Python
if __name__ == '__main__':

    start = time.time()
    thread1 = threading.Thread(target=sing)
    thread2 = threading.Thread(target=dance)
    thread1.start()
    thread2.start()
    thread1.join()
    thread2.join()
    end = time.time()

    print(f"花费的时间：{end - start}")
```
输出结果：
```txt
singing ...
dancing ...
singing ...
dancing ...
singing ...
dancing ...
花费的时间：3.0005264282226562
```
使用方式基本上和多进程的一样，只是把包名和函数名换了。

## 2.2 join 方法

参考多进程的设置方式。

## 2.3 线程执行带有参数的任务

参考多进程的设置方式，一模一样。

## 2.4 守护线程

参考多进程的设置方式，一模一样。

## 2.5  线程之间的执行顺序

线程之间的执行是无序的，以下面这段代码为例：

```Python
import threading
import time


def work():
    time.sleep(1)
    get_thread = threading.current_thread()
    print(get_thread)


if __name__ == '__main__':

    for i in range(5):
        sub_thread = threading.Thread(target=work)
        sub_thread.start()
```

执行结果是这样的：
```txt
<Thread(Thread-1 (work), started 140407841228480)>
<Thread(Thread-2 (work), started 140407830742720)>
<Thread(Thread-3 (work), started 140407820256960)>
<Thread(Thread-4 (work), started 140407809771200)>
<Thread(Thread-5 (work), started 140407799285440)>
```
从这一个结果上看可能就会得出错误的结论：哪个线程先创建，哪个线程就先执行。但事实上如果我们多执行几次这段代码，就会发现，可能会出现这样的结果：
```txt
<Thread(Thread-2 (work), started 130684293220032)>
<Thread(Thread-4 (work), started 130684272248512)>
<Thread(Thread-3 (work), started 130684282734272)>
<Thread(Thread-1 (work), started 130684303705792)>
<Thread(Thread-5 (work), started 130684261762752)>
```
可以看出，线程并非是按创建顺序来执行的。这是由于线程调度的随机性，打印顺序是不确定的。

# 3.总结-多线程和多进程的区别

## 3.1 工作原理

- **多线程（Threading）：**
  - 多线程是在 **同一个进程的不同线程** 中执行代码。线程是轻量级的执行单元，多个线程共享同一进程的内存空间。
  - 在 Python 中，线程受制于全局解释器锁（GIL），这个锁保证同一时刻只有一个线程执行 Python 字节码。因此，尽管在多线程环境中可以并发执行 I/O 密集型任务，但对于 CPU 密集型任务，多线程并不会真正实现并行执行。

- **多进程（Multiprocessing）：**
  - 多进程则是在 **多个独立的进程** 中运行，每个进程都有自己独立的内存空间和全局解释器锁（GIL）。这意味着不同进程可以真正地在多个 CPU 核心上并行运行，互不干扰。
  - 由于进程间不共享内存，因此进程之间的通信需要通过诸如管道（pipes）、队列（queues）等方式进行。

## 3.2 适用场景

- **多线程：**
  - 适合**I/O 密集型任务**，比如文件读写、网络请求等。在这种情况下，线程在等待 I/O 操作时可以释放 GIL，从而让其他线程运行。
  - **不适合 CPU 密集型任务**，如大量数学计算，因为 GIL 限制了同一时刻只能有一个线程在执行 Python 代码，无法充分利用多核 CPU。

- **多进程：**
  - 适合**CPU 密集型任务**，例如图像处理、复杂的数学计算、机器学习模型训练等。每个进程都拥有独立的 GIL，因此可以在多核 CPU 上并行执行任务。
  - 对于 I/O 密集型任务，也可以使用多进程，不过进程的启动和通信开销较大，性能未必优于多线程。

## 3.3 性能与资源开销

- **多线程：**
  - 线程共享同一进程的内存空间，因此线程之间的数据共享和通信非常快。
  - 线程是轻量级的，创建和管理的开销相对较小。
  - 由于 GIL 的限制，多线程无法充分利用多核 CPU 进行并行计算，在线程数量较多时，频繁的上下文切换也会影响性能。

- **多进程：**
  - 进程之间独立运行，不共享内存，每个进程都有自己的内存空间。这使得进程间的通信开销较大，数据传输需要序列化（例如通过 `pickle`）。
  - 进程的启动和管理开销较大，因为操作系统需要为每个进程分配内存和资源。
  - 多进程可以充分利用多核 CPU，实现真正的并行计算。

## 3.4 资源共享与数据传递

- **多线程：**
  - 线程共享内存空间，因此线程间的变量和数据是共享的。这带来了方便，但也容易引发竞争条件（race conditions）和线程安全问题，必须通过锁（`Lock`）、信号量（`Semaphore`）等机制来保护共享资源。

- **多进程：**
  - 进程不共享内存，因此每个进程有自己独立的数据空间。如果需要在进程间传递数据，必须使用进程间通信（IPC）机制，例如队列（`Queue`）、管道（`Pipe`）等。进程间通信的开销比线程间通信更大。

## 3.5 故障隔离

- **多线程：**
  - 由于线程共享内存空间，如果一个线程发生异常，可能会影响其他线程，甚至导致整个进程崩溃。

- **多进程：**
  - 每个进程是相互独立的，如果一个进程崩溃了，其他进程仍然可以继续运行。因此多进程的故障隔离性更好。