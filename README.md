# iOS 面试题

## iOS基础题

* `RunLoop` 的作用是什么？它的内部工作机制了解么？（最好结合线程和内存管理来说）

---

* `frame` 和 `bounds` 的区别？

---

* 什么是 **delegate**?

  **delegate** 是苹果用以实现回调方法的一种途径。通过 **delegate** 可以在不同类之间传递信息。

---

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

---

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

---

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

---

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

---

* **NSOperation** 与 **GCD** 的主要区别？
  1. GCD 的核心是 C 语言写的系统服务，执行和操作简单高效，因此 NSOperation 底层也通过 GCD 实现，换个说法就是 NSOperation 是对 GCD 更高层次的抽象，这是他们之间最本质的区别。因此如果希望自定义任务，建议使用 NSOperation；
  2. 依赖关系，NSOperation 可以设置两个 NSOperation 之间的依赖，第二个任务依赖于第一个任务完成执行，GCD 无法设置依赖关系，不过可以通过`dispatch_barrier_async`来实现这种效果；
  3. KVO(键值对观察)，NSOperation 和容易判断 Operation 当前的状态(是否执行，是否取消)，对此 GCD 无法通过 KVO 进行判断；
  4. 优先级，NSOperation 可以设置自身的优先级，但是优先级高的不一定先执行，GCD 只能设置队列的优先级，无法在执行的 block 设置优先级；
  5. 继承，NSOperation 是一个抽象类，实际开发中常用的两个类是 `NSInvocationOperation` 和 `NSBlockOperation` ，同样我们可以自定义 NSOperation，GCD 执行任务可以自由组装，没有继承那么高的代码复用度；
  6. 效率，直接使用 GCD 效率确实会更高效，NSOperation 会多一点开销，但是通过 NSOperation 可以获得依赖，优先级，继承，键值对观察这些优势，相对于多的那么一点开销确实很划算，鱼和熊掌不可得兼，取舍在于开发者自己； 

---

* 请说明并比较以下关键词：atomatic, nonatomic
  * atomic 修饰的对象会保证 setter 和 getter 的完整性，任何线程对其访问都可以得到一个完整的初始化后的对象。因为要保证操作完成，所以速度慢。它比 nonatomic 安全，但也并不是绝对的线程安全，例如多个线程同时调用 set 和 get 就会导致获得的对象值不一样。绝对的线程安全就要用关键词 synchronized 。
  * nonatomic 修饰的对象不保证 setter 和 getter 的完整性，所以多个线程对它进行访问，它可能会返回未初始化的对象。正因为如此，它比 atomic 快，但也是线程不安全的。

---

* 请说明并比较以下关键词：```__weak```，```__block```

  ```__weak``` 与 ```weak``` 基本相同。前者用于修饰变量（variable），后者用于修饰属性（property）。```__weak``` 主要用于防止 block 中的循环引用。
  ```__block``` 也用于修饰变量。它是引用修饰，所以其修饰的值是动态变化的，即可以被重新赋值的。```__block``` 用于修饰某些 block 内部将要修改的外部变量。
  ```__weak``` 和 ```__block``` 的使用场景几乎与 block 息息相关。而所谓 block，就是Objective-C 对于闭包的实现。闭包就是没有名字的函数，或者理解为指向函数的指针。

---

* 请说明并比较以下关键词：strong, weak, assign, copy
  * strong 表示指向并拥有该对象。其修饰的对象引用计数会增加1。该对象只要引用计数不为 0 则不会被销毁。当然强行将其设为 nil 可以销毁它。
  * weak 表示指向但不拥有该对象。其修饰的对象引用计数不会增加。无需手动设置，该对象会自行在内存中销毁。
  * assign 主要用于修饰基本数据类型，如 NSInteger 和 CGFloat，这些数值主要存在于栈上。
  * weak 一般用来修饰对象，assign 一般用来修饰基本数据类型。原因是assign 修饰的对象被释放后，指针的地址依然存在，造成野指针，在堆上容易造成崩溃。而栈上的内存系统会自动处理，不会造成野指针。
  * copy 与 strong 类似。不同之处是 strong 的复制是多个指针指向同一个地址，而 copy 的复制每次会在内存中拷贝一份对象，指针指向不同地址。copy 一般用在修饰有可变对应类型的不可变对象上，如 NSString , NSArray , NSDictionary 。
  * Objective-C 中，基本数据类型的默认关键字是 atomic , readwrite , assign ；普通属性的默认关键字是 atomic , readwrite , strong 。

