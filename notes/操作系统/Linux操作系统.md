[TOC]

### 内存映射

参考链接[1]

### 进程空间划分

一个进程空间分为用户空间和内核空间（Kernel），即把进程内用户与内核隔离开来。两者区别是：

* 进程间，用户空间的数据不可共享，所以用户空间 = 不可共享空间
* 进程间，内核空间的数据可共享，所以内核空间 = 可共享空间

所有进程共用1个内核空间，进程内用户空间与内核空间进行交互，需通过系统调用，主要函数：

* copy_from_user（）：将用户空间的数据拷贝到内核空间
* copy_to_user（）：将内核空间的数据拷贝到用户空间

![](F:\WorkCode\Dev-Notes\notes\操作系统\pics\进程间交互.png)

### 进程隔离 & 跨进程通信（IPC）

**进程隔离**

为了保证安全性 & 独立性，一个进程不能直接操作或者访问另一个进程，即进程之间是相互独立、隔离的。

**跨进程通信**

即进程间需进行数据交互、通信。

* 传统的跨进程通信

  ![](F:\WorkCode\Dev-Notes\notes\操作系统\pics\传统的跨进程通信.png)

* 使用了内存映射的跨进程通信

![](F:\WorkCode\Dev-Notes\notes\操作系统\pics\使用了内存映射的跨进程通信.png)

* Binder跨进程通信

![](F:\WorkCode\Dev-Notes\notes\操作系统\pics\Binder跨进程通信.png)

## 参考内容

1.[操作系统：图文详解 内存映射](<https://www.jianshu.com/p/719fc4758813>)

2.[图文详解 Android Binder跨进程通信的原理](<https://www.jianshu.com/p/4ee3fd07da14>)