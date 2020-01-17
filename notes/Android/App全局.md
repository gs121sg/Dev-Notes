[TOC]



## App

#### Android设计目标

Android平台的一些关键设计目标在其开发过程中逐步演化：

1）为移动设备提供完全开源的平台。Android的开源部分是一个自下而上的操作系统，包含各种应用程序，能够作为完整的产品上市。

2）通过健壮的和稳定的API强有力的支持具有专利的第三方应用。维护一个平台真正开源的同时使其具有专利的第三方应用足够稳定是一个挑战。Android采用了一个混合技术解决方案（具体说明定义明确的SDK并且在公开的API和内部实现之间进行分隔）和策略必要条件（通过CDD）。

3）允许全部第三方应用程序，从而在公平的环境中进行竞争。Android开源代码被设计成对于建立在其上的高级系统功能尽可能保持中立，这些高级系统功能从云服务到库和诸如应用程序商店一类的丰富的服务。

4）提供一种应用程序安全模型，在该模型中用户不必深度依赖第三方应用程序。操作系统必须保护用户免受应用程序不端行为的危害，这不断包括可能导致系统崩溃的有缺陷的应用程序，而且还包括更为微妙的对设备和用户数据的不当使用。用户越不需要信任应用程序，他们就越拥有自由来尝试和安装这些应用程序。

5）支持典型的移动用户界面：使用用户在许多应用中花费少量的时间。移动体验趋向于与应用程序进行短暂的交互：看一眼新收到的电子邮件，接收或者发送一条SMS信息或者IM，进入通讯录拨打一个电话，等等。系统需要对这些情况进行优化，以期获得快速的应用启动和切换时间。Android的目标一般是用200ms冷启动一个基本的应用程序到显示完整的交互式UI。

6）为用户管理应用程序进程，简化围绕应用程序的用户体验，从而使用户在使用完应用程序之后不用想着要将其关闭。移动设备还趋向于在没有交换空间的条件下运行，交换空间能够在当前运行的应用程序需要的RAM多于物理上可用的RAM时，使操作系统衰退得更加优雅。为了处理这两个需求，系统需要采取更积极主动的态度来管理进程，决定何时应该启动和停止它们。

7）鼓励应用程序以丰富和安全的方式互操作和协作。 移动应用程序是以某种方式返回到shell命令的：它们不是像桌面应用程序那样越来越大的单一设计，而是瞄准并聚焦于特定的需求。为帮助支持这一点，操作系统需要为这些应用程序提供新型的设施，使它们共同协作以创建更大的整体。

8）创建一个完全通用的操作系统。移动设备是通用计算的一种新的表现，而不是对传统桌面操作系统的简化。Android的设计应该足够丰富，从而使它至少能够像传统操作系统一样不断成长。

#### Linux拓展

**1. 唤醒锁**

​		移动设备上的电源管理不同于传统的计算机系统，所以为了管理系统如何进入睡眠，Android为Linux添加了一个新的功能，称为唤醒锁（wake lock），也称为悬停阻止器（suspend blocker）。

​		Android上的唤醒锁允许系统进入深度睡眠状态，而不必与一个明确的用户活动（例如关闭屏幕）绑在一起。具有唤醒锁的系统的默认状态是睡眠状态。当设备在运行时，为了保持它不回到睡眠，则需要持有一个唤醒锁。

​		当屏幕打开时，系统总是持有一个唤醒锁，这样就阻止了设备进入睡眠，所有它将保持运行，正如我们所期盼的。然而，在屏幕关闭时，系统本身一般并不持有唤醒锁，所以只有在某些其他实体持有唤醒锁的条件下才能保持系统不进入睡眠。当没有唤醒锁被持有时，系统进入睡眠，并且只能由于硬件中断才能将其从睡眠中唤醒。一旦系统已经进入了睡眠，硬件中断可以将其再次唤醒，如同在传统操作系统中那样。这样的中断源有基于时间的警报、来自蜂窝无线电的事件（例如呼入的呼叫）、到来的网络通信以及按下特定的硬件按钮（例如电源按钮）。针对这些事件的中断处理程序要求对标准Linux做出一个改变：在处理完中断之后，它们需要获得一个初始的唤醒锁从而使系统保持运行。

