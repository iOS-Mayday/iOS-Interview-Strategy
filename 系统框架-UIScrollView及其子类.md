UIScrollView 恐怕是所有 App 都绕不过去的类——尤其是它的子类 UITableView 和 UICollectionView。看看我们日常常见的 App，新闻类的今日头条，社交类的微博和微信，电商类的淘宝、腾讯，日常管理用的备忘录和图片 App 的缩放功能，都或多或少得使用了 UIScrollView 及其子类。

![](https://upload-images.jianshu.io/upload_images/22877992-841bdcd834a1cc4e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

当一个屏幕无法展示 App 需要展示的所有内容时，就是 UIScrollView 大展拳脚的时候：通过使用 UIScrollView，用户可以滑动或是缩放屏幕，来看单个屏幕无法展示的内容。如何定制不同 Cell 的 UI、如何与用户交互、如何与服务器端数据同步、如何在滑动时最大限度保证界面的流畅，这些都是考察的要点，是一个 iOS 工程师必备的基本技能。

## 基本理论

### 1.请说明并比较以下关键词：contentView，contentInset，contentSize，contentOffset。

**关键词：#UIScrollView**

*   **UIScrollView 上显示内容的区域被称为 contentView。**一般情况下我们对 UIScrollView 的操作，例如 addSubview 这样的操作都是在 contentView 上进行。

*   **contentInset 是指 contentView 与 UIScrollView 的边界。**与网页开发的 padding 类似，分别指 contentView 的四条边到 UIScrollView 的对应边的距离，分别为 top，bottom，left，right。

*   **contentSize 是指 contentView 的大小。**它一般超过屏幕大小，是整个 UIScrollView 实际内容的大小。比如一张图片有四个屏幕之大，我们在缩放的时候只能看到其 1/4 的内容，那么它的 contentSize 就是四个屏幕合起来的尺寸大小。

*   **contentOffset 是当前 contentView 浏览位置左上角点的坐标。**它是相对于整个 UIScrollView 左上角为左边原点而言。默认为 CGPointZero。

下图是对这几个关键词的说明：
![image](https://upload-images.jianshu.io/upload_images/22877992-44cd32767da40d28.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 2\. 请说明 UITableViewCell 的重用机制

**关键词：#UITableViewCell #reuseIdentifier**

UITableView 的每一行就是 UITableViewCell。绝大多数 UITableViewCell 的构图都一样，只是内容不同而已。所以我们将同一类型的 UITableViewCell 标记为相同的 Identifier，然后用reuseIdentifier 去进行构建，配合不同内容进行批量使用。

当用户滑动列表的时候，如果 reuseIdentifier 不为 nil，UITableView 会自动去调用已经生成好的UITableViewCell 来展示内容。否则每次滑动，UITableView 都会重新生成一个新的 UITableViewCell，这样极其浪费资源，而且容易造成主线程卡顿。

### 3\. 请说明并比较以下协议：UITableViewDelegate，UITableViewDataSource

**关键词：#数据 #UI**

*   一般在 UIViewController 上配置 UITableView，都会用到这 2 个协议，这 2 个协议由当前 UIVIewController 实现。

*   UITableViewDataSource 用来管控 UITableView 的实际数据：例如有多少 section，每个 section 有多少行，每行用哪种 UITableViewCell。其中 numOfRows 和 cellForRowAtIndexPath 这两个方法必须被实现，numOfSections 默认为 1。另外UITableViewDataSource还负责拖拽、修改、删除列表操作，因为这会对数据源进行修改。

*   UITableViewDelegate 用来处理 UITableView 的 UI 和交互：例如设置 UITableView 的 header 和 footer，点击、高亮某个 UITableViewCell 对应的操作。它所有的方法都是可选方法，有默认实现。

### 4\. 请说明并比较以下协议：UICollectionViewDelegate，UICollectionViewDataSource，UICollectionViewDelegateFlowLayout

**关键词：#数据 #UI #Layout**

*   一般在 UIViewController 上配置 UICollectionView，都会用到这 3 个协议，这 3 个协议由当前 UIVIewController 实现。

*   UICollectionViewDataSource 用来管控 UICollectionView 的实际数据：例如有多少 section，每个 section 有多少个 item，每个 item 对应的 UI 如何 。其中 numOfItems 和 cellForItemAtIndexPath 这两个方法必须被实现，numOfSections 默认为 1。另外UICollectionViewDataSource还负责拖拽、修改、删除列表操作，因为这会对数据源进行修改。

*   UICollectionViewDelegate 用来处理交互：例如设置点击、高亮某个 item 对应的操作。它所有的方法都是可选方法，有默认实现。

*   UICollectionViewDelegateFlowLayout 用来处理 UICollectionView 的布局及其行为。比如具体 item 的尺寸大小， item 之间的间距，header 和 footer 的大小和间距，以及 UICollectionView 的滚动方向。这个协议的所有方法也都是可选方法，有默认实现。

如果你正在跳槽或者正准备跳槽不妨动动小手，添加一下咱们的交流群[**931 542 608**](https://jq.qq.com/?_wv=1027&k=cfVxrMAH)来获取一份详细的大厂面试资料为你的跳槽多添一份保障。

## 拓展知识

### 5.代码实现：实现一个 10 行的列表，每行随机显示一个 0 – 100 之间的整数。用户可以删除、移动任何一行，下拉则列表中的数字重新刷新。

**关键词：#UITableViewDataSource #UITableViewDelegate #refreshControl**

本题主要考察 UITableView 最基本的用法：主要涉及 UITableViewDataSource，UITableViewDelegate 这两个协议的使用和 refreshControl 的我们将这道题拆解为 3 个步骤。第一步，实现一个 10 行列表，每行随机显示 0 到 100 之间的整数。示例代码如下：

```
class ViewController: UIViewController {
  @IBOutlet weak var tableView: UITableView!

  fileprivate var nums = [Int]()
  fileprivate let numOfRows = 10
  fileprivate let maxNum = 100

  override func viewDidLoad() {
    super.viewDidLoad()
    configureTableViewDataSource()
  }

  func configureTableViewDataSource() {
    generateRandomNums()
    tableView.reloadData()
  }

 func generateRandomNums() {
    nums.removeAll()

    for _ in 0..<numOfRows {
      nums.append(Int(arc4random_uniform(UInt32(maxNum))))
    }
  }
}

extension ViewController: UITableViewDataSource {
  func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
    return nums.count
  }

  func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
    let cell = UITableViewCell()
    cell.textLabel?.text = "\(nums[indexPath.row])"
    return cell
  }
}

```

第二步，实现下拉刷新的效果。主要就是给 tableView 添加 refreshControl，它能够重新生成随机数并加载 tableView。相关代码如下：

```
override func viewDidLoad() {
  …
    let refreshControl = UIRefreshControl()
  refreshControl.addTarget(self, action: Selector.handleRefresh, for: .valueChanged)
  tableView.addSubview(refreshControl)
}

@objc func handleRefresh() {
  configureTableViewDataSource()
  refreshControl.endRefreshing()
}

extension Selector {
  static let handleRefresh = #selector(ViewController.handleRefresh)
}

```

第三步，实现列表删除和移动功能。主要就是用 UITableViewDelegate 实现 move 和 delete 的操作，相关代码如下：

```
// MARK: - UITableViewDataSource
extension ViewController: UITableViewDataSource {
  func tableView(_ tableView: UITableView, moveRowAt sourceIndexPath: IndexPath, to destinationIndexPath: IndexPath) {

    let moveNum = nums[sourceIndexPath.row]
    nums.remove(at: sourceIndexPath.row)
    nums.insert(moveNum, at: destinationIndexPath.row)
  }

  func tableView(_ tableView: UITableView, commit editingStyle: UITableViewCellEditingStyle, forRowAt indexPath: IndexPath) {

    switch editingStyle {
    case .delete:
      nums.remove(at: indexPath.row)
      tableView.deleteRows(at: [indexPath], with: .automatic)
    default:
      break
    }
  }
}

```

注意，移动和删除操作必须在 tableView 进入编辑模式时才能进行操作。最简单的做法是直接在 viewDidLoad 里设置 tableView 的 isEditing 属性为 true。一般为了用户体验，我们会引入 navigationController，然后在导航栏的右上角添加 edit 按钮来让用户在普通和编辑模式中切换。

### 6\. UICollectionView 中的 Supplementary Views 和 Decoration Views 分别指什么？

**关键词：#补充 #装饰**

*   **Cells，Supplementary Views，Decoration Views 共同构成了整个 UICollectionView 的视图。**Cells 是最基本的、必须用户自己实现并配置的。而 Supplementary Views 和 Decoration Views 有默认实现，主要是用来美化 UICollecctionView 的。

*   **Supplementary Views 是补充视图。**一般用来设置每个 Seciton 的 Header View 或者Footer View，用来标记 Section 的 View。

*   **Decoration Views 是装饰视图。**完全跟数据没有关系的视图，负责给 cell 或者 supplementary Views 添加辅助视图用的，例如给单个 section 或整个 UICollectionView 的背景（background）设置就属于 Decoration Views。

*   Supplementary Views 的布局一般可以在 UICollectionViewFlowLayout 中实现完成。如果要定制化实现 Supplementary Views 和 Decoration Views，那就要实现 UICollectionViewLayout 抽象类中。

下图是 Cells、Supplementary Views、Decoration Views 的说明：
![image](https://upload-images.jianshu.io/upload_images/22877992-2cece327916fba67.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 优化进阶

### 7.UITableViewCell如何根据其内容自动设置其布局？

**关键词：#auto layout #UITableViewAutomaticDimension #estimatedRowHeight

主要有以下三步：

1.  用auto layout对UITableViewCell中所有子视图的位置和大小进行定义；
2.  将`rowHeight`设置为`UITableViewAutomaticDimension`
3.  给`estimatedRowHeight`赋值（随意值，不要太离谱即可）

示例代码：

```
tableView.rowHeight = UITableView.automaticDimension
tableView.estimatedRowHeight = 600

```

### 8.一个tableView滑动很慢，该怎样优化？

**关键词：#渲染 #多线程 #网络传输**

拿到问题第一步要分析原因，列表视图滑动很慢，肯定是 UI 或是数据上出了问题，它们可能是：

*   列表渲染时间较长。可能原因是某些 UI 控件比较复杂，或者图层过多。
*   界面渲染延后。可能原因是大量的操作或耗时的计算阻塞主线程。
*   数据源问题。可能原因是网络请求太慢，不能及时得到相应数据；也有可能是需要更新的数据太多，主线程一时处理不过来。

然后我们针对三个问题，分别去进行优化。

如果你正在跳槽或者正准备跳槽不妨动动小手，添加一下咱们的交流群[**931 542 608**](https://jq.qq.com/?_wv=1027&k=cfVxrMAH)来获取一份详细的大厂面试资料为你的跳槽多添一份保障。

*   第一个问题。首先检查 UITableViewCell 是否进行了复用。对于复杂视图的创建，可以采用惰性加载来推迟创建时间。尽量减少视图层级也是很好的优化方法。Facebook 推出的 ComponentKit 就是很好的解决方案。

*   第二个问题。可以用 GCD 多线程操作将复杂的计算放到后端线程，并进行缓存。例如布局计算或是非 UI 对象的创建和调整就可以如此操作。Linkedin 推出的 LayoutKit 就是很好的例子。

*   第三个问题。建议将网络端数据缓存并存储在手机端，将取得部分数据根据优先级进行顺序渲染，还可以优化服务器端的实现来优化网络请求。

*   另外对于界面渲染和优化其实 Facebook 和 Pinterest 维护的 ASDK 是目前为止功能最全、效果最好、使用最广的第三方解决方案。

### 9.说说实现预加载的方法

**关键词：#网络传输 #无限滚动 #Threshold**

在实际开发中，列表经常需要随着滑动而不停的展示新的内容。在滑动到一定程度后，我们就需要发送网络请求，以获得新的数据。网络请求是一种耗时且昂贵的操作，为了提高用户体验，开发者经常运用预加载的方式提前请求，这样可以在用户滑动到列表最底部之前提前获得最新数据，无需让用户等待。这就是无限滚动列表。

预加载的原理就是，根据当前 UITableView 所在位置，除以目前整个 contentView 的高度，来判断当前位置是否超过 Threshold，如果超过，就发起网络请求，获得数据。以下是示范代码：

```
override func scrollViewDidScroll(_ scrollView: UIScrollView) {
  let current = scrollView.contentOffset.y + scrollView.frame.size.height
  let total = scrollView.contentSize.height
  let ratio = current / total

  if ratio >= threshold {
    requestNewPage()
  }
}

```

以上就是一种最简单的预加载方法。它的缺点十分明显，就是当列表很长时，会出现新加载的页面还没看，应用就会发出另一次请求的情况。

举个例子，假设 Threshold 是 0.7，每个屏幕展示 10 个 cell，每次加载 10 个 cell 的数据，当浏览到第 28 个 cell 时，由于会加载第 40 到第 50 个 cell 的数据，可是我们之前加载的第 30 到第 40 个 cell 的数据还没有被访问。此时发出网络请求，就造成了浪费。

解决方法是将 Threshold 变成一个动态的值，随着数据的增长而增长。示范代码如下：

```
override func scrollViewDidScroll(_ scrollView: UIScrollView) {
    let current = scrollView.contentOffset.y + scrollView.frame.size.height
    let total = scrollView.contentSize.height
    let ratio = current / total

    let needRead = cellsPerPage * threshold + currentPage * cellsPerPage
    let totalCells = cellsPerPage * (currentPage + 1)
    let newThreshold = needRead / totalCells

    if ratio >= newThreshold {
        currentPage += 1
        requestNewPage()
    }
}

```

当然我们还可以进一步优化。例如用惰性加载只处理用户想看到的内容，或是用 ASDK 进行智能预加载。这样可以进一步提高用户体验，并使整个滑动的性能效率最大化。

### 10.如何用 UICollectionView 实现瀑布流界面？

**关键词：#UICollectionViewLayout**

面试中当场实现一个瀑布流，在不允许上网查询的情况下算是十分困难的了。而且代码量很大，所以我们这道题重在分析思路。假设我们已经有了 UICollectionView，现在要做的就是定制化每一个 cell，让他们的高度根据其实际内容设定，从而实现瀑布流。

我们知道要定制化 UICollectionView 的 layout 就一定要使用 UICollectionViewLayout。所以我们首先要做的就是创建一个该抽象类的子类，然后将其设定为当前 UICollectionView 的 Layout。

之后我们需要设计我们的 UICollectionViewLayout 子类，有 4 样东西我们需要一一设定：

*   collectionViewContentSize。由于瀑布流导致的尺寸变化我们重写 contentSize。其中宽度一般情况我们是可以确定的，它取决于每个item的宽度，一行几个 item，以及 contentInset 值。高度我们可以先设定为 0，之后在 prepare() 里进行更新。

*   prepare()。该方法发生在 UICollectionView 数据准备好，但界面还未布局之时。它用于计算各种布局信息，并设定每个 item 的相关属性。这里我们用横纵坐标轴分别进行计算每个 cell 的 xOffset 和 yOffset，然后将其转化为相应的 frame 并缓存起来。

*   layoutAttributesForElements(in:)。prepare() 完成布局之后该方法被调用，它决定了哪些 item 在 CollectionView 给定的区域内可见。我们只要取交集（intersect）即可。

*   layoutAttributesForItem(at:)。该方法需要我们针对每一个 item 设定 layoutAttribute。由于我们在 prepare() 中已经完成相应计算，此时只需返回对应 indexPath 的特定属性即可。

完成这些设定之后，我们发现 UICollectionView 里每个 item 里的高度需要从含有 UICollectionView 的 ViewController 里获得。为了避免循环引用，最好的方法就是在我们的 UICollectionViewLayout 子类中定义一个 protocol，然后让 ViewController 实现这个protocol，来完成高度的获得。Delelgate 这种模式的运用让整个设计的扩展度和灵活度变高。

至此我们就完成了 UICollectionView 实现瀑布流的全过程。以上只是一种比较直接的实现，最复杂的部分在于 prepare() 中运用 xOffset 和 yOffset 构建 LayoutAttributes 的过程，其中含有大量的数学计算。网上对于瀑布流有很多实现，大家不妨借鉴的同时，亲自动手，以加深对 UICollectionView 的理解。
# 推荐👇：

如果你正在跳槽或者正准备跳槽不妨动动小手，添加一下咱们的交流群[**931 542 608**](https://jq.qq.com/?_wv=1027&k=cfVxrMAH)来获取一份详细的大厂面试资料为你的跳槽多添一份保障。

![](https://upload-images.jianshu.io/upload_images/22877992-0bfc037cc50cae7d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