---

* 什么是 **ARC** ?

  ARC 全称是 Automatic Reference Counting，是 Objective-C 的内存管理机制。简单地来说，就是代码中自动加入了 retain/release ，原先需要手动添加的用来处理内存管理的引用计数的代码可以自动地由编译器完成了。

  ARC的使用是为了解决对象 retain 和 release 匹配的问题。以前手动管理造成内存泄漏或者重复释放的问题将不复存在。

  以前需要手动的通过 retain 去为对象获取内存，并用 release 释放内存。所以以前的操作称为 MRC (Manual Reference Counting) 。

---

* **dispatch_barrier_async** 的作用是什么？

  在并行队列中，为了保持某些任务的顺序，需要等待一些任务完成后才能继续进行，使用 barrier 来等待之前任务完成，避免数据竞争等问题。

  dispatch_barrier_async 函数会等待追加到 Concurrent Dispatch Queue 并行队列中的操作全部执行完之后，然后再执行 dispatch_barrier_async 函数追加的处理，等 dispatch_barrier_async 追加的处理执行结束之后，Concurrent Dispatch Queue才恢复之前的动作继续执行。

  打个比方：比如你们公司周末跟团旅游，高速休息站上，司机说：大家都去上厕所，速战速决，上完厕所就上高速。超大的公共厕所，大家同时去，程序猿很快就结束了，但程序媛就可能会慢一些，即使你第一个回来，司机也不会出发，司机要等待所有人都回来后，才能出发。 dispatch_barrier_async 函数追加的内容就如同 “上完厕所就上高速”这个动作。

  （注意：使用 dispatch_barrier_async ，该函数只能搭配自定义并行队列 dispatch_queue_t 使用。不能使用： dispatch_get_global_queue ，否则 dispatch_barrier_async 的作用会和 dispatch_async 的作用一模一样。 ）

---

* 内存泄漏和野指针的区别？

---

* block用什么修饰？

---

* NSString 和 NSArray 用 strong 修饰会有什么问题？

---

* iOS的内存管理机制

---

* 什么时候会出现循环引用，```__weak```、```__strong```、```__block``` 分别是什么作用?

---

* 说一下 autoreleasepool ?

---

* autoreleasepool 怎么做到释放对象的?

---

* nil Nil null NSNull 的区别?

---

* OC 中调用 nil 的方法会返回 nil 或 0，但有些时候有特殊情况，不是真正意义上的空/0，举例?

---

* 返回 struct 的方法并没有走send message，走的什么?

---

* 列举修饰符中，内存管理相关关键字及其作用?

Category中使用@property方式添加的属性，实质是什么？支持KVO吗？
isa指针是什么
meta-class是什么
NSDictionary的实现
OC调用C++ 的方式有哪些
runloop的理解，几种模式优先级排序
runloop是怎么实现的
iOS中有哪些方法创建线程
gcd once怎么保证once
GCD串行/并行队列以及sync/async的问题
比较一下线程操作的gcd和nsoperation
OC中提供哪些可扩展的方式
动态库和静态库的区别有哪些
displaylink和timer的区别
如何自己实现timer
不用runtime中的exchange，还有什么方法能达到hook的效果
用runtime交换方法，有些情况下，可能会出问题，怎么解决的
iOS的响应链（详细过程），可以用什么方法影响到响应链
iOS的锁有哪些？介绍一下自旋锁
描述一下OC的编译过程

## iOS实战题



## 网络题



## 数据结构&算法题



##设计模式题

