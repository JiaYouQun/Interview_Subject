# 面试题目总结
###### tips:不能保证作答是正确的，欢迎大家指出错误，这个项目会一直更新。

**一.iOS内存管理是怎么进行的？什么时候会释放一个对象？**

内存管理是个老生常谈的问题。
iOS的内存管理，即内存应用计数器。
原理就是，一块内存如果被至少一个地方引用，就不会被释放。
基于这个原理，设计了iOS对象的内存管理。
在ARC下，一个对象什么时候被引用，什么时候应用无效呢。
先说说被引用的情况。</p>
1.在一个{}代码块中，这个代码块是引用到了对象。{}代码块执行结束以后，对象的引用消失。</p>
2.另一个对象引用该对象作为自己的成员变量。</p>
在这里，ARC提供了两种类型来作为一个对象的成员变量，分别是：</p>
Strong,Weak(即assign).形象的表示了对一个成员变量的持有态度.</p>
即Strong引用。weak并不进行引用.</p>　
3.特殊对象block。block本质上可以理解为一个对象，block方法体{}内的对象会被它引用。</p>
4.全局变量。当一个对象成为全局变量，那么他将在程序生命周期中被持有。</p>
5.静态常量。一个静态常量的对象</p>

iOS的内存管理是使用引用计数技术，产生一个对象时引用计数加一，这个对象被其他对象引用时，自身的计数也会加一，每当其他的对象释放这个对象时，这个对象的引用计数会减一，直到这个对象的引用计数为0时，这个对象就会被释放.</p>　

**二.xcode的clang扫描（静态分析）都可以检查出所有的内存泄漏吗？如果不能，那些情况无法检查出来？**　　
循环引用，运行时的内存泄漏　　

**三.如何为一个类添加私有方法和属性？**

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

**八.如何在子线程启动一个定时器并保持运行**　　
- dispatch_queue_t queue = dispatch_queue_create("kk", DISPATCH_QUEUE_SERIAL);
-     // 串行队列中执行异步任务
-     dispatch_async(queue, ^{
-         // 在子线程中使用定时器
-         /*************************************************************/
-         // 第一种方式
　　       // 此种方式创建的timer已经添加至runloop中
-         [NSTimer scheduledTimerWithTimeInterval:2.0 target:self selector:@selector(doSomething) userInfo:nil repeats:NO];

-         // 在线程中使用定时器，如果不启动run loop，timer的事件是不会响应的，而子线程中runloop默认没有启动
-         // 让线程执行一个周期性的任务，如果不启动run loop， 线程跑完就可能被系统释放了
-         [[NSRunLoop currentRunLoop] run];// 如果没有这句，doSomething将不会执行！！！
-         // 第二种方式
-         NSTimer *timer = [NSTimer timerWithTimeInterval:2.0 target:self selector:@selector(doAnything) userInfo:nil repeats:NO];
-
-         // 将定时器添加到runloop中
-         [[NSRunLoop currentRunLoop] addTimer:timer forMode:NSDefaultRunLoopMode];
-
-         // 在线程中使用定时器，如果不启动run loop，timer的事件是不会响应的，而子线程中runloop默认没有启动
-         // 让线程执行一个周期性的任务，如果不启动run loop， 线程跑完就可能被系统释放了
-         [[NSRunLoop currentRunLoop] run];// 如果没有这句，doAnything将不会执行！！！
-
-         /*************************************************************/
-
-         NSLog(@"子线程结束");
-
-     });

**九.@synchronize（anyobject）{}作用是什么？其中参数anyobject的作用是什么**

**十.NSObject的performSelect.withObject，可以执行某个对象的方法，当参数多于两个时，怎么解决**

**十一.UITableView的delegate属性是strong，weak还是assign？为什么要这样设计**

**十二.OC与js交互有哪些方式？**

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

**十六.如何监听一个对象中某个属性的变化，至少写出两种方式，并比较优劣**

**十七.写一段代码，在不使用NSJSONSerilation和其他第三方库的情况下解析JSON数据**

**十八.消息响应链是什么？如何实现**

**十九.网络请求当中加密的手段有哪些？如何防止被人篡改和高频率的重复请求？**

**二十.app的登录密码应该如何保存才能做到安全？**
