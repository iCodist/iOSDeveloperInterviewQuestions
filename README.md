# iOS 面试题

## iOS基础题

* `RunLoop` 的作用是什么？它的内部工作机制了解么？（最好结合线程和内存管理来说）



* `frame` 和 `bounds` 的区别？



* 什么是 **delegate**?

  **delegate** 是苹果用以实现回调方法的一种途径。通过 **delegate** 可以在不同类之间传递信息。

  

* 谈谈你对 **MVC** 模式的理解？

  * **MVC** 是什么

    MVC 是一种设计思想，一种框架模式，是一种把应用中所有类组织起来的策略，它把你的程序分为三块，分别是：

    M（Model）：实际上考虑的是“什么”问题，你的程序本质上是什么，独立于 UI 工作。是程序中用于处理应用程序逻辑的部分，通常负责存取数据。

    C（Controller）：控制你 Model 如何呈现在屏幕上，当它需要数据的时候就告诉 Model，你帮我获取某某数据；当它需要 UI 展示和更新的时候就告诉 View，你帮我生成一个 UI 显示某某数据，是 Model 和 View 沟通的桥梁。

    V（View）：Controller 的手下，是 Controller 要使用的类，用于构建视图，通常是根据 Model 来创建视图的。

  * **MVC** 通信规则

    * Controller to Model

      可以直接单向通信。Controller 需要将 Model 呈现给用户，因此需要知道模型的一切，还需要有同 Model 完全通信的能力，并且能任意使用 Model 的公共 API。

    * Controller to View

      可以直接单向通信。Controller 通过 View 来布局用户界面。

    * Model to View

      永远不要直接通信。Model 是独立于 UI 的，并不需要和 View 直接通信，View 通过 Controller 获取 Model 数据

    * View to Controller

      View 不能对 Controller 知道的太多，因此要通过间接的方式通信。

  * **MVC** 模式的优点

    1. 低耦合；
    2. 有利于开发分工；
    3. 有利于组件重用；
    4. 可维护性强。



* ``NStimer`` 准吗？谈谈你的看法？如果不准该怎样实现一个精确的 ``NSTimer`` ?

  不准。

  * 原因

    1. ```NSTimer``` 加在 ```main runloop``` 中，模式是 ```NSDefaultRunLoopMode```，```main``` 负责所有主线程事件，例如 UI 界面的操作，复杂的运算，这样在同一个 ```Runloop``` 中 ```Timer``` 就会产生阻塞。

    2. 模式的改变。主线程的 ```RunLoop``` 里有两个预置的 Mode：```kCFRunLoopDefaultMode``` 和 ```UITrackingRunLoopMode```。

       当你创建一个 ```Timer``` 并加到 ```DefaultMode``` 时，```Timer``` 会得到重复回调，但此时滑动一个 ```ScrollView``` 时，```RunLoop``` 会将 mode 切换为 ```TrackingRunLoopMode```，这时 ```Timer``` 就不会被回调，且不会影响到滑动操作。所以 ```NSTimer``` 会出现不准的情况。

  * 实现更精准的 ``NSTimer`` 

    1. 在主线程中进行 ```NSTimer``` 操作，但是将 ```NSTimer``` 实例加到 ```main runloop``` 的特定 mode（模式）中。避免被复杂运算操作或者UI界面刷新所干扰。

       ```objective-c
       self.timer = [NSTimer timerWithTimeInterval:1 target:self selector:@selector(showTime) userInfo:nil repeats:YES];
       
       [[NSRunLoop currentRunLoop] addTimer:self.timer forMode:NSRunLoopCommonModes];
       ```

    2. 在子线程中进行 ```NSTimer``` 的操作，再在主线程中修改 UI 界面显示操作结果。

       ```objective-c
       - (void)timerMethod2 {
       	NSThread *thread = [[NSThread alloc] initWithTarget:self selector:@selector(newThread) object:nil];
       	[thread start];
       }
       
       - (void)newThread {
       	@autoreleasepool {
       		[NSTimer scheduledTimerWithTimeInterval:1.0 target:self selector:@selector(showTime) userInfo:nil repeats:YES];
       		[[NSRunLoop currentRunLoop] run];
       	}
       }
       ```



