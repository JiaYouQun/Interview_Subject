# 面试题目总结
###### tips:不能保证作答是正确的，欢迎大家指出错误，这个项目会一直更新。

**一.iOS内存管理是怎么进行的？什么时候会释放一个对象？**

iOS的内存管理，即内存引用计数管理。原理就是，一块内存如果被至少一个地方引用，就不会被释放。基于这个原理，设计了iOS对象的内存管理，用一个数字来记录该内存被引用的数量，当这个数字为0（实际情况并非如此）时，则释放该内存。在ARC下，一个对象什么时候被引用，什么时候引用无效呢。
先说说被引用的情况:<br /> 
1.在一个{}代码块中，这个代码块是引用到了对象。{}代码块执行结束以后，对象的引用消失。<br /> 
2.另一个对象引用该对象作为自己的成员变量。在这里，ARC提供了两种类型来作为一个对象的成员变量，分别是:strong,weak(即assign，略有不同，weak适配对象，当对象被销毁，weak的对象指针会被置为nil，调用nil执行方法，即对空对象发消息，不会对内存进行操作，即不会产生野指针错误。).形象的表示了对一个成员变量的持有态度.即strong引用。weak并不进行引用.<br />
3.特殊对象block。block本质上可以理解为一个对象，block方法体{}内的对象会被它引用。<br /> 
4.全局变量。当一个对象成为全局变量，那么他将在程序生命周期中被持有。<br /> 
5.静态常量。一个静态常量的对象<br /> 

iOS的内存管理是使用引用计数技术，产生一个对象时引用计数加一，这个对象被其他对象引用时，自身的计数也会加一，每当其他的对象释放这个对象时，这个对象的引用计数会减一，直到这个对象的引用计数为0时，这个对象就会被释放.<br /> 　

**二.xcode的clang扫描（静态分析）都可以检查出所有的内存泄漏吗？如果不能，那些情况无法检查出来？**　　  

循环引用，运行时的内存泄漏　　

**三.如何为一个类添加私有方法和属性？**  
如果是系统类，可以使用runtime添加私用方法：OBJC_EXPORT BOOL class_addMethod(Class cls, SEL name, IMP imp, 
const char *types) 返回是否添加成功。cls表示需要添加新方法的类，name表示selector的方法名称，imp是编译器生成的，指向实现方法的指针，也就是说，这个指针指向的方法就是我们要添加方法。  
属性： objc_setAssociatedObject(id object, const void *key, id value, OBJC_ASSOCIATION_RETAIN_NONATOMIC);   
说明：1.object表示给那个对象添加关联，   
     2.const void *key 表示id类型的key值（后续将用这个来取值）属性名，   
     3.id value属性值  
     4.策略，是一个枚举值。  

**四.本地通知和APNS的区别？**  
本地通知是有本地应用触发的，是基于时间的一种通知形式，比如闹钟，待办事项等。  
创建步骤：  
1.创建UILocationfication   
2.设置处理通知的时间   
3.通知的内容   
4.配置通知传递的参数（）（可选）   
5.调用通知，可以使用scheduleLocalNotification：按计划调度一个通知，也可以用presentLocalNotificationNow立即通知  

APNS 是远程推送通知由应用服务提供商发起，步骤：  
1.应用程序首次启动时发送请求给APNS  
2.APNS发回来device token  
3.app把device token发给自身的服务器  
4.自身服务器把带有device token的消息发给APNS  
5.APNS把消息发给iPhone应用程序

**五.iOS中不同app之间是怎么传值的，请至少写出两种，比较优劣**  
1.[UIApplication sharedApplication] openURL:url];  
2.share keychain  
3. 剪切板  
4. runtime

**六.如何捕获UIWebview中的HTTP请求**　　
1.shouldStartLoadWithRequest能够捕获http请求，但是像css，ajax请求都不会走这个代理。　　
2.NSURLProtocol子类的 canonicalRequestForRequest方法可以捕获http请求　　

**七.如何实现系统自带的侧滑返回上一个页面的交互动画**

**八.如何在子线程启动一个定时器并保持运行**　<br /> 　
因为子线程的RunLoop默认是关闭的，所以要定时器在子线程中保持运行，只要创建一个RunLoop（[NSRunLoop currentRunLoop]这个类方法会返回一个创建好的RunLoop）并把timer加入这个RunLoop中并启动即可，代码如下:  
            //创建一个子线程
            dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
            NSTimer *timer2 = [NSTimer timerWithTimeInterval:2 target:self selector:@selector(run) userInfo:nil repeats:YES] ;
            [[NSRunLoop currentRunLoop] addTimer:timer2 forMode:NSDefaultRunLoopMode];
            //子线程中的NSRunLoop默认是不开启的，所以必须开启才能执行run的代码
            [[NSRunLoop currentRunLoop] run];
            //NSLog不会执行，除非run方法的代码执行完毕
            NSLog(@"子线程中的currentRunLoop%@",[NSRunLoop currentRunLoop]);
            });　　