​		中断处理程序获得的唤醒锁必须持有足够长的时间，以便能够沿着栈向上将控制传递给内核中的驱动程序，由其继续对事件进行处理。然后，内核驱动程序负责获得自己的唤醒锁，在此之后，中断唤醒锁可以安全的得到释放而不存在系统进入睡眠的风险。

**2. 内存不足杀手**

​		Android为内存不足杀手施加了特别的压力。它没有交换空间，所以它处于内存不足情形会更为常见：除非通过放弃从最近使用的存储器映射的干净的RAM页面，否则没有办法缓解内存压力。即便如此，Android还是使用标准的Linux的配置，过度提交内存，也就是说，允许在RAM中分配地址空间而无须保证有可用的RAM对其提供后备。过度提交对于优化内存使用是一个极其重要的工具，这是因为mmap大文件（例如可执行文件）是很常见的，此处只需要将文件中全部数据的一小部分装入RAM。

​		考虑到这样的情形，常备的Linux内存不足杀手工作得并不好，因为它更多的被预定为最后的应急手段，并且很难正确的识别合理的进程来杀死。事实上，正如我们在后面要讨论的，Android广泛地依赖特定期运行内存不足杀手以收割（reap）进程，并且对于选择哪个进程的问题作出好的选择。

​		为解决这一问题，Android为内核引入了自己的内存不足杀手，具有不同的语义和设计目标。Android的内存不足杀手运行得更加积极进取：只要RAM变“低”则运行。低的RAM是由一个可调整的参数标识的，该参数指示在内核中有多少空闲的和缓存的RAM是可接受的。当系统变得低于这个极限时，内存不足杀手便运行以便从别处释放RAM。目标是确保系统绝不会进入坏的分页状态，当前台应用程序竞争RAM时坏的分页状态会对用户体验带来负面影响，因为负面不断换入换出会导致应用程序的执行变得非常缓慢。

​		与试图猜测哪个进程应该被杀死不同，Android的内存不足杀手非常严格的依赖由用户空间提供给它的信息。传统的Linux内存不足杀手具有每个进程的oom_adj参数，通过修改进程的总体坏度得分，该参数用于指导选择最佳的进程将其杀死。Android的内存不足杀手使用这个相同的参数，但是具有严格的顺序：具有较高oom_adj的进程总是在那些具有较低oom_adj的进程之前被杀死。

#### Android体系结构

![](F:\WorkCode\Dev-Notes\notes\Android\pics\Android进程层次结构.png)

​		Android建立在标准Linux内核之上，对内核本身只有少量最重要的拓展，一旦进入用户空间，Android的实现与传统的Linux发行版具有相当大的不同，并且以非常不一样的方式使用你已经了解的Linux功能特性。

​		如同传统的Linux系统一样，Android的第一个用户空间进程是init，它是所有其他进程的根。然而，Android的init启动的守护进程是不同的，这些守护进程更多的聚焦于底层细节（管理文件系统和硬件访问），而不是高层用户设施，例如调度定时任务。Android还有一层额外的进程，它们运行Dalvik的Java语言环境，负责执行系统中所有以Java实现的部分。

​		首先是init进程，它产生了一些底层守护进程。其中一个守护进程是zygote，它是高级Java语言进程的根。Android的init不以传统的方法运行shell，因为典型的Android设备没有本地控制台用于shell访问。作为替代，系统进程adbd监听请求shell访问的远程连接（例如通过USB），按要求为它们创建shell进程。

​		因为Android大部分是用Java语言编写的，所以zygote守护进程以及有它启动的进程是系统的中心。有zygote启动的第一个进程称为system_server，它包含全部核心操作系统服务，其关键部分是电源管理、包管理、窗口管理和活动管理。其它进程在需要的时候由zygote创建，这些进程中有一些是“持久的”进程，它们是基本操作系统的组成部分，例如phone进程中电话栈，它必须保持始终运行。另外的应用程序进程将在系统运行的过程中按需创建和终止。

##### Android框架API设计

