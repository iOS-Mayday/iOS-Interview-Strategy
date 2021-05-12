本章节主要针对 iOS 的主流开发语言 Objective-C 和 Swift 进行分析和对比，同时也整理了 Xcode 编辑器的使用技巧和经验。

正所谓工欲善其事必先利其器，说的就是考察的是开发者对自己手头工具和语言特性的掌握。

![](https://upload-images.jianshu.io/upload_images/22877992-f41f6443b65c112b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


在 iOS 开发中，语言的选择是最初的一步。现在苹果主推的编程语言是 Swift。

Swift 自 2014 年发布以来，已经历经 4 个版本的迭代。在 TIOBE 编程语言排行榜上的目前位列 12 位，超过 Ruby 并远远甩开其上代语言 Objective-C。从性能上来说，它的速度是 Objective-C 的 2.6 倍，Python 的 8.4 倍。更重要的是，Swift 是一门开源的语言，它的质量和进步接受着整个业界的建议、监督、关注。无论从哪个角度讲，Swift 都将取代 Objective-C，成为 iOS 开发的主流语言。

所以在面试中，我们会看到关于 Swift 的问题越来越多。对于这门语言的熟悉度，成为判断一个工程师水准的重要标准之一。下面的题目将会从实战和理论两个角度来探讨 Swift 的特性。

## 理论题

### 1\. 类（class）和结构体（struct）有什么区别？

**关键词：#引用类型 #值类型**

在 Swift 中，类是引用类型，结构体是值类型。值类型在传递和赋值时将进行复制，而引用类型则只会使用引用对象的一个"指向"。所以他们两者之间的区别就是两个类型的区别。
举个简单的例子，代码如下：

```
class Temperature {
  var value: Float = 37.0
}

class Person {
  var temp: Temperature?

  func sick() {
    temp?.value = 41.0
  }
}

let A = Person()
let B = Person()
let temp = Temperature()

A.temp = temp
B.temp = temp

A.sick()

```

上面这段代码，由于 Temperature 是 class ，为引用类型，故 A 的 temp 和 B 的 temp指向同一个对象。A 的 temp修改了，B 的 temp 也随之修改。这样 A 和 B 的 temp 的值都被改成了 41.0。如果将 Temperature 改为 struct，为值类型，则 A 的 temp 修改不影响 B 的 temp。

内存中，引用类型诸如类是在堆（heap）上，而值类型诸如结构体是在栈（stack）上进行存储和操作。相比于栈上的操作，堆上的操作更加复杂耗时，所以苹果官方推荐使用结构体，这样可以提高 App 运行的效率。

加分回答：

class 有这几个功能 struct 没有的：

*   class 可以继承，这样子类可以使用父类的特性和方法；
*   类型转换可以在 runtime 的时候检查和解释一个实例的类型；
*   可以用 deinit 来释放资源；
*   一个类可以被多次引用。

struct 也有这样几个优势：

*   结构较小，适用于复制操作，相比于一个 class 的实例被多次引用更加安全；
*   无须担心内存 memory leak 或者多线程冲突问题。

**类似问题：**

> 引用类型和值类型有什么区别？ Struct 相比 class 在使用上有什么优势？

### 2\. Swift 是面向对象还是函数式的编程语言？

**关键词：#面向对象 #函数式编程**

Swift 既是面向对象的，又是函数式的编程语言。

说 Swift 是面向对象的语言，是因为 Swift 支持类的封装、继承、和多态，从这点上来看与 Java 这类纯面向对象的语言几乎毫无差别。

说 Swift 是函数式编程语言，是因为 Swift 支持 map, reduce, filter, flatmap 这类去除中间状态、数学函数式的方法，更加强调运算结果而不是中间过程。

类似问题：
**为什么说 Swift 是函数式的编程语言？**

### 3\. 在 Swift 中，什么是可选型（optional） ？

**关键词：#Optional #nil**

在 Swift 中，可选型是为了表达当一个变量值为空的情况。当一个值为空时，它就是 nil。Swift 中无论是引用类型或是值类型的变量，都可以是可选型变量。举个例子：

```
// 值类型Float，value 默认值为37.0
var value: Float? = 37.0
// 值类型String，key 默认值为 nil
var key: String? = nil
// 引用类型 UIImage，image 默认值为 nil
let image: UIImage?

```

加分回答：

Objective-C 中没有明确提出可选型的概念，然而其引用类型却可以为 nil，以此来标识其变量值为空的情况。Swift 将这一理念扩大到值类型，并且明确提出了可选型的概念。

### 4.在 Swift 中，什么是泛型（Generics）？

**关键词：#泛型**

泛型在 Swift 中主要为增加代码的灵活性而生：它可以使得对应的代码满足任意类型的变量或方法。

举个简单的例子。我们要写出一个方法，可以交换两个 Int 值，一种写法如下：

```
func swap(_ a: inout Int, _ b: inout Int) {
  (a, b) = (b, a)
}

```

上面这种写法正确但并不高效。因为假如要再实现一个方法去交换两个 Float 值，那又得重写一遍。泛型就是为了解决这类问题而来，我们希望有一个一般性的方法，可以交换任意类型的变量：

```
func swap<T>(_ a: inout T, _ b: inout T) {
  (a, b) = (b, a)
}

```

注意，Swift 是类型安全的语言，所以这里交换的两个变量其类型必须一致。

### 5\. 请说明并比较以下关键词：Open, Public, Internal, File-private, Private

**关键词：#访问控制权限**

Swift 有五个级别的访问控制权限，从高到底依次为比如 Open, Public, Internal, File-private, Private。

他们遵循的基本原则是：高级别的变量不允许被定义为低级别变量的成员变量。比如一个 private 的 class 中不能含有 public 的 String。反之，低级别的变量却可以定义在高级别的变量中。比如 public 的 class 中可以含有 private 的 Int。

*   **Open** 具备最高的访问权限。其修饰的类和方法可以在任意 Module 中被访问和重写；它是 Swift 3 中新添加的访问权限。

*   **Public** 的权限仅次于 Open。与 Open 唯一的区别在于它修饰的对象可以在任意 Module 中被访问，但不能重写。

*   **Internal** 是默认的权限。它表示只能在当前定义的 Module 中访问和重写，它可以被一个 Module 中的多个文件访问，但不可以被其他的 Module 中被访问。

*   **File-private** 也是 Swift 3 新添加的权限。其被修饰的对象只能在当前文件中被使用。例如它可以被一个文件中的不同 class，extension，struct 共同使用。

*   **Private** 是最低的访问权限。它的对象只能在定义的作用域内及其对应的扩展内使用。离开了这个对象，即使是同一个文件中的对象，也无法访问。

类似问题：
**Swift 3 中新引入的 Open 和 File-private 关键词有什么用？**

### 6\. 请说明并比较以下关键词：strong, weak, unowned

**关键词：#引用类型 #内存管理**

Swift 的内存管理机制与 Objective-C一样为 ARC（Automatic Reference Counting）。它的基本原理是，一个对象在没有任何强引用指向它时，其占用的内存会被回收。反之，只要有任何一个强引用指向该对象，它就会一直存在于内存中。

*   **strong** 代表着强引用，是默认属性。当一个对象被声明为 strong 时，就表示父层级对该对象有一个强引用的指向。此时该对象的引用计数会增加1。

*   **weak** 代表着弱引用。当对象被声明为 weak 时，父层级对此对象没有指向，该对象的引用计数不会增加1。它在对象释放后弱引用也随即消失。继续访问该对象，程序会得到 nil，不会崩溃。

*   **unowned** 与弱引用本质上一样。唯一不同的是，对象在释放后，依然有一个无效的引用指向对象，它不是 Optional 也不指向 nil。如果继续访问该对象，程序就会崩溃。

**加分回答：**

weak 和 unowned 的引入是为了解决由 strong 带来的循环引用问题。简单来说，就是当两个对象互相有一个强指向去指向对方，这样导致两个对象在内存中无法释放（详情请参考第3章第3节第8题）。

weak 和 unowned 的使用场景有如下差别：

*   当访问对象时该对象可能已经被释放了，则用 weak。比如 delegate 的修饰。

*   当访问对象确定不可能被释放，则用 unowned。比如 self 的引用。

*   实际上为了安全起见，很多公司规定任何时候都使用 weak 去修饰。

### 7\. 在 Swift 中，怎样理解是 copy-on-write？

**关键词：#内存管理**

当值类型比如 struct 在复制时，复制的对象和原对象实际上在内存中指向同一个对象。当且仅当复制后的对象进行修改的时候，才会在内存中重新创建一个新的对象。举个例子：

```
// arrayA 是一个数组，为值类型
let arrayA = [1, 2, 3]
// arrayB 这个时候与 arrayA 在内存中是同一个东西，内存中并没有生成新的数组
var arrayB = arrayA
// arrayB 被修改了，此时 arrayB 在内存中变成了一个新的数组，而不是原来的 arrayA
arrayB.append(4)

```

上面的代码中我们可以看出，复制的数组和原数组共享同一个地址直到其中之一发生改变。这样设计使得值类型可以多次复制而无需耗费多余内存，只有变化的时候才会增加开销。**因此内存的使用更加高效**。

### 8\. 什么是属性观察（Property Observer）？

**关键词：#willSet #didSet**

属性观察是指在当前类型内对特定属性进行监视，并作出响应的行为。它是 Swift 的特性，有两种，为 willSet 和 didSet。举个例子：

```
var title: String {
  willSet {
    print("将标题从\(title)设置到\(newValue)")
  }
  didSet {
    print("已将标题从\(oldValue)设置到\(title)")
  }
}

```

上面这段代码对于 title 做了监听。当 title 发生改变前，willSet 对应的作用域将被执行，新的值是 newValue；当 title 发生改变之后，didSet 对应的作用域将被执行，原来的值为 oldValue。这就是属性观察。

**加分回答：**

初始化方法对属性的设定，以及在 willSet 和 didSet 中对属性的再次设定都不会触发属性观察的调用。

## Swift 面试实战题

### 9\. 结构体中修改成员变量的方法

**关键词：#mutating**
请问下面代码有什么问题？

```
protocol Pet {
  var name: String { get set }
}

struct MyDog: Pet {
  var name: String

  func changeName(name: String) {
    self.name = name
  }
}

```

应该在 func changeName(name: String) 前加上关键词 mutating，表示该方法将会修改结构体中自己的成员变量。

> 注意，在设计协议的时候，由于protocol 可以被 class 和 struct 或者 enum 实现，故而要考虑是否用 mutating 来修饰方法。

类（class）中不存在这个问题，因为类可以随意修改自己的成员变量。

### 10\. 用 Swift 实现或（||）操作

**关键词：#autoclosure**

这题解法很多，下面给出一种最直接的解法：

```
func ||(left: Bool, right: Bool) –> Bool {
  if left {
    return true
  } else {
    return right
  }
}

```

上面这种解法勉强正确，但是并不高效。或（||）操作的本质是当左边为真的时候，我们无需计算右边。而上面这种事先，是将右边默认值预先准备好，再传入进行操作。当右边值的计算十分复杂时会 造成了性能上的浪费。所以，上面这种做法违反了或（||）操作的本质。正确的实现方法如下：

```
func ||(left: Bool, right: @autoclosure () -> Bool) –> Bool {
  if left {
    return true
  } else {
    return right()
  }
}

```

autoclosure 可以将右边值的计算推迟到判定 left 为 false 的时候，这样就可以避免第一种方法带来的不必要开销了。

### 11\. 实现一个函数。输入是任一整数，输出要返回输入的整数+ 2

**关键词：#柯里化**

这道题看似简单，直接这样写：

```
func addTwo(_ num: Int) -> Int {
    return num + 2
}

```

接下来面试官会说，那假如我要实现 + 4 呢？程序员想了一想，又定义了另一个方法：

```
func addFour(_ num: Int) -> Int {
    return num + 4
}

```

这时面试官会问，假如我要实现返回 + 6, + 8 的操作呢？能不能只定义一次方法呢？正确的写法是利用
Swift 的 Currying 特性：

```
func add(_ num: Int) -> (Int) -> Int {
  return { val in
    return num + val
  }
}

let addTwo = add(2), addFour = add(4), addSix = add(6), addEight = add(8)

```

Swift 中的柯里化（柯里化） 特性是函数式编程思想的体现。它将接受多个参数的方法进行变形，并用高阶函数的方式进行处理，使整个代码更加灵活。

### 12\. 实现一个函数。求 0 到 100（包括0和100）以内是偶数并且恰好是其他数字平方的数。

**关键词：#函数式编程**

这道题十分简单。最简单粗暴的写法如下：

```
func evenSquareNums(from: Int, to: Int) -> [Int] {
  var res = [Int]()

  for num in from...to where num % 2 == 0{
    if (from...to).contains(num * num) {
      res.append(num * num)
    }
  }

  return res
}

evenSquareNums(from: 0, to: 100)

```

上面的写法正确，但不够优雅。首先这个方法完全可以利用泛型进行优化，同时可以在创建 res 数组时加上 reserveCapacity 以保证其性能。其实面对这个题目，最简单直接的写法是用函数式编程的思路，一行既可以解决问题：

```
(0...10).map { $0 * $0 }.filter { $0 % 2 == 0 }

```

Swift 有函数式编程的思想。其中 flatMap, map, reduce, filter 是其代表的方法。本题中考察了 map
和 filter 的组合使用。相比于一般的 for 循环，这样的写法要更加得简洁漂亮。

# 推荐👇：

如果你正在跳槽或者正准备跳槽不妨动动小手，添加一下咱们的交流群[**931 542 608**](https://jq.qq.com/?_wv=1027&k=cfVxrMAH)来获取一份详细的大厂面试资料为你的跳槽多添一份保障。

![](https://upload-images.jianshu.io/upload_images/22877992-0bfc037cc50cae7d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