**九.@synchronize（anyobject）{}作用是什么？其中参数anyobject的作用是什么**
@synchronized()是线程锁，作用跟NSLock类似，保证在{}里面执行的操作是安全的，anyobject是需要在安全线程里面执行的参数。
**十.NSObject的performSelect.withObject，可以执行某个对象的方法，当参数多于两个时，怎么解决**

**十一.UITableView的delegate属性是strong，weak还是assign？为什么要这样设计**

**十二.OC与js交互有哪些方式？**  
js调用OC  
- (BOOL)webView:(UIWebView *)webView shouldStartLoadWithRequest:(NSURLRequest *)request navigationType:(UIWebViewNavigationType)navigationType  
截取到加载的webview的数据，主要是request.url。  

OC调用js  
[web stringByEvaluatingJavaScriptFromString:[NSString stringWithFormat:@"contxt_1('%@')",commonString]];  
其中contxt_1这个是js里面定义的函数，commonString是函数的参数。

通过JavaScriptCore调用
js调用OC代码  

//获取js的运行环境
_jsContext=[webView valueForKeyPath:@"documentView.webView.mainFrame.javaScriptContext"];
//当前控制器self执行代理方法
_jsContext[@"native"] = self;
JSExportAs
(calculateForJS  /** handleFactorialCalculateWithNumber 作为js方法的别名 */,
- (void)handleFactorialCalculateWithNumber:(NSNumber *)number
);
在.m中实现handleFactorialCalculateWithNumber即可。

//block的方式
//html调用无参数OC,不管是否有参数都可以写^(){}这样的block，后续通过[JSContext currentArguments]就能获取到传过来的参数。或者直接写 ^(NSArray *myArray){}那么传过来的参数就是myArray.
//在js中定义的方法test1
_jsContext[@"test1"] = ^(NSArray *myArray){
        NSArray * args = [JSContext currentArguments];//传过来的参数
        for (id  obj in args) {
        NSLog(@"html传过来的参数%@",obj);
    }
};

OC调用js
/**
* 获取js里面的方法,这时候返回值是一个方法名
*/
JSValue *picCallback = _jsContext[@"picCallp"];
/**
*  改变P标签，其中的@“Call P”是OC的字符串作为参数
*/
JSValue *newValue = [picCallback callWithArguments:@[@"P标签的内容改变了"]];

NSLog(@"%@",[newValue toNumber]);



**十三.AutoLayout和AutoresizingMask的不同，前者相比后者优越性体现在哪些地方**

**十四.以下代码有什么问题？**  
            -(void)viewDidLoad{  
                [super viewDidLoad];  
                [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(handleNotification:) name:@“notify_test” object:nil];
            }


        -(void)handleNotification:(NSNotification *)notification{  
            [super handleNotification:notification];  
            NSString *name =  [notification.userInfo objectForKey:@“name”];  
            NSInterger *type = [notification.userInfo objectForKey:@“type”];  
           if(type == 1){  
             [self.goodsName addObject:name];  
             [self.collectionView reloadData];  
           }
        }
**十五.EXC_BAD_ACCESS在什么情况下出现，如何捕捉异常？**
EXC_BAD_ACCESS出现一般是因为代码中调用了已经被销毁的对象。这种异常只要在xcode中开启僵尸模式，就可以定位到具体的代码。
**十六.如何监听一个对象中某个属性的变化，至少写出两种方式，并比较优劣**

**十七.写一段代码，在不使用NSJSONSerilation和其他第三方库的情况下解析JSON数据**

**十八.消息响应链是什么？如何实现**

**十九.网络请求当中加密的手段有哪些？如何防止被人篡改和高频率的重复请求？**

**二十.app的登录密码应该如何保存才能做到安全？**

**二十一.有些图片加载比较慢，你是怎么优化性能的**  
利用多线程断点下载，比如图片有5M,在请求头中写请求信息比如1-3M，另一个请求头中写请求信息线程下载3-5M. 两个线程的数据都下载完毕了就合起来。

**二十二.是否可以把比较耗时的操作放在NSNotificationCenter中**  
如果在异步线程中发的通知，那么可以执行比较耗时的操作。
如果是在主线程发的通知，那么就不可以执行比较耗时的操作。

