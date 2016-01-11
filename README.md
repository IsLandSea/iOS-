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
    
    4:用 dispatch_async 处理后台任务
    在重载 UIViewController 的 viewDidLoad 时容易加入太多杂乱的工作（too much       clutter），这通常会引起视图控制器出现前更长的等待。如果可能，最好是卸下一些工作放到后台，如果它们不是绝对必须要运行在加载时间里。
这听起来像是 dispatch_async 能做的事情！
- (void)viewDidLoad
{   
    [super viewDidLoad];
    NSAssert(_image, @"Image not set; required to use view controller");
    self.photoImageView.image = _image;

    //Resize if neccessary to ensure it's not pixelated
    if (_image.size.height <= self.photoImageView.bounds.size.height &&
        _image.size.width <= self.photoImageView.bounds.size.width) {
        [self.photoImageView setContentMode:UIViewContentModeCenter];
    }
     dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_HIGH, 0), ^{ // 1
        UIImage *overlayImage = [self faceOverlayImageFromImage:_image];
        dispatch_async(dispatch_get_main_queue(), ^{ // 2
            [self fadeInNewImage:overlayImage]; // 3
        });
    });
}
    下面来说明上面的新代码所做的事：你首先将工作从主线程移到全局线程。
    因为这是一个 dispatch_async() ，Block会被异步地提交，意味着调用线程地执行将会继续。
    这就使得 viewDidLoad更早地在主线程完成，让加载过程感觉起来更加快速。同时，一个人脸检测过程会启动并将在稍后完成。
    在这里，人脸检测过程完成，并生成了一个新的图像。既然你要使用此新图像更新你的 UIImageView ，
    那么你就添加一个新的 Block到主线程。记住——你必须总是在主线程访问 UIKit 的类。
   最后，你用 fadeInNewImage: 更新 UI ，它执行一个淡入过程切换到新的曲棍球眼睛图像。
编译并运行你的应用；选择一个图像然后你会注意到视图控制器加载明显变快，曲棍球眼睛稍微在之后就加上了。这给应用带来了不错的效果，和之前的显示差别巨大。dispatch_async 添加一个 Block 到队列就立即返回了。任务会在之后由 GCD 决定执行。当你需要在后台执行一个基于网络或 CPU 紧张的任务时就使用 dispatch_async ，这样就不会阻塞当前线程。

 5:dispatch_after 延后工作
 
 6:dispatch_once() 以线程安全的方式执行且仅执行其代码块一次。试图访问临界区（即传递给 dispatch_once 的代码）的不同的线程会在临界区已有一个线程的情况下被阻塞，直到临界区完成为止。

- (void)viewDidLoad
{
  [super viewDidLoad];


  dispatch_sync(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_HIGH, 0), ^{

      NSLog(@"First Log");

  });
  
  
  NSLog(@"Second Log");
}

 7:dispatch_sync Block 被添加到一个全局队列中，将在稍后执行。进程将在主线程挂起直到该 Block 完成。同时，全局队列并发处理任务；要记得
 
 Block 在全局队列中将按照 FIFO 顺序出列，但可以并发执行。全局队列处理 dispatch_sync Block 加入之前已经出现在队列中的任务。终于，轮到
 
 dispatch_sync Block 。这个 Block 完成，因此主线程上的任务可以恢复。viewDidLoad 方法完成，主队列继续处理其他任务
 
 dispatch_sync 添加任务到一个队列并等待直到任务完成。dispatch_async做类似的事情，但不同之处是它不会等待任务的完成，而是立即继续“调用线程”的其它任务。
 
 - (void)viewDidLoad
{
  [super viewDidLoad];


  dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_HIGH, 0), ^{


      NSLog(@"First Log");

  });
  

  NSLog(@"Second Log");
  
}

viewDidLoad 在主线程执行。主线程目前在 viewDidLoad 内，正要到达 dispatch_async 。dispatch_async Block

被添加到一个全局队列中，将在稍后执行。viewDidLoad 在添加 

dispatch_async到全局队列后继续进行，主线程把注意力转向剩下的任务。同时，全局队列并发地处理它未完成地任务。记住 Block 在全局队列中将按照 

FIFO 顺序出列，但可以并发执行。添加到 dispatch_async 的代码块开始执行。dispatch_async Block 完成，两个 NSLog 

语句将它们的输出放在控制台上。

   
   
    总结：除了上面这些，你还通过利用 dispatch_after 来延迟显示提示信息，以及利用 dispatch_async 将 CPU 密集型任务从 ViewController 
    
    的初始化过程中剥离出来异步执行，达到了增强应用的用户体验的目的。
    
