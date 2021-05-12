iOS 开发的官方 IDE 是 Xcode，它也是 Apple 平台最主流的开发工具。目前 Xcode 已经更新到第 9 个版本，功能也是涵盖开发、测试、性能分析、文档查询、源代码管理等多个方面，可谓是 App 开发一站式的平台。

![](https://upload-images.jianshu.io/upload_images/22877992-f05c385e0b9a2e37.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

Xcode 诞生于 2003 年，发展至今，已经可以支持除 Objective-C 和 Swift 之外其他 6 种语言：C、C++与 Objective-C 密不可分；自动化方面则多用 Ruby，例如我们熟知的 fastlane 和 cocoapods；Automation 工具的脚本大多采用 Javascript； 刚刚发布的 CoreML 采用的模型工具则是用 Python 编成。最新的 Xcode 采用完全由 Swift 重写的 Souce Editor，在代码修改、补全、模拟器运行方面有了很大提升。目前最大的缺点是稳定性不够。

对于 iOS 工程师而言，熟练运用 Xcode 是必备技能 ，而对 Xcode 的理解深浅亦是工程师水平的分水岭。本节将从基本的 Xcode 开发知识开始，逐渐深入到 Intruments 性能分析和 LLDB 调试，针对 Swift 专门设计的 Playground 也将有所涉及。

## Xcode 调试

### 1\. LLDB 中 p 和 po 有什么区别？

**关键词：#调试 #命令**

*   p 是 expr – 的缩写。它做的工作是把接收到的参数在当前环境下编译，然后打印出对应的值。
*   po 是 expr –o– 的缩写。它所做的操作与 p 相同。如果接收到的参数是个指针，它会调用对象的 description 方法，并进行打印；如果是个 core foundation 对象，那么会调用 CFShow 方法，并进行打印。如果这两个方法都调用失败，po 打印出和 p 相同的内容。
*   总的来说 po 相对于 p 会打印出更多内容。一般工作中，用 p 即可，因为 p 操作较少效率较高。

### 2.Xcode 中的 Runtime issues 和 Buildtime issues 指什么？

![image](https://upload-images.jianshu.io/upload_images/22877992-5beaab3d4d2067d1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**关键词：#调试 #编译器**

*   Buildtime issues 有三类：编译器识别出的警告（Warning），错误（Error），以及静态分析（Static Code Analysis）。前两者无须赘述，静态分析错误一般有这几类：未初始化的变量，未使用数据，API 使用错误。比如下面一段代码：

```
class SampleViewController: UIViewController {
  override func viewDidLoad() {
    let numList: [Int]
    let otherNumList = numList
    let anotherNumList: [Int]?  
  }
}

```

这段代码中有三个错误。首先 numList 未初始化就赋值给 otherNumList ；其次 anotherNumList 并未使用；最后是 API 使用错误，没有调用 super.viewDidLoad() 方法。

*   Runtime issues 有三类：线程问题，UI 布局和渲染问题，以及内存问题。线程相关问题有很多，最常见的就是数据竞争（data race）。比如下面这段代码：

```
var balance = 0
let fullTimeSalary = 1000, partTimeSalary = 1000
DispatchQueue.global().async {
  for _ in 1...12 {
    balance += partTimeSalary
  }
}
for _ in 1...12 {
  balance += fullTimeSalary
}

```

这段代码中两个线程同时对 balance 进行写操作，谁先写、balance 值为多少就会变成一个两个线程角力的情况。这种多线程对同一个值进行写操作的行为就是数据竞争。

UI 布局问题就是诸如尺寸设定没给全或者设定模糊，autolayout 引擎无法渲染的问题。内存问题最常见的就是内存泄漏，比如循环引用就是一个经典的错误。

## 分析与优化

### 3\. App 启动时间过长，该怎样优化？

**关键词：#调试 #启动优化**

App 启动时间过长，可能有多个原因造成。理论上 App 的启动时间是由 main() 函数之前的加载时间（t1）和 main() 函数之后的加载时间（t2）。

关于 t1 我们需要分析 App 的启动日志，具体方法是在 Xcode 中添加 DYLD_PRINT_STATISTICS
环境变量，并将其值设置为 1，这样就可以得到如下的启动日志：

```
Total pre-main time: 1.3 seconds (100.0%)
         dylib loading time: 107.45 milliseconds (8.0%)
        rebase/binding time: 376.56 milliseconds (28.2%)
            ObjC setup time: 166.96 milliseconds (12.5%)
           initializer time: 684.01 milliseconds (51.2%)
           slowest intializers :
               libSystem.dylib : 297.56 milliseconds (22.2%)
    libMainThreadChecker.dylib :  33.00 milliseconds (2.4%)
        libLLVMContainer.dylib : 113.09 milliseconds (8.4%)
                       ModelIO : 189.45 milliseconds (14.1%)

```

然后我们就可以知道，App 启动主要在这三个方面耗费时间，动态库加载，重定位和绑定，以及对象的初始化。所以优化的手段也有了，简单来说就是：

*   减少动态库数量，dylib loading time 会下降，苹果的推荐是动态库不要多于 6 个
*   减少 Objective-C 的类数量，例如合并或者删除，这样可以加快动态链接，rebase/binding time 会下降
*   使用 initialize 方法替换 load 方法，或是尽量将 load 方法中的代码延后调用，initializer time 会下降

关于 t2，主要是构建第一个界面并完成渲染的时间。所以这个需要在具体的界面布局和渲染代码中进行打点观察，诸如 viewDidLoad 和 viewWillAppear 这两个函数就很值得关注。

### 4.如何用 Xcode 检测代码中的循环引用？

**关键词：#调试 #内存检测**

有两种方法可以检测。

其一是使用 Xcode 中的 Memory Debug Graph。点击下图所示的调试工具栏中的按钮，Xcode 会自动检测内存相关的 memory runtime issue。点击相关问题处 Xcode 就会给出详细的循环引用示意图。

![image](https://upload-images.jianshu.io/upload_images/22877992-a450fa65290976bc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

另一种解决方法是用 Instruments 里面的 Leak 选项——这是一个专门检测内存泄漏的工具。进入页面后发现 Leak Checks 中出现内存泄漏时，我们可以将导航栏切换到 call tree 模式下，强烈建议在 Display Settings 中勾选 Separate by Thread 和 Hide System Libraries 两个选项，这样可以隐藏掉系统和应用本身的调用路径，帮助我们更方便的找出 retain cycle 位置。

![image](https://upload-images.jianshu.io/upload_images/22877992-5ca9bff4d6bc9913.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 5\. 该怎样解决 EXC_BAD_ACCESS？

**关键词：#调试**

EXC_BAD_ACCESS 主要原因是访问了某些已经释放的对象，或者访问了它们已经释放的成员变量或方法。解决方法主要有以下几种：

*   设置全局断点快速定位 bug 所在，这种方法效果一般；

*   重写 object 的 respondsToSelector 方法，这种方法效果一般且要在每个 class 上进行定点排查，不推荐；

*   使用 Zombie 和 Address Sanitizer，可以在绝大多数情况下定位问题代码，如下图：

![image](https://upload-images.jianshu.io/upload_images/22877992-97579882275e48f9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## Playground 技巧

### 6.在实际开发中，我们会测试网络请求收到的数据。要调试 api.org/get 是否工作，工程师在 Playground 中写下了以下代码。假设 API 和网络正常工作，请问这段程序将会打印出什么内容？

**关键词：#调试 #延时运行**

```
let url = URL(string: “api.org/get”)
let task = URLSession.shared.dataTask(with: url!) { (data, response, error) in
  do {
    let dictionary = try JSONSerialization.jsonObject(with: data!, options: [])
    print(dictionary)
  } catch {
    print(“error!”)
  }
}

```

答案是：什么内容都不会打印出来。原因是 Playground 执行完了所有语句，自动退出。如果要让 Playground 具备延时运行的特性，可以在 Playground 中加上一下代码：

```
import PlaygroundSupport
PlaygroundPage.current.needsIndefiniteExecution = true

```

这样我们就可以打印出返回的 dictionary 中的内容了。

### 7\. 代码实现：请在 playground 中实现一个 10 行的列表，每行随机显示一个 0 – 100 之间的整数。

**关键词：#调试 #可视化开发**

本题主要考察面试者的基本编程能力，对于 API 的熟悉程度和 Playground 可视化编程的了解。完整代码如下：

```
import UIKit
import PlaygroundSupport

class ViewController: UIViewController {
    lazy var tableView: UITableView = {
       return UITableView()
    }()
    lazy var nums: [Int] = {
        var array = Array(0...99)
        array.shuffle()
        return array
    }()

    struct Constants {
        static let CellIndentifier = "defaultCell"
    }

    override func viewDidLoad() {
        super.viewDidLoad()

        // autolayout for tableView
        tableView.translatesAutoresizingMaskIntoConstraints = false

        view.addSubview(tableView)

        let tableViewContraints = [
            tableView.topAnchor.constraint(equalTo: view.topAnchor),
            tableView.leadingAnchor.constraint(equalTo: view.leadingAnchor),
            tableView.widthAnchor.constraint(equalTo: view.widthAnchor),
            tableView.heightAnchor.constraint(equalTo: view.heightAnchor)
        ]
        NSLayoutConstraint.activate(tableViewContraints)

        // set up tableView
        tableView.dataSource = self
        tableView.register(UITableViewCell.self, forCellReuseIdentifier: Constants.CellIndentifier)
    }
}

extension ViewController: UITableViewDataSource {
    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return nums.count
    }

    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        let cell = tableView.dequeueReusableCell(withIdentifier: Constants.CellIndentifier, for: indexPath)

        cell.textLabel?.text = String(nums[indexPath.row])

        return cell
    }
}

PlaygroundPage.current.liveView = ViewController()

```

# 推荐👇：

如果你正在跳槽或者正准备跳槽不妨动动小手，添加一下咱们的交流群[**931 542 608**](https://jq.qq.com/?_wv=1027&k=cfVxrMAH)来获取一份详细的大厂面试资料为你的跳槽多添一份保障。

![](https://upload-images.jianshu.io/upload_images/22877992-0bfc037cc50cae7d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