​	应用程序通过调用系统提供的库与操作系统进行交互，这些库合起来构成Android框架（Android Framework）。这些库中有一些可以在进程内部执行其工作，但是许多库需要与其它进程执行进程间通信，这通常是在system_server进程中提供服务的。

![](F:\WorkCode\Dev-Notes\notes\Android\pics\API调用交互.png)

​		如上图所示，本例中package manager（包管理器），提供了一个框架API，供应用程序在本地进程中调用，此处API是PackageManager类。在内部PackageManager类必须获得与system_server中相应服务的连接。为达到这一目的，在引导之时system_server在service manager中明确定义名称并发布一个服务，service manager是由init启动的一个守护进程。应用程序中的PackageManager从service manager中检索"package"，并使用相同的名字连接到system_server中PackageManagerService。PackageManagerService实现对所有客户应用程序之间的交互活动进行仲裁，并且维护多个应用程序所需要的状态。

##### Binder IPC

​		Android的系统设计特别围绕进程隔离，不但在应用程序之间，而且在系统本身的不同部分之间隔离进程。这就要求进行大量的进程间通信，从而在不同的进程之间实现协同，这需要做大量的工作并得到正确的结果。Android的Binder进程通信机制是一个丰富的 通用IPC设施，Android系统的大部分就建立在该设施之上。 

​		Binder体系结构分为三个层次，如图所示。(1) 在栈的最底层是一个内核模块，实现了进程交互，并且通过内核的ioctl函数将其展露（ioctl是一个通用的内核调用，用来发送定制的命令给内核 驱动程序和模块）。(2) 在内核模块之上，是一个基本的面向对象的用户空间API，允许应用程序通过IBinder和Binder类创建并且与IPC端点进行交互。(3) 在顶部是一个基于接口的编程模型，应用程序在其中声明它们的IPC接口，并且不再需要关心IPC在底层是如何发生的细节问题。

###### Binder内核模块

###### Binder接口和AIDL

​		Binder IPC最后的部分是经常使用的，基于高级接口的程序设计模型。在这里我们不是和Binder对象和Parcel数据打交道，而是按照接口和方法来思考问题。

​		这一层主要的 部分是一个命令行工具，称为AIDL（Android Interface Definition Language，Android接口定义语言）。该工具是一个接口编译器，它以接口的抽象描述为输入，生成定义接口所必需的源代码，并且实现适当的编组和解组代码，这样的代码是进行远程调用所需要的。

``````
packge com.example
interface IExample{
	void print(String msg);
}
``````

接口描述由AIDL进行编译，生成三个Java语言类，如下图所示：

![](F:\WorkCode\Dev-Notes\notes\Android\pics\Binder接口继承层次结构.png)

1）IExample提供Java语言接口定义；

2）IExample.Stub是实现该接口的基类。它集成自Binder，这意味着它可以是IPC调用的接收者；它继承自IExample，因为这是正在实现的接口。这个类的目的是执行解组：将到来的onTransact调用转换成IExample的适当的方法调用。

3）IExample.Proxy是IPC调用的另一端，负责执行调用的编组。它是IExample的一个具体的实现，实现它的每个方法，将调用转换成适当的Parcel内容，并且通过与之通信的IBinder上的transact调用将其发送出去。

具体调用过程如图所示：

![](F:\WorkCode\Dev-Notes\notes\Android\pics\基于AIDL的Binder IPC的完整路径.png)

1）将方法调用编组成一个Parcel，调用底层BinderProxy上的transact。

2）BinderProxy构造成一个内核事务，并且通过ioctl()调用将其交付给内核。

3）内核将事务传递给意中的进程，将其交付给一个正在其自己的ioctl调用中等待的线程。

4）事务解码回到衣蛾Parcel，并且在适当的本地对象上调用onTransact，在这里本地对象是ExampleImpl（它是IExample.Stub的一个子类）。

5）IExample.Stub将Parcel解码成适当的方法和参数以便进行调用，这里调用的是print。

6）ExampleImpl中print的具体实现最终会执行。 

###### Binder用户空间API

