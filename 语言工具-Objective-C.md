Objective-C 是苹果为 iOS 和 Mac 开发量身定制的语言。它随着 iPhone 的出现而大火，直到今天国内外大多数的 App 依然是用 Objective-C 在写。

![](https://upload-images.jianshu.io/upload_images/22877992-79869bcea6d496de.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

Objective-C 一度在 TIOBE 排行榜上位列第 3 名，仅次于 Java 和 C。其市场占有份额也远超其他语言。看名字我们可以知道，它与 C 语言有千丝万缕的联系，事实上也确实如此：Objective-C 是 C 语言的超集，它在 C 语言主体上加上了面向对象的特性。这是为了 App 开发的方便，同时也兼顾了语言的整体性能。

现在的面试中，传统大厂如 BAT 对 Objective-C 的语言进行较多考察，日常开发也是以 Objective-C 为主。 本章将探讨 Objective-C 的基本语言特性，其动态特性将与 Swift 比较中设计。

## Objective-C 面试理论题

### 1.什么是 ARC？

**关键词：#内存管理**

ARC全称是Automatic Reference Counting，是Objective-C和Swift的内存管理机制。它是根据对象的引用计数来判断当前对象的生命周期：当有一个新的指针指向这个对象时，我们将其引用计数加 1，当某个指针不再指向这个对象时，我们将其引用计数减 1，当对象的引用计数变为 0 时，说明这个对象不再被任何指针指向了，这个时候我们就可以将对象销毁，回收内存。

简单地来说，就是代码中自动加入了 retain/release，原先需要手动添加的用来处理内存管理的引用计数的代码可以自动地由编译器完成了。

ARC 的使用是为了解决对象 retain 和 release 匹配的问题。以前手动管理造成内存泄漏或者重复释放的问题将不复存在。

**加分回答:**

以前需要手动的通过 retain 去为对象获取内存，并用 release 释放内存。所以以前的操作称为 MRC (Manual Reference Counting)。

ARC 与 Garbage Collection 的区别在于 Garbage Collection 在 runtime 时管理内存，可以解决 retain cycle，而 ARC 在 compile time 时管理内存。

**类似问题：**

> Objective-C 的内存管理机制是什么？

### 2.什么情况下会出现循环引用？

**关键词：#内存管理**

循环引用是指 2 个或以上对象互相强引用，导致所有对象无法释放的现象。这是内存泄漏的一种情况。举个例子：

```
=== class Father ===
@interface Father: NSObject
@property (strong, nonatomic) Son *son;
@end
=== class Son ===
@interface Son: NSObject
@property (strong, nonatomic) Father *father; 
@end

```

上述代码有两个类，分别为爸爸和儿子。爸爸对儿子强引用，儿子对爸爸强引用。这样释放儿子必须先释放爸爸，要释放爸爸必须先释放儿子。如此一来，两个对象都无法释放。

解决方法是将 Father 中的 Son 对象属性从 strong 改为 weak。

**加分回答:**

内存泄漏可以用 Xcode 中的 Debug Memory Graph 去检查：
![image](https://upload-images.jianshu.io/upload_images/22877992-06bc21a2aae0dd1b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

同时 Xcode 也会在 runtime 中自动汇报内存泄漏的问题：
![image](https://upload-images.jianshu.io/upload_images/22877992-659e793fa8098a72.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 3.请说明并比较以下关键词：strong, weak, assign, copy

**关键词：#内存管理 #引用类型**

*   `strong` 表示指向并拥有该对象。其修饰的对象引用计数会增加 1。该对象只要引用计数不为 0 则不会被销毁。当然强行将其设为 nil 可以销毁它。
*   `weak` 表示指向但不拥有该对象。其修饰的对象引用计数不会增加。无需手动设置，该对象会自行在内存中销毁。
*   `assign` 主要用于修饰基本数据类型，如 NSInteger 和 CGFloat ，这些数值主要存在于栈上。
*   `weak` 一般用来修饰对象，`assign` 一般用来修饰基本数据类型。原因是 assign 修饰的对象被释放后，指针的地址依然存在，造成野指针，在堆上容易造成崩溃。而栈上的内存系统会自动处理，不会造成野指针。
*   `copy` 与 `strong` 类似。不同之处是 `strong` 的复制是多个指针指向同一个地址，而 `copy` 的复制每次会在内存中拷贝一份对象，指针指向不同地址。`copy` 一般用在修饰有可变对应类型的不可变对象上，如 NSString, NSArray, NSDictionary。

**加分回答:**

Objective-C 中，基本数据类型的默认关键字是 atomic, readwrite, assign；普通属性的默认关键字是 atomic, readwrite, strong。

### 4.请说明并比较以下关键词：`atomic`, `nonatomic`

**关键词：#线程**

*   `atomic` 修饰的对象会保证 setter 和 getter 的完整性，任何线程对其访问都可以得到一个完整的初始化后的对象。因为要保证操作完成，所以速度慢。它比 `nonatomic` 安全，但也并不是绝对的线程安全，例如多个线程同时调用 set 和 get 就会导致获得的对象值不一样。绝对的线程安全就要用 `@synchronized`。

*   `nonatomic` 修饰的对象不保证 setter 和 getter 的完整性，所以多个线程对它进行访问，它可能会返回未初始化的对象。正因为如此，它比 `atomic` 快，但也是线程不安全的。

**类似问题：**

> `atomic` 是百分之百线程安全的吗？

### 5.runloop 和线程有什么关系？

**关键词：#线程 #runloop**

runloop 是每一个线程一直运行的一个对象，它主要用来负责响应需要处理的各种事件和消息。每一个线程都有且仅有一个 runloop 与其对应，没有线程，就没有 runloop。

其中所有线程中，只有主线程的 runloop 是默认启动的，main 函数会设置一个 NSRunLoop 对象。其他线程，runloop 默认是没有启动的，我们可以通过 `[NSRunLoop currentRunLoop]` 来获得。

### 6.请说明并比较以下关键词：`__weak`，`__block`

**关键词：#变量修改 #block**

`__weak` 与 `weak` 基本相同。前者用于修饰变量（variable），后者用于修饰属性（property）。`__weak` 主要用于防止 block 中的循环引用。
`__block` 也用于修饰变量。它是引用修饰，所以其修饰的值是动态变化的，即可以被重新赋值的。`__block`用于修饰某些 block 内部将要修改的外部变量。

**加分回答:**

`__weak` 和 `__block` 的使用场景几乎与 block 息息相关。而所谓 block，就是 Objective-C 对于闭包的实现。闭包就是没有名字的函数，或者理解为指向函数的指针。

### 7.什么是 block？它和代理的区别是什么？

**关键词：#回调**

在 iOS 开发中，block 和代理都是回调的方式。Block 是一段封装好的代码，例如下面这个最简单的例子：

```
// 动画结束后的内容就是一个 block
[UIView animateWithDuration:1.0 animations:^{
   NSLog(@”动画已经完成”);
}];

// 声明一个名为 sum 的 block，返回两个整型数值之和
NSInteger (^sumOfNumbers)(NSInteger a, NSInteger b) = ^( NSInteger a, NSInteger b) {
    return a + b;
};

```

而代理的声明和实现一般分开，比如我们熟悉的 UITableViewDelegate 就是代理声明在 UITableView 中，实现在某个 UIViewController 中。

两者的区别首先在于 block 集中代码块而代理分散代码块，所以 block 更适用于轻便、简单的回调，如网络传输。而代理适用于公共接口较多，这样做也更易于解耦代码架构。

另一个区别在于 block 运行成本高。block 出栈需要将使用的数据从栈内存拷贝到堆内存，当然对象的话就是加计数，使用完或者 block 置 nil 后才消除；delegate 只是保存了一个对象指针，直接回调，没有额外消耗。相对 C 的函数指针，只多做了一个查表动作。

注意 `block` 容易造成循环引用，解决方法是用 `__weak` 关键词修饰变量构成弱引用。

## Objective-C 面试实战题

### 8.属性声明代码风格考查

**关键词：#属性声明**

```
@property (nonatomic, strong) NSString *title;
@property (assign, nonatomic) int workID;

```

*   title 不应该用 strong 来修饰，应该用 copy。因为 NSString 是不可变的数据类型，它有对应的 NSMutableString 的数据类型，用 strong 来修饰会有 NSString 被修改的可能性。举个例子：

```
self.title = @”title”;
NSMutableString *mutableTitle = @”mutableTitle”;
self.title = mutableTitle;

```

原来 title 的值是 ”title”，结果后来被改成了 ”mutableTitle”。
有对应可变数据类型的不可变数据类型都应该修饰为 copy。copy 表示该属性不保留新值，而是将其拷贝。这样一来，属性的封装性就可以得到保护，其对应的值是不会无意间被修改的。如果对可变类型如 NSMutableString 用 copy 来修饰，那么当对其进行修改时，程序会崩溃。

*   workID 不应该用 int，而应该用 NSInteger。Int 只表示 32 位的整型数，而 NSInteger 在 32 位机器上与 int 一样，在 64 位机器上则是 64 位的整型数。对于不同类型机器，NSInteger 更加灵活和准确。同理请用 NSUInteger 替代 unsigned，CGFloat 替代 float。
*   属性声明时，最好遵循原子性，读写，内存管理的顺序。这样可读性更高。正确的写法如下：

```
@property (nonatomic, copy) NSString *title;
@property (nonatomic, assign) NSInteger workID;

```

### 9.架构解耦代码考查

**关键词：#属性声明 #架构解耦**

```
typedef enum {
  Normal;
  VIP;
} CustomerType;

@Interface Customer: NSObject 

@property (nonatomic, copy) NSString *name;
@property (nonatomic, strong) UIImage *profileImage;
@property (nonatomic, assign) CustomerType customerType;

@end

```

*   enum 定义的写法不够好。苹果官方推荐使用 NS_ENUM 来定义枚举。同时枚举的每个类型前应加上 enum 的名称，这样方便混编时直接在 Swift 中调用。

*   UIImage 不应该出现在 Customer 中。Customer 明显是一个 Model 类，UIImage 应该归属于 View 部分，无论是 MVC 还是 MVVM 亦或是 VIPER，Model 都应该和 View 划清界限，避免整个架构耦合。下面是正确的代码：

```
typedef NS_ENUM(NSInteger, CustomerType){
  CustomerTypeNormal;
  CustomerTypeVIP;
};

@Interface Customer: NSObject 

@property (nonatomic, copy) NSString *name;
@property (nonatomic, strong) NSData *profileImageData;
@property (nonatomic, assign) CustomerType customerType;

@end

```

### 10.语法考察：请问下面代码打印出什么？

**关键词：#内存管理 #引用类型**

```
NSString *firstStr = @”helloworld”;
NSString *secondStr = @”helloworld”;

if (firstStr == secondStr) {
  NSLog(@”Equal”);
} else {
  NSLog(@”Not Equal”);
}

```

打印出Equal。

*   == 这个符号判断的不是这两个值是否相等，而是这两个指针是否指向同一个对象。如果要判断两个 NSString 是否值相同，平时开发应该用 isEqualToString 这个方法。
*   上面的代码中，两个指针指向不同的对象，尽管它们的值相同。但是 iOS 的编译器优化了内存分配，当两个指针指向两个值一样的 NSString 时，两者指向同一个内存地址。所以这道题会进入 if 的判断，打印出 "Equal" 字符串。

### 11.语法考察：下面代码中有什么 bug？

**关键词：#多线程**

```
- (void)viewDidLoad {
  UILabel *alertLabel = [[UILabel alloc] initWithFrame:CGRectMake(100,100,100,100)];
  alertLabel.text = @"Wait 4 seconds...";
  [self.view addSubview:alertLabel];

  NSOperationQueue *backgroundQueue = [[NSOperationQueue alloc] init];
  [backgroundQueue addOperationWithBlock:^{
    [NSThread sleepUntilDate:[NSDate dateWithTimeIntervalSinceNow:4]];
    alertLabel.text = @"Ready to go!”
  }];
}

```

Bug 在于，在等了 4 秒之后，alertLabel 并不会更新为 Ready to Go。

原因是，所有 UI 的相关操作应该在主线程进行。当我们可以在一个后台线程中等待 4 秒，但是一定要在主线程中更新 alertLabel。

最简单的修正如下：

```
- (void)viewDidLoad {
  UILabel *alertLabel = [[UILabel alloc] initWithFrame:CGRectMake(100,100,100,100)];
  alertLabel.text = @"Wait 4 seconds...";
  [self.view addSubview:alertLabel];

  NSOperationQueue *backgroundQueue = [[NSOperationQueue alloc] init];
  [backgroundQueue addOperationWithBlock:^{
    [NSThread sleepUntilDate:[NSDate dateWithTimeIntervalSinceNow:4]];
      [[NSOperationQueue mainQueue] addOperationWithBlock:^{
    alertLabel.text = @"Ready to go!”
      }];
  }];
}

```

### 12.以 scheduledTimerWithTimeInterval 的方式触发的 timer，在滑动页面上的列表时，timer 会暂停，为什么？该如何解决？

**关键词：#线程 #runloop**

原因在于滑动时当前线程的 runloop 切换了 mode 用于列表滑动，导致 timer 暂停。

runloop 中的 mode 主要用来指定事件在 runloop 中的优先级，有以下几种：

*   Default（NSDefaultRunLoopMode）：默认，一般情况下使用；
*   Connection（NSConnectionReplyMode）：一般系统用来处理 NSConnection 相关事件，开发者一般用不到；
*   Modal（NSModalPanelRunLoopMode）：处理 modal panels 事件；
*   Event Tracking（NSEventTrackingRunLoopMode）：用于处理拖拽和用户交互的模式。
*   Common（NSRunloopCommonModes）：模式合集。默认包括 Default，Modal，Event Tracking 三大模式，可以处理几乎所有事件。

回到题中的情境。滑动列表时，runloop 的 mode 由原来的 Default 模式切换到了 Event Tracking 模式，timer 原来好好的运行在 Default 模式中，被关闭后自然就停止工作了。

解决方法其一是将 timer 加入到 NSRunloopCommonModes 中。其二是将 timer 放到另一个线程中，然后开启另一个线程的 runloop，这样可以保证与主线程互不干扰，而现在主线程正在处理页面滑动。示例代码如下：

```
// 方法1
[[NSRunLoop currentRunLoop] addTimer:timer forMode:NSRunLoopCommonModes];

// 方法2
dispatch_async(dispatch_get_global_queue(0, 0), ^{
    timer = [NSTimer scheduledTimerWithTimeInterval:1 target:self selector:@selector(repeat:) userInfo:nil repeats:true];
    [[NSRunLoop currentRunLoop] run];
});

```
# 推荐👇：

如果你正在跳槽或者正准备跳槽不妨动动小手，添加一下咱们的交流群[**931 542 608**](https://jq.qq.com/?_wv=1027&k=cfVxrMAH)来获取一份详细的大厂面试资料为你的跳槽多添一份保障。

![](https://upload-images.jianshu.io/upload_images/22877992-0bfc037cc50cae7d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