* 可能出现内存泄漏的几种原因，聊聊你的看法？

  * 原因

    1. 第三方框架不当使用；
    2. block 循环引用；
    3. delegate 循环引用；
    4. NSTimer 循环引用；
    5. 非 OC 对象内存处理；
    6. 地图类处理；
    7. 大次数循环内存暴涨；

  * 追问一：非 OC 对象如何处理？

    非 OC 对象，其需要手动执行释放操作例：```CGImageRelease(ref)```，否则会造成大量的内存泄漏导致程序崩溃。

    其他的对于 ```CoreFoundation``` 框架下的某些对象或变量需要手动释放、C语言代码中的 ```malloc``` 等需要对应 ```free``` 。

  * 追问二：地图类内存若泄漏，如何处理？

    地图是比较耗费 App 内存的，因此在根据文档实现某地图相关功能的同时，需要注意内存的正确释放，大体需要注意的是，在使用完毕时将地图、代理等置空为 nil ；

    注意地图中标注（大头针）的复用，并且在使用完毕时清空标注数组等。



* OC 你了解的锁有哪些？

  * 什么是锁？

    在计算机科学中，锁是一种同步机制，用于存在多线程的环境中实施对资源的访问限制。你可以理解成它用于排除并发的一种策略！

    在iOS中，锁分为递归锁、条件锁、分布式锁、一般锁（根据NSLock类里面的分类进行划分）。

  * 常用的锁

    1. @synchronized 关键字加锁
    2. NSLock 对象锁
    3. NSCondition
    4. NSConditionLock 条件锁
    5. NSRecursiveLock 递归锁
    6. pthread_mutex 互斥锁（C语言）
    7. dispatch_semaphore 信号量实现加锁（GCD）
    8. OSSpinLock
    9. pthread_rwlock
    10. POSIX Conditions
    11. os_unfair_lock

  * 追问一：自旋和互斥的对比？

    * 相同点

      都能保证同一时间只有一个线程访问共享资源。都能保证线程安全。

    * 不同点

      - 互斥锁：如果共享数据已经有其他线程加锁了，线程会进入休眠状态等待锁。一旦被访问的资源被解锁，则等待资源的线程会被唤醒。
      - 自旋锁：如果共享数据已经有其他线程加锁了，线程会以死循环的方式等待锁，一旦被访问的资源被解锁，则等待资源的线程会立即执行。

    自旋锁的效率高于互斥锁。

    使用自旋锁时要注意：

    由于自旋时不释放CPU，因而持有自旋锁的线程应该尽快释放自旋锁，否则等待该自旋锁的线程会一直在哪里自旋，这就会浪费CPU时间。

    持有自旋锁的线程在 ```sleep``` 之前应该释放自旋锁以便其他可以获得该自旋锁。内核编程中，如果持有自旋锁的代码 ```sleep``` 了就可能导致整个系统挂起。

    使用任何锁都需要消耗系统资源（内存资源和CPU时间），这种资源消耗可以分为两类：

    1. 建立锁所需要的资源；
    2. 当线程被阻塞时所需要的资源。

     

* **NSOperation** 与 **GCD** 的主要区别？

  1. GCD 的核心是 C 语言写的系统服务，执行和操作简单高效，因此 NSOperation 底层也通过 GCD 实现，换个说法就是 NSOperation 是对 GCD 更高层次的抽象，这是他们之间最本质的区别。因此如果希望自定义任务，建议使用 NSOperation；
  2. 依赖关系，NSOperation 可以设置两个 NSOperation 之间的依赖，第二个任务依赖于第一个任务完成执行，GCD 无法设置依赖关系，不过可以通过`dispatch_barrier_async`来实现这种效果；
  3. KVO(键值对观察)，NSOperation 和容易判断 Operation 当前的状态(是否执行，是否取消)，对此 GCD 无法通过 KVO 进行判断；
  4. 优先级，NSOperation 可以设置自身的优先级，但是优先级高的不一定先执行，GCD 只能设置队列的优先级，无法在执行的 block 设置优先级；
  5. 继承，NSOperation 是一个抽象类，实际开发中常用的两个类是 `NSInvocationOperation` 和 `NSBlockOperation` ，同样我们可以自定义 NSOperation，GCD 执行任务可以自由组装，没有继承那么高的代码复用度；
  6. 效率，直接使用 GCD 效率确实会更高效，NSOperation 会多一点开销，但是通过 NSOperation 可以获得依赖，优先级，继承，键值对观察这些优势，相对于多的那么一点开销确实很划算，鱼和熊掌不可得兼，取舍在于开发者自己；

   

## iOS实战题



## 网络题



## 数据结构&算法题



##设计模式题

