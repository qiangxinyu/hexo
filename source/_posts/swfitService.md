title: 使用 Swift 搭建服务器
---

[原文地址](http://www.cocoachina.com/swift/20160318/15712.html)


自从苹果官方发布了一个 Swift 的 Linux 开源版本之后，服务端 Swift 终于迎来了一个令人激动的前景。我的好奇心终于无法克制，是时候尝试一下服务端 Swift 了！

除了用过几个 Baas 以外，我没有任何后端编程经验，但幸运的是开源社区已经提供了现成的框架。我试了一下 [Tanner Nelson][Tanner Nelson] 推荐的 [Vapor][Vapor] 框架。它的使用非常简单，非常适合我当前的任务，在这篇文档中还会使用到 Heroku。我决定使用 Heroku 的原因是我们的后端团队在使用它，它对于前端来说非常友好。

写到这里的时候，为了解决 Heroku 框架运行中的几个小问题，我专门提交了一个 [pull request][pull request] 。如果代码还没被合并的话，请设置你的包管理器从 [这里][这里] 下载。

### 安装

要继续本教程，首先，你需要一个 [Heroku][Heroku] 账号 ，并安装好 [Swift Development Snapshot][Swift Development Snapshot] 。写到这里的时候，在它的正式版中还未包含 swift 包管理器。因此为了使用这个工具，你必须下载开发版的 snapshot。

#### 开始

我们的目标是创建一个简单的 Swift 服务器并运行在 Heroku 上。这不需要在 linux 环境下进行，就像是你在使用本地服务器。你只消创建一个本地的 Xcode 项目，对项目进行一些设置，然后就可以在 [Swift 包管理器][Swift 包管理器] 中运行它。整过过程分为 4 个步骤：

####### 将 main.swift 拷贝根目录下的 Sources 目录

<a href = "/images/swfitService/swiftService1.png"><img src = "/images/swfitService/swiftService1.png" width = 720 height = 598 alt = "swiftService1.png"/></a>


####### 创建一个 Package.swift 文件

<a href = "/images/swfitService/swiftService2.png"><img src = "/images/swfitService/swiftService2.png" width = 720 height = 218 alt = "swiftService2.png"/></a>

####### 将 .build 目录添加到 import paths 

要使用自动补全和语法加亮功能，需要将 Swift 包管理器的 build 目录提交到 import paths 中。注意，在 import paths 的 Debug 中设置的 debug 目录，而 Release 项则输入 release 目录。

<a href = "/images/swfitService/swiftService3.png"><img src = "/images/swfitService/swiftService3.png" width = 720 height = 334 alt = "swiftService3.png"/></a>

####### 用 toolchain 运行 Xcode

如果你使用 Xcode 7.3，你可以用 Xcode > Toolchains 菜单开启一个Xcode 实例，用于打开 swift snapshot。因为我们不能在 Xcode 中进行编译，我们只能以命令行的方式进行编译。

<a href = "/images/swfitService/swiftServie4.png"><img src = "/images/swfitService/swiftService4.png" width = 594 height = 16 alt = "swiftService4.png"/></a>

#### 编写服务器

令我高兴的是，为了进行概念验证，我需要编写的代码其实只有寥寥数行。我启动和运行服务器的代码甚至不到 10 行。

<a href = "/images/swfitService/swiftServie5.png"><img src = "/images/swfitService/swiftService5.png" width = 720 height = 239 alt = "swiftService5.png"/></a>

要启动服务器，只需在终端中输入一句命令，：

<a href = "/images/swfitService/swiftService6.png"><img src = "/images/swfitService/swiftService6.png" width = 720 height = 126 alt = "swiftService6.png"/></a>

好了，让我们打开浏览器。我的浏览器安装了 json 插件，你的画面或许会有不同。

<a href = "/images/swfitService/swiftService7.png"><img src = "/images/swfitService/swiftService7.png" width = 556 height = 240 alt = "swiftService7.png"/></a>

#### 迁移到云上

服务器在本地顺利运行起来了，但如果放到云端则更好。我迫不及待地想将 App 在云上跑起来。对于我来说这是一个全新的挑战，幸运的是，我得到了 [Vincent Toms][Vincent Toms] 的悉心指导。

Heroku 的安装是一件非常愉悦的体验，几分钟后我就创建了一个 Heroku App，接下来我就要上传我的项目了。

####### 出错啦

这只是今天的诸多错误中的一个。我已经预计到事情不可能一帆风顺，因此我查看了 Vapor 的文档，最终发现问题出在所谓的 [buildpacks][buildpacks] 上。Heorku 提供了一些标准的 buildpacks，但完全没有针对 Swift 的 buildpacks。无奈之下求救于开源社区，刚好看到 Kyle Fuller 的 [buildpack][buildpack] 。集成它就简单得多了。

<a href = "/images/swfitService/swiftService8.png"><img src = "/images/swfitService/swiftService8.png" width = 720 height = 85 alt = "swiftService8.png"/></a>

用这个 buildpack 启动后，App 成功加载，接下来就是访问它的 URL。

####### 再次出错

<a href = "/images/swfitService/swiftService9.png"><img src = "/images/swfitService/swiftService9.png" width = 720 height = 147 alt = "swiftService9.png"/></a>

事情不会那么顺利的，是吧？经过 google 一番，仔细查看了一些例子，我发现我还差一个 Procfile。浏览一下这个文件的内容，你就能明白这个文件是干什么用的了。

<a href = "/images/swfitService/swiftService10.png"><img src = "/images/swfitService/swiftService10.png" width = 664 height = 170 alt = "swiftService10.png"/></a>


buildpack 创建了可执行文件，但 Heroku 并不知道。通过 Procfile，我们告诉 Heroku 去运行 SwiftServerIO 可执行文件。上传这个 Procfile。

####### 仍然出错

Heroku 编译的 2 分支仿佛变得无比漫长。我重新打开了浏览器，发现仍然报错。


<a href = "/images/swfitService/swiftService11.png"><img src = "/images/swfitService/swiftService11.png" width = 720 height = 153 alt = "swiftService11.png"/></a>

我以为 Heroku 可能还未启动完成（实际不是），因此我又等了一小会，终于发现出错了。可执行文件生成了， process file 也就绪，一定是别的什么地方出问题了。再次 google，一直到我最终发现我需要设置 App 的规模([scale up][scale up])。这要使用到 Heroku 的 toolbelt 中的一个简单命令。

heroku ps:scale web=1



Heroku 在免费情况下只有一个 dyno（Heroku 计费单位，10~50 个请求/秒）。但对于我们的简单服务器来说，这也够了。因此，在我们将 scale web 设置为 1 个 dyno 之后，再次用浏览器查看。

成功了！

<a href = "/images/swfitService/swiftService13.png"><img src = "/images/swfitService/swiftService13.png" width = 168 height = 166 alt = "swiftService13.png" /></a>

成功了！服务器启动，出现了万能的 hello world！经过一番欢呼雀跃之后，让我们真正来问一声好吧！

####### 响应请求

在 main.swfit 文件中添加一小段代码，让服务器在问好的同时能够因人而异。就微微偷一下懒，新加一个路由，让服务器根据输入输出不同的问候语。


<a href = "/images/swfitService/swiftService14.png"><img src = "/images/swfitService/swiftService14.png" width = 720 height = 296 alt = "swiftService14.png" /></a>


一切正常，但根据一般规律，我仍然做好了出错的心理准备。提交修改，push 代码到 Heroku。

####### Say Hello!

<a href = "/images/swfitService/swiftService15.png"><img src = "/images/swfitService/swiftService15.png" width = 413 height = 151 alt = "swiftService15.png" /></a>

大约一份多钟的编译之后，在浏览器中访问 URL，服务器返回了问候语。[你可以在这里查看效果][你可以在这里查看效果] 。

接下来是什么？

可以说，服务端 Swift 的今天离不开社区强大支持。对于我来说，能够从云端获取 JSON 是一个令人兴奋的开始，我已经迫不及待地想看看接下来还会发生什么。

当然在此之前，我不得不和我在 [Intrepid Pursuits][Intrepid Pursuits] 的同事们一起，继续编写一个有一个 iOS App。如果你想了解我的最新动态，请在访问我的 [Github][Github] 或者 [Twitter][Twitter]。

[服务端 Swift][服务端 Swift] 

附言

在 [这里][这里] 下载源代码。

在 [Journal][Journal] 文件夹中，是 step-by-step 指南。


[Tanner Nelson]: https://twitter.com/tanner0101
[Vapor]: https://github.com/qutheory/vapor
[pull request]: https://github.com/qutheory/vapor/pull/25
[这里]: https://github.com/loganwright/vapor
[Heroku]: https://signup.heroku.com/?c=70130000001x9jFAAQ
[Swift Development Snapshot]: https://swift.org/download/
[Swift 包管理器]: https://swift.org/package-manager/
[Vincent Toms]: https://twitter.com/vincenttoms
[buildpacks]: https://devcenter.heroku.com/articles/buildpacks
[buildpack]: https://github.com/kylef/heroku-buildpack-swift
[scale up]: https://devcenter.heroku.com/articles/scaling#cli
[你可以在这里查看效果]: http://swift-server-io.herokuapp.com/hello/reader
[Intrepid Pursuits]: http://intrepid.io/
[Github]: https://github.com/loganwright
[Twitter]: https://twitter.com/logmaestro
[服务端 Swift]: http://www.swiftserver.io/hello/server-side-swift
[这里]: http://github.com/loganwright/swift-server-io
[Journal]: https://github.com/LoganWright/swift-server-io/tree/master/Journal