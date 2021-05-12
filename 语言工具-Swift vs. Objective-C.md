我曾经一度在想苹果为什么要大费周章的出一门新语言，而不是去把同样的精力和时间放在优化 Objective-C 上？后来 Chris Lattner 在他的访谈中说，因为 Objective-C 是一门以 C 语言为基础的语言，所以天生具备 C 的缺点；况且这门语言历经多年，各种弊病也是积重难返。所以，苹果决定，重新开发一门语言，名为 Swift。

![](https://upload-images.jianshu.io/upload_images/22877992-7c18b06a4d66401e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


所以，Swift 从一开始就要和 Objective-C 语言分道扬镳。我们会发现 Swift 注重安全性，Objective-C 注重灵活性；Swift 有函数式编程、面向对象编程、面向协议编程，Objective-C 几乎只有面向对象编程；Swift 更注重值类型的数据结构，而 Objective-C 遵循 C 的老一套，注重指针和索引； Swift 是静态类型语言，Objective-C 却是动态类型语言。

本章将从数据结构、编程思路、语言特性三个角度来回答两种语言的异同。从比较当中我们也更能体会，尽管两者都是为 iOS 开发而定制的语言，Objective-C 和 Swift 依然有着天壤之别。

## 数据结构

### 1.说说 Swift 为什么将 String，Array，Dictionary设计成值类型？

**关键词：#引用类型 #值类型 #多线程 #协议**

要解答这个问题，就要和 Objective-C 中相同的数据结构设计进行比较。Objective-C 中，字符串，数组，字典，皆被设计为引用类型。

*   值类型相比引用类型，最大的优势在于内存使用的高效。值类型在栈上操作，引用类型在堆上操作。栈上的操作仅仅是单个指针的上下移动，而堆上的操作则牵涉到合并、移位、重新链接等。也就是说 Swift 这样设计，大幅减少了堆上的内存分配和回收的次数。同时 copy-on-write 又将值传递和复制的开销降到了最低。

*   String，Array，Dictionary 设计成值类型，也是为了线程安全考虑。通过 Swift 的 let 设置，使得这些数据达到了真正意义上的“不变”，它也从根本上解决了多线程中内存访问和操作顺序的问题。

### 2.用 Swift 将协议（protocol）中的部分方法设计成可选（optional），该怎样实现？

**关键词：#协议 #协议扩展**

`@optional`和 `@required` 是 Objective-C 中特有的关键字。

在 Swift 中，默认所有方法在协议中都是必须实现的。而且，协议里方法不可以直接定义 optional。先给出两种解决方案：

*   在协议和方法前都加上 `@objc` 关键字，然后再在方法前加上 optional 关键字。该方法实际上是把协议转化为 Objective-C 的方式然后进行可选定义。示例如下：

```
@objc protocol SomeProtocol {
  func requiredFunc()
  @objc optional func optionalFunc()
}

```

*   用扩展（extension）来规定可选方法。在 Swift 中，协议扩展（protocol extension）可以定义部分方法的默认实现，这样这些方法在实际调用中就是可选实现的了。示例如下：

```
protocol SomeProtocol {
  func requiredFunc()
  func optionalFunc()
}

extension SomeProtocol {
  func optionalFunc() {
    print(“Dumb Implementation”)
  }
}

Class SomeClass: SomeProtocol {
  func requiredFunc() {
    print(“Only need to implement the required”)
  }
}

```

### 3.下面代码有什么问题？

**关键词：#协议**

```
protocol SomeProtocolDelegate {
  func doSomething()
}

class SomeClass {
  weak var delegate: SomeProtocolDelegate?
}

```

SomeClass 中的 delegate 那行会报错。

Swift 中的协议不仅可以被 class 这样的引用类型来实现，也可能被 struct 或者 enum 这样的值类型实现（这是和 Objective-C 最大的不同）。weak 关键词是用于 ARC 环境下，为引用类型提供引用计数这般的内存管理。它是不能被用来修饰值类型的。

有两种修正方法。

*   在protocol前加上 `@objc`。Objective-C 中协议只能由 class 来实现，这样一来 weak 修饰的对象就与Objective-C 一样，只是 class 类型。修正如下：

```
@objc protocol SomeProtocolDelegate {
  func doSomething()
}

```

*   在 SomeProtocolDelegate 后添加关键词 class 。如此一来声明了该协议只能由 class 来实现。修正如下：

```
protocol SomeProtocolDelegate: class {
  func doSomething()
}

```

## 编程思路

### 4.在 Swift 和 Objective-C 的混编项目中，如何在 Swift 文件中调用 Objective-C 文件中已经定义的方法？如何在 Objective-C 文件中调用 Swift 文件中定义的方法？

**关键词：#头文件 #@objc**

*   在 Swift 中，若要使用 Objective-C 代码，可以在 ProjectName-Bridging-Header.h 里添加 Objective-C 的头文件名称，这样在 Swift 文件中即可调用相应的 Objective-C 代码。一般情况 Xcode 会在 Swift 项目中第一次创建 Objective-C 文件时自动创建 ProjectName-Bridging-Header.h 文件。

*   Objective-C 中若要调用 Swift 代码，可以导入 Swift 生成的头函数 ProjectName-Swift.h 来实现。

**加分回答：**

*   在 Swift 文件中，若要规定固定的方法或属性暴露给 Objective-C 使用，可以在方法或属性前加上 `@objc` 来声明。如果该类是 NSObject 子类，那么 Swift 会在非 private 的方法或属性前自动加上 `@objc` 。

### 5.试比较 Swift 和 Objective-C 中的初始化方法（init）有什么异同？

**关键词：#初始化**

一言以蔽之，在 Swift 中的初始化方法更加严格和准确。

*   Objective-C 中，初始化方法无法保证所有成员变量都完成初始化；编译器对属性设置并无警告，但是实际操作中会出现初始化不完全的问题；初始化方法与普通方法并无实际差别，可以多次调用。

*   在 Swift 中，初始化方法必须保证所有非 optional 的成员变量都完成初始化。同时新增 convenience 和 required 两个修饰初始化方法的关键词。convenience 只是提供一种方便的初始化方法，必须通过调用同一个类中 designated 初始化方法来完成。required 是强制子类重写父类中所修饰的初始化方法。

### 6.试比较 Swift 和 Objective-C 中的协议（Protocol）有什么异同？

**关键词：#协议**

相同点在于，Swift 和 Objective-C 中的 Protocol 都可以被用作代理。Objective-C 中的 Protocol 类似于 Java 中的 Interface，实际开发中主要用于适配器模式（Adapter Pattern，详见第3章第4节设计模式）。

不同点在于，Swift 的 Protocol 还可以对接口进行抽象，例如 Sequence，配合拓展（extension）、泛型、关联类型等可以实现面向协议的编程，从而大大提高整个代码的灵活性。同时 Swift 的 Protocol 还可以用于值类型，如结构体和枚举。

## 语言特性

### 7.谈谈对 Objective-C 和 Swift 动态特性的理解

**关键词：#动态特性 #@runtime #面向协议编程**

runtime 其实就是 Objective-C 的动态机制。runtime 执行的是编译后的代码，这时它可以动态加载对象、添加方法、修改属性、传递信息等等。具体过程是在 Objective-C 中对象调用方法时，如 [self.tableview reload]，发生了两件事。

*   编译阶段，编译器（compiler）会把这句话翻译成 `objc_msgSend(self.tableview,[@selector](https://xiaozhuanlan.com/u/undefined)(reload))`，把消息发送给 self.tableview。

*   运行阶段，接收者 self.tableview 会响应这个消息，期间可能会直接执行、转发消息，也可能会找不到方法崩溃。

所以整个流程是编译器翻译–> 给接收者发送消息 –> 接收者响应消息三个流程。

如 [self.tableview reload] 中，self.tableview 就是接收者，reload 就是消息，所以方法调用的格式在编译器看来是 [receiver message]。

其中接收者如何响应代码，就发生在运行时（runtime）。runtime 执行的是编译后的代码，这时它可以动态加载对象、添加方法、修改属性、传递信息等等，runtime 的运行机制就是 Objective-C 的动态特性。

Swift 目前被公认为是一门静态语言。它的动态特性都是通过桥接 OC 来实现。如果要把动态特性写得更Swift 一点，可以用protocol来处理，比如 OC 中的 reflection 这样写：

```
if ([someImage respondsToSelector:@selector(shake)]) {
  [someImage performSelector:shake];
}

```

Swift中可以这样写：

```
if let shakeableImage = someImage as? Shakeable {
  shakeableImage.shake()
}

```

### 8.以下代码输出什么？

**关键词：#动态特性 #协议 #扩展**

```
protocol Chef {
  func makeFood()
}

extension Chef {
  func makeFood() {
    print("Make Food")
  }
}

struct SeafoodChef: Chef {
  func makeFood() {
    print("Cook Seafood")
  }
}

let chefOne: Chef = SeafoodChef()
let chefTwo: SeafoodChef = SeafoodChef()
chefOne.makeFood()
chefTwo.makeFood()

```

会打印出两行 Cook Seafood。

原因在于 Swift 中，协议中是动态派发，扩展中是静态派发。也就是说，协议中如果有方法声明，那么方法在调用中，会根据对象的实际类型进行调用。

此题中 makeFood() 方法在 Chef 协议中已经声明，而 chefOne 虽然声明为 Chef，但实际实现为 SeaFoodChef 。所以根据实际情况，makeFood() 会调用 SeaFoodChef 中的实现。chefTwo 也是同样的道理。

> 追问：如果 protocol 中没有声明 makeFood() 方法，代码又会输出什么?

会打印出两行，第一行为 Make Food，第二行为 Cook Seafood。

因为协议中没有声明 makeFood() 方法，所以此时只会按照扩展中进行静态派发。也就是说，会根据对象的声明类型进行调用。chefOne 声明为 Chef，所以调用扩展中的实现，chefTwo 声明为 SeafoodChef，所以调用 SeafoodChef 中的实现。

### 9\. message send 如果找不到对象，会如何进行后续处理？

**关键词：#动态特性**

找不到对象分 2 种情况：对象为空（nil）；对象不为空，却找不到对应的方法。

*   对象为空时，Objective-C 在向 nil 发送消息是有效的，在 runtime 中不会产生任何效果。如果信息中的方法返回值是对象，那么给 nil 发送消息返回 nil；如果方法返回值是结构体，那么给 nil 发送消息返回 0 。

*   对象不为空却找不到对应的方法时，程序异常，引发 unrecognized selector。

### 10\. 什么是method swizzling？

**关键词：#动态特性**

每个类都维护一个方法列表，其中方法名与其实现是一一对应的关系，即 SEL（方法名）和 IMP（指向实现的指针）的对应关系。method swizzling 可以在 runtime 时将 SEL 和 IMP 进行更换。比如 SELa 原来对应 IMPa，SELb 原来对应 IMPb，method swizzling 之后，SELa 就可以对应 IMPb，Selb 就对应 IMPa。下面是一个封装好的实现示范：

```
//方法一的 SEL 和 Method SEL
oneSEL = @selector(methodOne:);
Method oneMethod = class_getInstanceMethod(selfClass, oneSEL);

//方法二的 SEL 和 Method
SEL twoSEL = @selector(methodTwo:);
Method twoMethod = class_getInstanceMethod(selfClass, twoSEL); /

//給方法一添加实现，可以避免方法一没有实现
BOOL addSucc = class_addMethod(selfClass, oneSEL, method_getImplementation(twoMethod), method_getTypeEncoding(twoMethod));

if (addSucc) { //添加成功：将方法一的实现替换到方法二
  class_replaceMethod(selfClass, twoSEL, method_getImplementation(oneMethod), method_getTypeEncoding(oneMethod));
}else { //添加失败：方法一已经有实现，直接将方法一和方法二的实现交换
  method_exchangeImplementations(oneMethod, twoMethod);
}

```

**加分回答：**

*   方法交换应该保证唯一性和原子性。唯一性是指应该尽可能在 ＋load 方法中实现，这样可以保证方法一定会调用且不会出现异常。原子性是指使用 dispatch_once 来执行方法交换，这样可以保证只运行一次。

*   不要轻易使用 method swizzling。因为动态交换方法实现并没有编译器的安全保证，可能会在运行时造成奇怪的 bug。

### 11\. Swift 和 Objective-C 的自省（Introspection）有什么不同？

**关键词：#动态特性**

自省在 Objective-C 中就是：判断一个对象是不是属于某个类的操作。它有两种形式：

```
[obj isKindOfClass:[SomeClass class]];
[obj isMemberOfClass:[SomeClass class]];

```

第一句话，isKindOfClass 用来判断 obj 是否为 SomeClass 或其子类的实例对象；第二句话，isMemberOfClass 则对 obj 做出判断，当且仅当 obj 是 SomeClass（非子类）的实例对象时才返回真。这两个方法的使用有个前提，即 obj 必须是 NSObject 或其子类。

Swift 中由于很多 class 并非继承自 NSObject，故而 Swift 用 is 来进行判断。它相当于 isKindOfClass。相比之下优点是 is 不仅可以用于任何 class 类型上，也可以用来判断 enum 和 struct 类型。

**加分回答：**

自省经常与动态类型一起运用。动态类型就是 id 类型，任何类型的对象都可以用 id 来代指。这个时候我们常常用自省来判断对象的实际所属类。示例代码如下：

```
id vehicle = someCarInstance;

if ([vehicle isKindOfClass: [Car class]]) {
  NSLog(“vehicle is a car”);
  if ([vehicle isMemberOfClass: [Tesla class]]) {
    NSLog(“vehicle is a Tesla”);
  }
} else if ([vehicle isKindOfClass: [Truck class]]) {
  NSLog(“vehicle is a a truck”);
}

```

### 12\. 能否通过 Category 给已有的类添加属性（property）？

**关键词：#动态特性**

答案是可以。无论是对于 Objective-C 还是 Swift 而言。

Objective-C 中，正常情况下在 Category 中添加属性会报错，说是找不到 getter 和 setter 方法，这是因为 Category 不会自动生成这两个方法。解决方法是引入运行时头文件，配合关联对象的方法来实现。主要的两个函数是 objc_getAssociatedObject 和 objc_setAssociatedObject 。Swift 中，解决方法与 Objective-C 中相同。只是在写法上更加 Swift 化。

假如我们有个 class 叫 User。由于我们 App 要打开国际化市场了，所以 PM 要求我们 User 能满足有中间名字的老外。于是我们会想在它的 Category 里给它添加 middleName 这个属性。示例的 Objective-C 代码如下：

```
// .h
@interface User (Foreign)
@property (nonatomic, copy) NSString *middleName;
@end

// .m
#import "User + Foreign.h"
#import <objc/runtime.h>

static void *middleNameKey = &middleNameKey;

@implementation User (Foreign) 
- (void)setMiddleName:(NSString *)middleName
{  
  objc_setAssociatedObject(self, &middleNameKey, middleName, OBJC_ASSOCIATION_COPY_NONATOMIC);  
}  
- (NSString *)middleName  
{  
  return objc_getAssociatedObject(self, &middleNameKey);  
}
@end

```

这段代码解释一下是这样的：

1.  在 ".h" 文件中添加属性。此属性是私有属性。
2.  在".m" 文件中引入运行时头文件 `<objc/runtime.h>` ，接着设置关联属性的 Key，最后实现 setter 和 getter。

其中 objc_setAssociatedObject 这个方法的四个参数分别为原对象，关联属性 key，关联属性，关联策略。具体细节可参考苹果官方 API，这里不做展开。用 Swift 实现类似功能是这样的：

```
private var middleNameKey: Void?

extension User {
  var middleName: String? {
    get {
      return objc_getAssociatedObject(self, &middleNameKey) as? String
    }

    set {
      objc_setAssociatedObject(self, &middleNameKey, newValue,. OBJC_ASSOCIATION_COPY_NONATOMIC)
    }
  }
}

```

# 小编推荐阅读
* [2021年-iOS面试进阶资料总结（备战年后）](https://www.jianshu.com/p/22b6744fef42)
* [2020年面试：整理出一份高级iOS面试题](https://www.jianshu.com/p/939765be93a7)
* [2020 — iOS 面试败北感悟](https://www.jianshu.com/p/1380bd22832e)
* [2020年6月最新iOS面试题总结（答案篇）](https://www.jianshu.com/p/407f473df36a)