​		大多数用户空间代码不直接与Binder内核模块交互。相反，存在一个用户空间的面向对象的库，它提供了更加简单的API。这些用户空间API的第一层相当于直接的映射到内核概念，采用如下三个类的形式。

1）IBinder是Binder对象的抽象接口。其关键方法是transact，它将一个事务提交给对象。接收事务的实现可能是本地进程中的一个对象，或者是另一个进程的对象。如果它在另一个进程中，则将会通过内核模块交互进行。

2）Binder是一个具体的Binder对象。Binder子类的主要职责是查看它接收到事务数据，其关键方法是onTransact。

3）Parcel是一个容器，用于读和写Binder事务中的数据。它拥有用于读和写类型化数据（整数、字符串、数组）的方法，但是更加重要的是它可以读和写对任何IBinder对象的引用，使用适当的数据结构供内核跨进程理解和传输该引用。

#### Dalvik

​		Dalvik在Android上实现了Java语言环境，它负责运行应用程序以及大部分系统代码。system_service进程中的几乎一切，从包管理（package manager），窗口管理器（window manager），活动管理器（activity manager），都是由Dalvik执行的Java语言代码实现的。

​		编写Java应用程序时，源代码是用Java编写的，然后使用传统的Java工具将其编译成标准的Java字节代码。在此之后，Android引入一个新的步骤：将Java字节码转换成Dalvik的更加紧凑的字节码表示。应用程序的Dalvik字节码版本最后封装成最后的应用程序二进制文件，并且最终安装的设备上。

​		Android的体系结构高度依赖Linux的系统原语，包括内存管理、安全以及跨安全边界的通信。对于核心操作系统概念，Android并不使用Java语言，例如对于进程采用传统的操作系统方法进行隔离。这意味着，每个应用程序运行在自己的Linux进程中，具有自己的Dalvik环境。使用进程进行这样的隔离使得Android能够借力于Linux的功能特性来管理进程，从内存隔离到进程结束时清除与进程相关的所有资源。除了进程以外，Android依赖于Linux的安全特性，而不是使用Java的SecurityManager体系结构。

​		Linux进程和安全的安全大大简化了Dalvik环境，因为它不再需要负责系统稳定性和健壮性这些关键的方面。北非偶然地，它还允许应用程序在它们的实现中自由的使用本机代码，这对于游戏特别重要，因为游戏通常建立在基于C++的引擎之上。

​		zygote负责启动并初始化Dalvik到一个阶段，在此处已经准备开始运行用Java写的系统或应用程序代码。所有基于Dalvik的新进程都是从zygote创建的，使得它们能够在环境已经准备就绪的条件下进行执行。由zygote启动的进程还预装了Android框架的许多部分，这些部分对于系统和应用程序而言是公共的，同时还有需要使用的资源和其它东西。注意，从zygote创建新进程涉及Linux的fork，但是不存在exec调用。新进程是最初zygote进程的复制品，拥有已经建立好的所有预初始化状态，并且做好了运行的准备。调用fork之后，新进程有了自己单独的Dalvik环境，只是它与zygote通过写时复制页面共享预装载和初始化的数据。现在让新的可运行进程准备就绪所剩下的所有的事情是给它一个正确的标识（UID等），完成Dalvik启动线程所需要的初始化工作，以及装载要运行的应用程序或系统代码。

![](F:\WorkCode\Dev-Notes\notes\Android\pics\从zygote创建新的Dalvik进程.png)

#### App进程优先级？

前台进程 一 可视进程 一 服务进程 一 后台进程 一 空进程

```
前台进程
1.

可见进程
服务进程
后台进程
空进程
```

#### 谈谈对Context的理解？

**SDK中对Context说明**

1.它描述的是一个应用程序的信息，即上下文；

2.它是一个抽象类，android提供了该抽象类的具体实现类即ContextIml；

3.通过它可以获取应用程序的资源、类、以及应用级别操作。例如：启动Activity，发送广播，接受Intent信息等等；

**Context相关类的继承关系**

![](F:\WorkCode\Dev-Notes\notes\Android\pics\Context相关类的继承关系.gif)

* Context

  抽象类，提供一组通用的API；

