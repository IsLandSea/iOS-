# iOS-
iOS多线程编程有哪些方式
iOS的三种多线程技术特点:

1.NSThread:

    1> 使用NSThread对象建立一个线程非常方便;

    2> 但是!要使用NSThread管理多个线程非常困难,不推荐使用;

    3> 技巧!使用[NSThread currentThread]跟踪任务所在线程,适用于这三种技术.
    NSThread:
   
    优点：NSThread 比其他两个轻量级
    
    缺点：需要自己管理线程的生命周期，线程同步。线程同步对数据的加锁会有一定的系统开销
    
    2:Cocoa operation 
    优点：不需要关心线程管理，数据同步的事情，可以把精力放在自己需要执行的操作上。

    2.NSOperation/NSOperationQueue:

    1> 是使用GCD实现的一套Objective-C的API;

    2> 是面向对象的多线程技术;

    3> 提供了一些在GCD中不容易实现的特性,如:限制最大并发数量,操作之间的依赖关系.

3.GCD---Grand Central Dispatch:
     Grand Central Dispatch (GCD)是Apple开发的一个多核编程的解决方法
     
    1> 是基于C语言的底层API;

    2> 用Block定义任务,使用起来非常灵活便捷;

    3> 提供了更多的控制能力以及操作队列中所不能使用的底层函数.
    
    队列:

    dispatch_queue_t

    串行队列: 队列中的任务只会顺序执行;

    并行队列: 队列中的任务通常会并发执行.
    
    操作:

    dispatch_async 异步操作,会并发执行,无法确定任务的执行顺序;

    dispatch_sync 同步操作,会依次顺序执行,能够决定任务的执行顺序.
    
    队列不是线程,也不表示对应的CPU.队列就是负责调度的.多线程技术的目的,就是为了在一个CPU上实现快速切换!
     
     在串行队列中:

    同步操作不会新建线程,操作顺序执行(没用!);

    异步操作会新建线程,操作顺序执行(非常有用!) (应用场景:既不影响主线程,又需要顺序执行的操作).
    
    在并行队列中:

    同步操作不会新建线程,操作顺序执行;

    异步操作会新建多个线程,操作无序执行(有用,容易出错),队列前如果有其他任务,会等待前面的任务完成之后再执行.应用场景:既不影响主线程,又不需要顺序执行的操作.
    
    全局队列:

    全局队列是系统的,直接拿过来(GET)用就可以,与并行对立类似,但调试时,无法确认操作所在队列.
    
    主队列:

    每一个应用程序都对应唯一一个主队列,直接GET即可,在多线程开发中,使用主队列更新UI;
    
    GCD公开有5个不同的队列:运行在主线程中的主队列,3个不同优先级的后台队列以及一个优先级更低的后台队列(用于I/O).

    自定义队列:串行和并行队列.自定义队列非常强大,建议在开发中使用.
    
    //循环引用
    
    如果self对象持有操作对象的引用,同时操作对象当中又直接访问了self时,才会造成循环引用.

    单纯在操作对象中使用self不会造成循环引用.
    
    block在copy时都会对block内部用到的对象进行强引用(ARC)或者retainCount增1(非ARC)。
    
    在ARC与非ARC环境下对block使用不当都会引起循环引用问题，一般表现为，
    
    某个类将block作为自己的属性变量，然后该类在block的方法体里面又使用了该类本身，
    
    简单说就是self.someBlock = ^(Type var){[self dosomething];或者self.otherVar = XXX;或者_otherVar = ...};
    
    block的这种循环引用会被编译器捕捉到并及时提醒.
    
    3:声明delegate时请用assign(MRC)或者weak(ARC)，千万别手贱玩一下retain或者strong，毕竟这基本逃不掉循环引用了！
    
    
    
    
    
