title: WKWebView
---

`WKWebView` 是苹果在 iOS 8 中引入的新组件，目的是给出一个新的高性能的 `Web View` 解决方案，摆脱过去 `UIWebView ` 的老旧笨重特别是内存占用量巨大的问题。

苹果将 `UIWebViewDelegate` 与 `UIWebView` 重构成了 [14 个类和 3 个协议][14 个类和 3 个协议]，引入了不少新的功能和接口，这可以在一定程度上看做苹果对其封锁 `Web View` 内核的行为作出的补偿：既然你们都说 `UIWebView` 太渣，那我就造一个不渣的给你们用呗~~ 众所周知，连 Chrome 的 iOS 版用的也是 `UIWebView` 的内核。

`WKWebView` 有以下几大主要进步：

- 浏览器内核渲染进程提取出 App，由系统进行统一管理，这减少了相当一部分的性能损失。
- js 可以直接使用已经事先注入 js runtime 的 js 接口给 Native 层传值，不必再通过苦逼的 iframe 制造页面刷新再解析自定义协议的奇怪方式。
- 支持高达 60 fps 的滚动刷新率，内置了手势探测。


###### <font color = #1ba856 > 开始使用 WKWebView </font>

创建一个名为 BuildYourOwnHybridDevelopmentFramework 的 Single View Application 项目。在 `ViewController` 顶部引入 `WebKit`：

`import WebKit`

之后创建一个 WKWebView 类型的成员变量，并对其进行初始化，加入到页面上：

```swift
var wk: WKWebView!
override func viewDidLoad() {
    super.viewDidLoad()
}
override func didReceiveMemoryWarning() {
    super.didReceiveMemoryWarning()
    // Dispose of any resources that can be recreated.
}


override func viewDidAppear(animated: Bool) {
    super.viewDidAppear(animated)


    self.wk = WKWebView(frame: self.view.frame)
    self.wk.loadRequest(NSURLRequest(URL: NSURL(string: "http://www.baidu.com/")!))
    self.view.addSubview(self.wk)
}
```

遭遇 BUG

如果你用的是 Xcode 7，这时候你应该得到了一块雪白雪白的屏幕，我们遭遇了 bug。这是因为 iOS 9 SDK 中默认不再支持访问非 HTTPS 的地址，在 `info.plist` 加入:

```plist
<key>NSAppTransportSecurity</key>
<dict>
	<key>NSAllowsArbitraryLoads</key>
	<true/>
</dict>
```

再次运行项目，搞定！

查看效果

<a href = "/images/WKWebView/WKWebView_0.png"><img src = "/images/WKWebView/WKWebView_0.png" width = 365 height = 690 alt = "WKWebView_0.png"></a>


###### <font color = #1ba856 > 简易错误处理 WKWebView </font>

为了更方便地在开发中调试问题，我们需要处理一下页面加载失败的事件。

加入代理：

```swift
class ViewController: UIViewController, WKNavigationDelegate {
... ...
```

设置代理对象为 self：

```swift
self.wk.navigationDelegate = self
```

处理加载失败事件

我们可以使用以下方式让 View Controller 更优雅，更短小精悍：

```swift
private typealias wkNavigationDelegate = ViewController
extension wkNavigationDelegate {
    func webView(webView: WKWebView, didFailNavigation navigation: WKNavigation!,
     withError error: NSError) {
        NSLog(error.debugDescription)
    }
    func webView(webView: WKWebView, didFailProvisionalNavigation 
    navigation: WKNavigation!, withError error: NSError) {
        NSLog(error.debugDescription)
    }
}
```

查看效果

我们把 网址从 <http://www.baidu.com/> 改成 <http://www.baidu/>，运行，得到错误：

<a href = "/images/WKWebView/WKWebView_1.png"><img src = "/images/WKWebView/WKWebView_1.png" width = 545 height = 114 alt = "WKWebView_1.png"></a>

搞定！

###### <font color = #1ba856 > 显示弹窗 WKWebView </font>



在 UIWebView 里，js 的 alert() 弹窗会自动以系统弹窗的形式展示，但是 WKWebview 把这个接口也暴露给了我们，让我们自己 handle js 传来的 alert()。下面我们将自己写代码 handle 住这个事件，并展示为系统弹窗。

加入代理：

```swift
class ViewController: UIViewController, WKNavigationDelegate, WKUIDelegate {
... ...
```

设置代理对象为 self：

```swift
self.wk.UIDelegate = self
```


处理 alert() 事件


```swift
private typealias wkUIDelegate = ViewController
extension wkUIDelegate {
    func webView(webView: WKWebView, runJavaScriptAlertPanelWithMessage message: String,
     initiatedByFrame frame: WKFrameInfo, completionHandler: () -> Void) {
        let ac = UIAlertController(title: webView.title, message: message, preferredStyle:
         UIAlertControllerStyle.Alert)
        ac.addAction(UIAlertAction(title: "OK", style: UIAlertActionStyle.Cancel, handler:
         { (aa) -> Void in
            completionHandler()
        }))
        self.presentViewController(ac, animated: true, completion: nil)
    }
}
```

执行 alert()

把网址改为正确的 <http://www.baidu.com/>，运行项目。然后使用 Safari 自带的 Web View 调试工具执行 alert() 函数：

<a href = "/images/WKWebView/WKWebView_2.png"><img src = "/images/WKWebView/WKWebView_2.png" width = 455 height = 262 alt = "WKWebView_2.png"></a>

查看效果

<a href = "/images/WKWebView/WKWebView_3.png"><img src = "/images/WKWebView/WKWebView_3.png" width = 546 height = 410 alt = "WKWebView_3.png"></a>

OK

[14 个类和 3 个协议]: https://developer.apple.com/library/ios/documentation/Cocoa/Reference/WebKit/ObjC_classic/index.html