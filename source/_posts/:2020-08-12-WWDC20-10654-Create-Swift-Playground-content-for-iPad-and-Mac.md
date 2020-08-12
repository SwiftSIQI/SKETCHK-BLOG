---
title: WWDC20 10654 - Create Swift Playground content for iPad and Mac
comments: true
date: 2020-08-12 12:02:26
updated:
tags:
  - WWDC
  - Swift
  - Playground
categories:
  - Swift
---

这是一个麻雀虽小，但五脏俱全的 Session，在短短的 8 分钟里，不仅包含了功能介绍，也有相应的代码演示。

不过整个 session 都是围绕在如何利用 Swift Playground 为 iPad 和 Mac 创建优质内容而展开的，所以都是很细节和琐碎的点。

<!-- more -->

## 引子

这是一个麻雀虽小，但五脏俱全的 Session，在短短的 8 分钟里，不仅包含了功能介绍，也有相应的代码演示。

不过整个 session 都是围绕在如何利用 Swift Playground 为 iPad 和 Mac 创建优质内容而展开的，所以都是很细节和琐碎的点。

下面的脑图展示了这个 session 的所有知识点，方便你复习和加深理解：

![01.png](01.png)

## Swift Playground 在界面上的改变

如果使用过 Swift Playground ，那么你应该能回忆起，在 iPad 上是可以看到代码补全功能提供的 token。

![IMAGE.png](02.jpg)

为了适配 Mac 平台，Apple 团队为 Swift Playground 的代码补全功能做了进一步的优化。

现在，在 Mac 上不仅可以看到这些 token，还可以看到相应的帮助文档

![IMAGE.png](03.jpg)

除此之外，帮助文档没有对语言类型做限制，我们完全可以将文档的语言类型变为使用者的本地语言，通过这样的方式来降低他们的理解成本

如果想创建 API 的帮助文档，可以使用 3 个 `/` 来声明，除了描述方法外，也可以为参数添加说明，下图就是一个例子

![IMAGE.png](04.jpg)

在 iPad 上的话，可以通过 Quick Help 的弹窗或者代码补全的提示条来展示相应的代码说明，它的效果如下所示

![IMAGE.png](05.jpg)

## 针对不同平台展示定制化内容的能力

在 Playground book 格式的文件中，提供了两个新的 API 用来区分不同平台的定制化能力，它们分别是 `supportedDevices` 和 `requiredCapbilities`

> 这里要记住一点，Playground 和 Playground Book 是两个不一样的东西，在 Xcode 或者 Swift Playground 里是无法直接触发 Playground Book 的 target 或者 Project 的，需要到官方下载 [Swift Playgrounds Author Template](https://developer.apple.com/download/more/?=Swift%20Playgrounds%20Author%20Template), 如果你想更进一步了解这两种格式的区别，建议阅读 [官方的说明文档](https://developer.apple.com/documentation/swift_playgrounds/creating_and_running_a_playground_book) 和 [Using Swift Playgrounds & Playground Books](https://medium.com/@barbulescualex/using-swift-playgrounds-playground-books-87c2707be2b5) 

`supportedDevices` 这个 key 是用来区分不同平台的，例如是 iPad 还是 Mac，而 `requiredCapbilities` 则是用来明确平台能力的，例如需要 AR 能力（Mac 拥有，而 iPad 没有），需要 WIFI 能力（两个平台都具备）。

它们的设定是在 manifest 文件和 feed json 文件中进行的，需要设定相应的 key 值，具体的示例可以参考下面的两张图

![IMAGE.png](06.jpg)

![IMAGE.png](07.jpg)

关于 `requiredCapbilities` 支持的键值，可以查看 UIRequiredDeviceCapabilities 的 API 说明，这里放一个[传送门](https://developer.apple.com/documentation/bundleresources/information_property_list/uirequireddevicecapabilities)

已经知道了这些能力，那么在代码里如何使用呢，下面就是一段示例代码，我们可以根据不同的平台来编译不同的代码

![IMAGE.png](08.jpg)

为了让 Playground Book 的体验更好，我们肯定希望开发者去适配更多的能力，例如 AR，重力感应，GPS等，但如果用户一开始是在 Mac 上使用的话，可能会无法看到这些功能，例如你的逻辑是只有检测到具备这个功能才显示某个按钮，如果不具备则隐藏。

那么我们要怎么做才能暗示用户呢，Apple 的建议是在语言的描述上做好引导，例如在不同的平台上，对操作的描述是这样的

![IMAGE.png](09.jpg)

如果是一个通用的功能，说辞可以选择 tap 或者 select

![IMAGE.png](10.jpg)

## 适配系统的设置规则

在开发实际内容的过程中，我们要考虑到系统的设置，例如主色调，副色调和暗色模式等！

![IMAGE.png](11.jpg)

> 这张图单纯就是觉得主讲的妹子蛮可爱的，想骗你们看 session.....

Apple 提供的原生组件会自动响应这些变化，而开发者自定义的组件需要适配才能实现同样的功能，这一点需要开发者做好，涉及的资源建议放到 Asset catalog 中

除了上面的问题外，在开发跨平台的内容时，还需要关注 live view 的 Safe Area。

例如下面的视图中，左侧是在 ipad 上运行的 Live View, 右侧则是在 Mac 上运行的效果，仔细观察的话，会发现右侧的 Live view 在右上角多出有一些按钮，这就会导致两个 Live View 的 Safe Area 的区域不一致

![IMAGE.png](12.jpg)

之前开发者会使用 `liveViewSafeAreaGuide` 这个特殊的 API 获取相关值，现在终于可以直接使用 `safeAreaLayoutGuide` 这个更通用，更好理解的 API 了

![IMAGE.png](13.jpg)

## 总结

这篇 Session 的内容就到此结束了，我们复习一下整个文章的内容，大体有 3 部分

1. Swift Playground 针对不同平台做了 UI 上的调整，使其更符合每个平台的使用习惯
2. 提供了更加强大的定制化能力，不仅包括了平台定制，还包括了能力定制
3. 开发者需要做好适配系统设置的工作，Apple 也为开发者提供了一些便捷能力

另外，如果各位有时间的话，强烈推荐观看一下 Explore Swan's Quest 里的四个 Session，加起来的全部观影时间也就 35 分钟，但看完以后，你会发现原来 Swift Playground 竟然还有这么多有意思的用法，感觉像打开了一个新的世界一样。

相信我，这绝对不是骗你。

![IMAGE.png](14.jpg)