* ContextIml

  Context的实现类，实现Context类的功能。该类大部分功能都是直接调用其属性mPackageInfo去完成。这说明ContextIml是一种轻量级类，而PackageInfo才是真正重量级的类。一个App中所有ContextIml实例，都对应同一个packageInfo对象。

* ContextWrapper

  对Context类的包装，该类的构造函数包含了Context的引用，即ContextIml对象。

* ContextThemeWrapper

  内部包含了Theme相关的接口，即android:theme属性制定的。只有Activity需要主题，Service不需要主题，所以Service直接继承于ContextWrapper类。

**创建Context实例时机**

总Context实例个数 = Service个数 + Activity个数 + 1（Application对应的Context实例）

* 创建Application对象的时机

  每个应用程序在第一次启动时，都会首先创建Application对象，在handleBindApplication()方法中，该函数位于ActivityThread.java类中。

  ```
  
  //创建Application时同时创建的ContextIml实例
  private final void handleBindApplication(AppBindData data){
      ...
  	///创建Application对象
      Application app = data.info.makeApplication(data.restrictedBackupMode, null);
      ...
  }
   
  public Application makeApplication(boolean forceDefaultAppClass, Instrumentation instrumentation) {
  	...
  	try {
          java.lang.ClassLoader cl = getClassLoader();
          ContextImpl appContext = new ContextImpl();    //创建一个ContextImpl对象实例
          appContext.init(this, null, mActivityThread);  //初始化该ContextIml实例的相关属性
          ///新建一个Application对象 
          app = mActivityThread.mInstrumentation.newApplication(
                  cl, appClass, appContext);
         appContext.setOuterContext(app);  //将该Application实例传递给该ContextImpl实例         
      } 
  	...
  }
  ```

* 创建Activity对象的时机

  通过startActivity()或startActivityForResult()请求启动一个Activity时，如果系统检测需要新创建一个Activity对象时，就会回调handleLaunchActivity()方法，该方法继而调用performLaunchActivity()方法，去创建一个Activity实例，并且回调onCreate()、onStart()方法等，函数都位于ActivityThread.java类。

  ```
  //创建一个Activity实例时同时创建ContextIml实例
  private final void handleLaunchActivity(ActivityRecord r, Intent customIntent) {
  	...
  	Activity a = performLaunchActivity(r, customIntent);  //启动一个Activity
  }
  private final Activity performLaunchActivity(ActivityRecord r, Intent customIntent) {
  	...
  	Activity activity = null;
      try {
      	//创建一个Activity对象实例
          java.lang.ClassLoader cl = r.packageInfo.getClassLoader();
          activity = mInstrumentation.newActivity(cl, component.getClassName(), r.intent);
      }
      if (activity != null) {
          ContextImpl appContext = new ContextImpl();      //创建一个Activity实例
          appContext.init(r.packageInfo, r.token, this);   //初始化该ContextIml实例的相关属性
          appContext.setOuterContext(activity);            //将该Activity信息传递给该ContextImpl实例
          ...
      }
      ...    
  }
  ```

* 创建Service对象的时机

  通过startService或者bindService时，如果系统检测到需要新创建一个Service实例，就会回调handleCreateService()方法，完成相关数据操作，函数位于ActivityThread.java类中。

  ```
  //创建一个Service实例时同时创建ContextIml实例
  private final void handleCreateService(CreateServiceData data){
  	...
  	//创建一个Service实例
  	Service service = null;
      try {
          java.lang.ClassLoader cl = packageInfo.getClassLoader();
          service = (Service) cl.loadClass(data.info.name).newInstance();
      } catch (Exception e) {
      }
  	...
  	ContextImpl context = new ContextImpl(); //创建一个ContextImpl对象实例
      context.init(packageInfo, null, this);   //初始化该ContextIml实例的相关属性
      //获得我们之前创建的Application对象信息
      Application app = packageInfo.makeApplication(false, mInstrumentation);
      //将该Service信息传递给该ContextImpl实例
      context.setOuterContext(service);
      ...
  }
  ```

* 法

## 参考

[Android中Context详解 ---- 你所不知道的Context](<https://blog.csdn.net/qinjuning/article/details/7310620>)