**二十三.NStimer准吗？谈谈你的看法？如果不准该怎样实现一个精确的NSTimer?（来自cocoa社区的题目和答案）**  
1.不准
2.不准的原因如下：
1、NSTimer加在main runloop中，模式是NSDefaultRunLoopMode，main负责所有主线程事件，例如UI界面的操作，复杂的运算，这样在同一个runloop中timer就会产生阻塞。
2、模式的改变。主线程的 RunLoop 里有两个预置的 Mode：kCFRunLoopDefaultMode 和 UITrackingRunLoopMode。
当你创建一个 Timer 并加到 DefaultMode 时，Timer 会得到重复回调，但此时滑动一个ScrollView时，RunLoop 会将 mode 切换为 TrackingRunLoopMode，这时 Timer 就不会被回调，并且也不会影响到滑动操作。所以就会影响到NSTimer不准的情况。
PS:DefaultMode 是 App 平时所处的状态，rackingRunLoopMode 是追踪 ScrollView 滑动时的状态。
方法一：
1、在主线程中进行NSTimer操作，但是将NSTimer实例加到main runloop的特定mode（模式）中。避免被复杂运算操作或者UI界面刷新所干扰。
self.timer = [NSTimer timerWithTimeInterval:1 target:self selector:@selector(showTime) userInfo:nil repeats:YES];
[[NSRunLoop currentRunLoop] addTimer:self.timer forMode:NSRunLoopCommonModes];
2、在子线程中进行NSTimer的操作，再在主线程中修改UI界面显示操作结果；
- (void)timerMethod2 {
NSThread *thread = [[NSThread alloc] initWithTarget:self selector:@selector(newThread) object:nil];
[thread start];
}

- (void)newThread
{
@autoreleasepool
{
[NSTimer scheduledTimerWithTimeInterval:1.0 target:self selector:@selector(showTime) userInfo:nil repeats:YES];
[[NSRunLoop currentRunLoop] run];
}
}

总结：
一开始的时候系统就为我们将主线程的main runloop隐式的启动了。
在创建线程的时候，可以主动获取当前线程的runloop。每个子线程对应一个runloop
方法二：
使用示例
使用mach内核级的函数可以使用mach_absolute_time()获取到CPU的tickcount的计数值，可以通过”mach_timebase_info”函数获取到纳秒级的精确度 。然后使用mach_wait_until(uint64_t deadline)函数，直到指定的时间之后，就可以执行指定任务了。
关于数据结构mach_timebase_info的定义如下：
struct mach_timebase_info {uint32_t numer;uint32_t denom;};
#include
#include
static const uint64_t NANOS_PER_USEC = 1000ULL;
static const uint64_t NANOS_PER_MILLISEC = 1000ULL * NANOS_PER_USEC;
static const uint64_t NANOS_PER_SEC = 1000ULL * NANOS_PER_MILLISEC;
static mach_timebase_info_data_t timebase_info;
static uint64_t nanos_to_abs(uint64_t nanos) {
    return nanos * timebase_info.denom / timebase_info.numer;
}
void example_mach_wait_until(int seconds)
{
    mach_timebase_info(&timebase_info);
    uint64_t time_to_wait = nanos_to_abs(seconds * NANOS_PER_SEC);
    uint64_t now = mach_absolute_time();
    mach_wait_until(now + time_to_wait);
}
方法三：直接使用GCD替代！
// 获得队列
// dispatch_queue_t queue = dispatch_get_global_queue(0, 0);
dispatch_queue_t queue =dispatch_get_main_queue();

// 创建一个定时器(dispatch_source_t本质还是个OC对象) dispatch_source_t类型
self.timer =dispatch_source_create(DISPATCH_SOURCE_TYPE_TIMER,0, 0, queue);

//设置定时器的各种属性（几时开始任务，每隔多长时间执行一次）
// GCD的时间参数，一般是纳秒（1秒 == 10的9次方纳秒）
//何时开始执行第一个任务
// dispatch_time(DISPATCH_TIME_NOW, 1.0 * NSEC_PER_SEC)比当前时间晚3秒
dispatch_time_t start =dispatch_time(DISPATCH_TIME_NOW, (int64_t)(1.0 * NSEC_PER_SEC));
uint64_t interval = (uint64_t)(1.0 *NSEC_PER_SEC);
dispatch_source_set_timer(self.timer, start, interval,0);

// 设置回调
dispatch_source_set_event_handler(self.timer, ^{
NSLog(@"------------%@", [NSThread currentThread]);
//(你想要处理业务,block方式)
count++;
if (count == 4) {
// 取消定时器
dispatch_cancel(self.timer);
self.timer = nil;
}
});

//dispatch_source_set_event_handler_f(self.timer, dispatch_function_t handler)(函数调用方式);
// 启动定时器
dispatch_resume(self.timer);





