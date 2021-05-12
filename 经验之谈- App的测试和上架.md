很多程序员在完成开发后，最期待的就是模拟器上一遍跑通，然后就可以交差了。其实专业的 iOS 开发者除了在开发前十分周全的计划，开发中考虑各种细节问题和边界情况，开发后还会做大量的测试。

![](https://upload-images.jianshu.io/upload_images/22877992-7145c0fad0e71aed.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

测试在我看来分为三种。第一种是普通的单元测试、UI 测试、性能测试，对于某个模块甚至会做大量的集成测试，这类测试基本上检验了软件上所有可能的逻辑漏洞。第二种测试是真机测试，一般大公司会配备专业的 QA 去手动测试各种情况，之前第一种测试难以覆盖的情况以及一些必须要硬件真机去测试的情况属于此种测试。第三种测试就是将 App 的 beta 版本放在 Testflight 上进行内测，这种测试将邀请特定用户进行体验，以做最后的功能校验。在硅谷，测试一直被看做工程师日常工作的一部分，甚至某些公司在开发上采用了依据测试来写代码的 TDD（Test Driven Development）模式。遗憾的是，因为各种原因，目前国内的互联网公司在测试产品上主要依靠 QA 完成。

我们作为专业的 iOS 开发者，虽然无需深度掌握测试技能，但至少应该明白测试的重要性，并能独立完成基本的测试操作。在确保 App 安全无虞的上架、日后类似的 bug 不再重犯，测试的效果无可替代。

在完成测试后，也并不代表着 App 就一定可以通过苹果的审核进入商店。苹果官方有明确的审核指南。本节亦会挑选常见的 App Store 相关的上传、下载、审核问题进行探讨。

## 测试相关

### 1.一个 App 崩溃了，可能是什么原因造成的？

**关键词：#代码 #内存 #网络 #第三方**

*   **代码出错。**利用了 Objective-C 的动态性能，编译时不会报错，结果运行之后程序找不到对应的实现，产生崩溃。比如下面这个例子。

```
// o1 和 o2 有实现方法 myMethod，但是 o3 没有
MyObject *o1 = ...
MyObject *o2 = ...
NSObject *o3 = ...

NSArray *array = @[o1, o2, o3];
for (id obj in array) {
  [obj meMethod];
}

// Runtime error: unrecognized selector sent to instance XXX

```

*   **内存不够。**比如 App 在运行时占用了手机大量的内存，此时App就会崩溃。经常发生在低配或内存容量很少的手机。这个问题可以通过 Xcode Instruments 调试判断出来。

*   **网络原因。**当网络不佳时，App 的请求得不到即时的响应而导致的超时；或是用户数量太多，服务器端过载而影响到手机端崩溃。其实这些都可以在优化服务器端配置和处理手机端异常中改进用户体验。

*   **第三方。**开发中使用了第三方的工具有可能有病毒或是 bug。另外广告的弹出也可能很阻塞线程或侵占内存，导致 App 崩溃。

一般解决 App 崩溃的方式是检查对应的机器日志。国外主流的检测工具是 twitter 开发、google 维护的 Fabric。国内主流的工具是腾讯的 Bugly。

### 2.在模拟机上完成所有测试之后，是否就不需要在实机上再进行测试了？

**关键词：#功能 #硬件**

答案是，需不需要实际测验要看具体情况。模拟机可以完成绝大多数的功能检测。但是真机和模拟机的差别还是存在的，主要集中在功能和硬件上：

*   **功能方面。**模拟器不支持 Email、通话、短信等功能，同时也不支持 Accessibility 的 VoiceOver功能，如果 App 是支持残疾人使用的，请务必在真机上测试。

*   **硬件方面。**模拟器不支持相机、音频输入、蓝牙等硬件功能。如果 App 支持手环诸如 Apple Watch 联动，请务必在真机上测试。

如果 App 不会涉及到这些差异，那理论上无需用真机进行测试。当然谨慎起见，如果时间充裕是一定要将主要功能在真机上测试的。

如果你正在跳槽或者正准备跳槽不妨动动小手，添加一下咱们的交流群[**101 295 1431**](https://jq.qq.com/?_wv=1027&k=SSQAVXir)来获取一份详细的大厂面试资料为你的跳槽多添一份保障。

### 3.为什么在单元测试中引入代码模块要用 `@testable` 关键词？

**关键词：#internal**

测试时，我们经常需要导入开发中的 module。普通的 import module 虽然完成了导入，但是只能调用 module 的 public 变量和方法。

单元测试和UI测试中，很多 public 的方法是多个内部方法的整合，与其测试复杂的 public 方法，不如单独测试其组成的一个个小的 internal 方法。

此时如果能够调用 module 中的 internal 变量和方法，那么将会大大方便测试。`@testable import` 就表示，module 中的 internal 变量和方法也可以在测试中被使用。

### 4.代码实战：试着对下面的方法写出对应的单元测试

**关键词：#异步 #mock**

```
func loadContent() {
  let url = "https://app-info.rtf"
  let session = URLSession.shared
  let client = HTTPClient(session: session)

  client.get(url: url) {(data, error) in
    if let error = error {
      print("Error: \(error)")
      return
    }

    if let data = data {
      print("Data is successfully fetched from server")
    }
  }
}

```

上面这段代码，是一段访问服务器端返回数据的方法。这道题如果用来测试，涉及到两个知识点：第一个是如何测试异步访问，第二个是使用 mock。我们来分别解释。

首先，如何测试异步访问。用 expectation 。本题中我们设定好 expectation 中网络端会返回 data，然后在异步的线程中调用 fulfill() 方法，即表示异步成功结束时会触发。接着我们等待异步结束，当然我们会设定超时的阈值。

其次，为什么要使用 mock。测试中， 访问服务器端并接收到数据返回是不切实际的举动：首先如果测试时真的调用服务器接口，你无法保证服务器返回的数据是什么，会不会报错，也就无法准确的测试各种情况；其次，调用接口牵扯到真实的服务器逻辑，会修改服务器数据，对于测试来讲这显然没有必要；最后，每次访问服务器端再返回数据比较耗时，这样整个测试效率很差。所以我们可以模拟服务器返回数据的过程，用一个假的 client 去“装模作样”地访问服务器端，并且从本地直接返回确定好的数据。至此整个操作就无需真的依赖网络，并且我们可以就各种返回情况进行模拟测试。

下面是示例代码：

```
var dataLoaded: Data?

func test_loadContent_shouldReturnData() {
  let url = "https://app-info.rtf"
  let session = MockSession()
  let client = HTTPClient(session: session)

  // 用NSPredicate来过滤条件，只有dataLoaded不为nil才会被记录
  let pred = NSPredicate(format: "dataLoaded != nil")
  let exp = expectation(for: pred, evaluateWith: self, handler: nil)

  client.get(url: url) { [weak self] (data, error) in
    self?.dataLoaded = data
    // 当异步成功结束时触发expectation
    exp.fulfill()
  }
  // 等待expectation被触发，超时时间设定为5秒
  wait(for: [exp], timeout: 5.0)
  // 判断expectation出发后dataLoaded是否不为nil，否则测试失败
  XCTAssertNotNil(dataLoaded, "No data is received!")
}

```

### 5.谈谈 iOS 中的性能测试（performance test）？

**关键词：#耗时 #scheme**

所谓性能测试，就是检测一个方法快慢的测试。我们一般设定一个基础值，比如 0.01s，然后运行性能测试，测试后会显示本次测试耗时以及平均运行耗时。你可以跟基础值进行比较，并且设定最大上限，比如 10%。这样如果测试超过最大上限耗时，比如 0.01s * 1.1 = 0.011s，那么此次测试就失败了。性能测试的示例图如下：

![image](https://upload-images.jianshu.io/upload_images/22877992-954fe8aa252457be.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

性能测试一般用在分析那些可能会很耗时的方法上。比如在设备上存取操作、网络端的请求、复杂的计算等等。

注意性能测试和 Instruments 的性能优化不同，前者是 App 的性能的底线：如果不满足性能测试的时间标准，那么用户体验将会受到极大影响，甚至被苹果拒绝上架。后者则是在性能上锦上添花的优化操作，是一个可以提高用户体验的任务。性能优化有时就算不做，也无伤大雅。性能测试则是要求方法必须满足指定的耗时要求。

一般情况下，建议单独开一个专门的 scheme 来运行性能测试。这样可以清晰得将其和单元测试或是 UI 测试区分开来，借用快捷键 cmd+U 来单独运行性能测试也更加方便。

如果你正在跳槽或者正准备跳槽不妨动动小手，添加一下咱们的交流群[**101 295 1431**](https://jq.qq.com/?_wv=1027&k=SSQAVXir)来获取一份详细的大厂面试资料为你的跳槽多添一份保障。

### 6.谈谈 iOS 中的 UI 测试？

**关键词：#record #XCUIElement #Identifier #iPhone vs. iPad**

首先 UI 测试特殊的地方在于。我们并不需要完全的手写代码，Xcode 的 record 功能可以自动生成 UI 测试代码。我们只需给出判断条件和代码优化即可。

其次 UI 测试的 API 中有这几个值得注意。XCUIApplication 对应的实例是应用的入口，其次所有的UI控件都是 XCUIElement。UI 测试是根据它们对应的 title 属性进行指定（如果有 title 重名的情况，则选择 XCTest 框架搜索到的第一个对应 UI 控间）。我们当然还可以通过 accessibility Identifier 来指定一个 UI 控件。所以我们一般 UI 测试都是通过具体行动（点击、滑动）之后比较不同 UI 控件的状态，异或是寻找指定页面出现的 UI 控件来进行测试。

最后 UI 测试会牵涉不同机器不同尺寸的问题。比如 iPhone 用的是 tableView 而 iPad 用的是 splitView，由于 UI 布局不同，UI 控件的位置差异也是需要特殊处理的。

UI 测试更关注的是用户行为/体验，而单元测试则关注单个方法的逻辑正确。**UI测试能覆盖到单元测试都无法覆盖到的部分**，例如：

1.  在给定输入时，输出通过了单元测试；但实际上输出的格式并不满足要求，在屏幕上也会因为尺寸问题被缩进。这时就需要 UI 测试来检查。
2.  键盘在某界面会无故弹出却无法收起。此时程序在逻辑上正确，单元测试毫无问题；然而 UI 测试却可以检测出屏幕上某些 UI 控件因为被键盘遮挡而无法点击。

### 7.如何检查测试覆盖率？

**关键词：#coverage**

运行完测试之后，切换到日志导航，点击刚刚测试的结果，在导航栏上点击 Coverage 即可得到如下测试覆盖率示意图：
![image](https://upload-images.jianshu.io/upload_images/22877992-b80d7f1c295c45d1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我们不仅可以查看整个 App 的测试覆盖率，也可以查看每个文件的测试覆盖率。单独点击一个文件进入其中，红色部分表示测试没有覆盖到的地方。

代码覆盖率越高说明测试越完善。当然我们不必追求 100% 的代码覆盖率。注意测试覆盖率一般以运行完所有单元、性能、UI 测试之后的数据为准。

## App Store相关

### 8.什么是 iOS 中的 App ID？

**关键词：#teamid #bundleid**

每一个 App 都有独立的 ID 来唯一确定，这就是 App ID。它由两部分构成：Team ID 和 Bundle ID。两者以点区分，组合在一起就是 App ID 的形式：Team ID.Bundle ID。

Team ID 指定 App 是由某个具体的开发者或团队开发。Bundle ID 指定 App 或与之相关的一系列 App。Bundle ID 可以唯一确定 App。

Bundle ID 是在 Xcode 项目中确定的。一个单独的 Xcode 项目可能有多个目标文件，对应也可能产生多个 App。比如 beta 版和 pro 版，付费版和免费版等等。

### 9.什么是 iOS 中的 Code Signing？

**关键词：#密匙 #安全**

为了确定 App 是谁开发，开发之后有没有被修改，Apple 引进了 Code Signing 的机制。

有了它，在从 App Store 下载 App 后，iOS 和 MacOS 系统可以通过签名确认是谁开发了 App，以及签名是否有效。

只要 App 对应的可执行的文件被修改，签名就认定为无效。对于无效的签名系统将拒绝运行 App，以保证整个系统的安全性和用户体验。

Code Signing 对应的签名是由一对公共和私有的密匙，以及一个由 Apple 签发的证书构成。其中私有的密匙用来产生签名；证书则包含了公共密匙并由此认定 App 的开发者。

### 10.什么是 iOS 中的 App Thinning？

**关键词：#最小**

App Thinning ，中文翻译为“应用瘦身”，指的是 App store 和操作系统在安装 iOS 或者 watchOS 的 App 的时候通过一些列的优化，尽可能减少安装包的大小，使得 App 以最节省资源的、最合适的大小被安装到你的设备上。其中有三种类型：slicing, bitcode, and on-demand resources。

*   Slicing 指的是根据不同的设备，App 对应产生相应的版本。如 iPad 版本只包含 iPad 版本的图片资源和布局代码，iPhone 版本则类似。此时下载 App 的时候，只需要下载对应版本的 App 即可。

*   Bitcode 是一个 llvm 编译 App 时生成的中间形式。上传或下载新版本的 App 时，苹果会针对 Bitcode 包含的信息进行针对性地添加或筛选，而不是完整地提交或下载一个新的 App。在 iOS 中它是可选的，在 WatchOS 中 Bitcode 则是必须的。

*   On-Demand Resources 是只提供部分的 App 内容，只要足以满足其基本运行即可。比如某些游戏 App，一开始下载之后只能运行最初的内容，而不是全部的内容。如果玩家有兴趣继续探索，App Store 就会解锁后续内容，将其下载更新到游戏中。

### 11.向 App Store 提交 App 有哪些可能被拒的原因？

**关键词：#崩溃 #第三方 #版权 #材料不全**

App Store 的审核虽然现在越来越快，被拒绝的成本越来越低，但是做到在提交 App 之前仔细检查，争取一次性通过，依然是 iOS 开发者的基本素养。

被拒绝的原因有很多，最主要的有以下几种：

*   **崩溃。**程序本身有 bug、第三方服务器出错都有可能。注意我们平常测试是在线下环境中跑 App，而App Store 是在线上环境运行。所以提交审核的时候，还是应该在线上环境运行以防万一。

*   **第三方。**如 App 需要安装第三方应用，比如需要 QQ 登录，而测试员的手机中又没有装 QQ，如果出现提示安装 QQ，就可能被拒；另外使用第三方的广告，也有可能因为违规被拒。

*   **版权。**比如第三方客户端套用某平台的名字；App 描述或命名中为了点击和排名硬塞某些无关的关键词；亦或是山寨现成 App 的行为；App 中包含没有授权的内容也是被拒的理由。注意苹果对某些关键词（比如 Android）非常敏感，绝对不要出现在 App 的提交中。

*   **材料不全。**有时 App 会因为缺少材料导致 App Store 无法审核。比如缺少截图或者使用错误的截图；与硬件相关的 App 提交时，官方没有相关硬件，此时需要开发者提供相关视频。

上面只是部分案例。苹果官方有专门的审核导读文件（App Store Review Guidelines）
，建议开发者在上传 App 前应该仔细研读，并一一检查。

# 推荐👇：

如果你正在跳槽或者正准备跳槽不妨动动小手，添加一下咱们的交流群[**931 542 608**](https://jq.qq.com/?_wv=1027&k=cfVxrMAH)来获取一份详细的大厂面试资料为你的跳槽多添一份保障。

![](https://upload-images.jianshu.io/upload_images/22877992-0bfc037cc50cae7d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
