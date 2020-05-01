---
title: 一次关于 Swift 在 iOS 生态圈里的现状调研
comments: true
date: 2020-04-30 16:53:06
updated:
tags:
  - Swift
categories:
  - Swift
---

这是一次调研性质的文章，通过它让我们来看看 Swift 在国内的现状到底是什么样子的？

<!-- more -->
## Swift 的发展历程

### 概述

通过官网的 [Document Revision History](https://docs.swift.org/swift-book/RevisionHistory/RevisionHistory.html) 和 [Xcode Release Notes](https://developer.apple.com/documentation/xcode_release_notes) 梳理了一下 Swift 的发展历程和重大事件。

| Swift 版本 | Xcode 版本| 发布时间 | 重大事件 |
|  --------- | --------- | -------- | -------- |
| Swift 1.0 ~ 1.2 | 6.x   | 2014 | 语⾔发布 |
| Swift 2.0 ~ 2.2.1 | 7.x | 2015 | 对协议，泛型能力进一步扩展，开始支持 Linux，随后出现了以 Swift 语言为核心的后端框架 Perfect，Vapor，Kitura |
| Swift 3.0 ~ 3.3.1 | 8.x | 2016 | 发布了 Swift Package Manager，同时以 GCD，Core Graphics 为代表的基础库 API 风格发生了大幅度转变，摆脱了 Objective-C 时代的烙印 |
| Swift 4.0 ~ 4.1.3 | 9.x | 2017 |  在整体的语法，使用和理念上基本定型，提出了 Codable 协议，同时 Xcode 的 Swift Syntax Mirgration 的最低版本固定为 4  |
| Swift 4.2 ~ 4.2.4 | 10.x | 2018 | Swift社区从邮件列表转向论坛，语言小幅升级，主要是功能完善，性能提升，同年 Swift for TensorFlow 发布并开源 |
| Swift 5.0 ~ 5.0.3 | 10.2.x | 2019 | ABI 稳定，iOS 12 开始内置 Swift 运行时 |
| Swift 5.1 ~ 5.2 | 11.x | 2020 | 新增 Property Wrapper ，Opaque Type 等新的语法功能，同年 WWDC 上，Apple 发布了 SwiftUI，Combine，Catalyst 等 Swift 语言的专属 SDK |
| Swift 5.3 |  |  | 质量和性能增强，增加对 Windows 和其他 Linux 发行版的支持。 |

结合着自己的 Swift 学习经历，不难发现：

在 Swift 4 之前，由于语言整体还没定型，确实存在着发一个新版本，学一门新语言的情况，但在 Swift 4 之后，Swift 变化变得收敛了许多，不过也出现了入门容易，精通难的情况，毕竟光 Swift 的语法糖数量就快赶上了 C++ 了。

### 语言排行榜

Swift 语言从诞生之日开始，就一直存在各种各样的争议：一方面的焦点在于 Swift 的应用领域还是集中在 Apple 生态下，让人觉得不够大气，毕竟新时代的语言就是要全能，另一方面的焦点就是 Swift 语言的变化太快，每个版本都是是全新的感觉，这让开发者意识到，东西虽好，但代码还是要一个个的自己改的。

不过随着时间的推移，Swift 在后端，人工智能，物联网上的解决方案和应用场景不断出现，它已经远远不在是一个只能在 Apple 生态下运行的语言，关于这个非常推荐看看 Onevcat 在 GMTC 2019 上的分享：[在分歧中发展——2019，我们能用 Swift 做什么](https://gmtc.infoq.cn/2019/beijing/presentation/1802)

ABI 在 Swift 5.0 的时候也终于稳定了，虽然 ABI 稳定是使用 binary 发布框架的必要非充分条件，但都 ABI了，module stability 也不会太远了，这些信号都让开发者的信心不断增强。

为了验证这一点，我们也可以从编程语言排行榜 [TIOBE](https://www.tiobe.com/tiobe-index/) 和 [PYPL](http://pypl.github.io/PYPL.html) 里看出一些端倪。

下图为 TIOBE 2020 年 四月排行榜，Swift 的排名在 11 名，Objective-C  的排名在 17 名，份额差距在 0.6% 左右：

![TIOBE 2020 年 四月排行榜](01.jpg)

下图为 PYPL 2020 年 四月排行榜，Swift 的排名在 9 名，Objective-C  的排名在 8 名，份额差距在 0.17% 左右：

![PYPL 2020 年 四月排行榜](02.jpg)

不论哪种排名，我们应该都可以得出这样一个结论：相比于 Objective-C 的下降趋势，Swift 的未来会更让人期待。

## 社区活跃度

观察一个编程语言活跃度的最好地方就是 Github，通过 Pull Requests ，Issues，Pushes，Stars 的情况，我们就可以大致了解到它的情况。

恰好 [Githut](https://madnight.github.io/githut/#/stars/2020/1) 提供了这样的能力，通过观察 2020 年第一季度的数据，我们可以清楚的观察到 Swift ，Objective-C 等语言在社区的活跃度。

### 整体趋势图

下面四张图的 Y 轴分别代表了 Pull Requests ，Issues，Pushes，Stars 的数量，蓝色的线代表 Objective-C ，浅橙色的线代表 Swift，深橙色的线代表 Kotlin。

![Pull Requests](03.jpg)

![Issues](04.jpg)

![Pushes](05.jpg)

![Stars](06.jpg)

我们可以发现，从 2016 年开始，Swift 的数据已经超过了 Objective-C，现在有一种取而代之的快速发展趋势，这说明社区中更多的开发者习惯把精力放在 Swift 上，而不是放在 Objective-C 上。

这就给相应的开发者一个提醒，如果你坚持使用 Objective-C，那么你可能会面临一个风险，你所依赖的第三方开发库，他们的原作者很有可能已经不愿意去维护他们了。

同时，我们也可以发现 Swift 和 Kotlin 作为端上的新语言，它们的发展趋势是稳步上升的，这代表它们在社区是受欢迎的，那这也意味着掌握这些新技术会更容易与其他程序员进行交流和沟通，至于未来能不能完全取代现有语言的份额，还需要时间来证明。

### 个体情况对比

如果关注过一些 iOS 相关的 Newsletters，我们应该都可以感受到，相比于 Swift 的第三方开源库，同类型的 Objective-C 开源库明显处于劣势，不论是新增数量，还是活跃度；甚至有些 Objective-C 的库已经被开发者废弃，转而使用 Swift 重构。

下面是两种语言的两个常见方向的第三方开源库的数据对比，我们明显可以看出 Objective-C 项目的活力在下降，甚至出现了 2 年没有更新的状态。

| 名称 | Massonry | SnapKit | Pop | Hero  |
| -----| ---------| --------| ----| ----- |
| 语言                | Objective-C | Swift | Objective-C | Swift |
| Stars               | 17.7K | 16.3K | 19.8K | 18.3K |
| Opening issue       | 108 | 64 | 44 | 167 |
| Opening Pull Request| 19 | 16 | 14 | 14 |
| Latest Commit       | 2017.09.28 | 2019.11.27 | 2018.10.12 | 2020.04.26 |
| Latest release      | 2017.09 | 2019.08 | 2018.10.12 | 2019.10.29 |

对于那些还在持续更新的 Objective-C 库，它的更新频率又是什么样子的呢？我们拿 AFNetworking 和 Alamofire 做一个比对：

| AFNetworking 版本 | 时间点 | 间隔时间 | Alamofire 版本 | 时间点 | 间隔时间 | 
| ------------------| -------| ---------| -------------- | ------ | -------- |
| 4.0.1 | 2020.04.21 | 1 个月 | 5.1.0 | 2020.04.05 |0.5 个月
| 4.0.0 | 2020.03.31 | 12 个月 | 5.0.5 | 2020.03.24 | 0.2 个月
| 2.7.0 | 2019.02.13 | 7 个月 | 5.0.4 | 2020.03.16 | 0.1 个月
| 3.2.1 | 2018.05.05 | 5 个月 | 5.0.3 | 2020.03.15 | 0.5 个月
| 3.2.0 | 2017.12.16 | 25 个月 | 5.0.2 | 2020.02.24 | 0.1 个月
| 2.6.3 | 2015.11.11 | | 5.0.1 | 2020.02.23 |

虽然 AFNetworking 还在更新，但其更新的周期实在是让人摸不着头脑，相比于 Alamofire 稳定的更新频率，你到底愿意用哪个呢？

举个实际的例子，比如大家都是做 HTTP 请求的，3.0 协议已经发出，从社区的反馈来看，未来的开发者更愿意把精力放在 Swift 上面，现在 SSL Certificate Verify 的验证，是可以把证书链从上至下全部验证一遍，这在 Alamofire 里已经支持的非常好，而 AFNetworking 在此领域目前是缺失的。

这就有可能在未来的某个时间点，工程所依赖的，第三方的，不可替换的，核心的开发框架有严重的问题或者功能缺失时，项目的进度会受影响，作为开发者会陷入进退两难的状况。

## Apple 的 SDK

在 Apple 的 [Apple Developer Documentation列表](https://developer.apple.com/documentation) 中，我们发现共有 196 个条目，其中 Swift 独占的 SDK 共计 10 个，Objective-C 独占的 SDK 共计 14 个。

| 维度 | 个数 | SDK名称 |
| ------------------| --------| ----------------  | 
| Swift 独占 | 10      | Swift(Swift Standard Library)，Combine，SwiftUI，RealityKit，CareKit，Create ML(Create ML， Create MLUI)，Playground Support，PlaygroundBluetooth，Apple CryptoKit，Swift Packages(Swift Package Manager)              | 
| Objective-C 独占 | 14      | QTKit (macOS 专属)，Professional Video Applications(FxPlug，macOS 专属)，xcselect （macOS 专属），DarwinNotify，DriverKit（macOS 专属），EndpointSecurity（macOS 专属），HIDDriverKit（macOS 专属），IOUSBHost（macOS 专属），Kernel（macOS 专属），NetworkingDriverKit（硬件驱动相关），PCIDriverKit（硬件驱动相关），SerialDriverKit（硬件驱动相关），USBDriverKit（硬件驱动相关），USBSerialDriverKit（硬件驱动相关）           |

虽然乍一看 Objective-C 独占的 SDK 较多，但这些 SDK 主要是面向 macOS 和硬件驱动方向的，对 iOS 开发本身没有任何影响。

而 Swift 独占的 SDK 就完全不一样了， Apple 这些年主推的技术方向，例如 AR，AI，Health，SwiftUI，Combine 等一系列 SDK 都只有 Swift 的版本了，这无疑在暗示开发者：来吧，用就用 Swift 吧！

## Swift 在国内外 iOS 客户端的使用现状

说了这么多，Swift 在 iOS 生态里到底用的怎么样？最直接的办法就是看看它在 Apple Store 里的占有率，这里我们会考察 2019 和 2020 年的情况。

### 2019 年

很庆幸，这部分工作，淘宝团队已经做过了！

相关的内容在 [从探索到落地，手淘引入 Swift “历险记” 2019](https://mp.weixin.qq.com/s/oHGkoGzhMs-l8TX6t0831w) 一文中已经披露，这里借用它们原话：

> 我们通过爬虫分析国内外 APP Store 排行榜 Top 1000 的APP，通过文件扫描分析得到结论。
>
> * 国内使用 Swift 的 APP 约占比 22%，美区使用 Swift 的 APP 约占比 78%，其中美区剩余没有使用 Swift 的 APP 大部分来自中国地区本地化的产品，如抖音，快手等，可以得出一个结论，国内还是小众的 Swift，在国外已经是现状。
> * Github/Stack Over Flow 社区等 Objective-C 开源库和问题提问已经基本停滞，未来我们在落地新技术，Objective-C 可能已经是最坏的打算，加之 WWDC 17年以来，苹果不再提供 Objective-C 的示例，组内同学也多次遇见 Objective-C Bug 去社区提问，毫无热度的情况。
> * 苹果在 WWDC19 年发布了 4 个 Pure Swift 框架，无法简单的被 Objective-C 混编。
>
> 未来我们极有可能因为苹果的强制推进风格和社区文化的落后产生技术踏空，无法迅速响应业务，甚至无法招聘到会使用 Objective-C 的工程师。

![IMAGE](07.jpg)

通过了解，淘宝的数据来源是七麦数据提供的，日期为 2019 年 02 月 19 日，[国内排行榜传送门2019版](https://www.qimai.cn/rank/index/brand/free/device/iphone/country/cn/genre/5000/date/2019-02-19)和[国外排行榜传送门2019 版](https://www.qimai.cn/rank/index/brand/free/device/iphone/country/us/genre/5000/date/2019-02-19)

### 2020 年

现在回到 2020 年，Swift 在国内外的应用情况又发生了什么变化呢？

为了得到这个问题的答案，我也打算对国内外免费榜里的 APP 进行扫描，先不说 1000 个 ipa 的下载工作量有多大，256G 的硬盘估计也装不下这么多 APP，索性就弄前 102 名吧！

> 本来是想做前 100 的，只是因为当时下载国内 ipa 的时候多下载了 2 个，索性本着样本越多越接近真实情况的道理就把国外的数据量也放到了 102 个，并没有什么特别原因。

扫描的原理借鉴了 [如何检测 iOS 应用程序是否使用 Swift？](https://mp.weixin.qq.com/s/vF_oOWFLimlyRi4mZpgpeQ) 里的思路，相应的工具已经放在 Github 供大家使用，点击[传送门](https://github.com/ZRTransmitter/SwiftAppAnalyzer)即可，这里要感谢一下好基友 [@ForelaxX](https://github.com/ForelaxX) 制作了这个脚本工具。

App 排行榜的数据来源是七麦数据提供的，日期为 2020 年 4 月 27 日，[国内排行榜传送门2020版](https://www.qimai.cn/rank/index/brand/all/device/iphone/country/cn/genre/5000/date/2020-04-27)和[国外排行榜传送门2020版](https://www.qimai.cn/rank/index/brand/all/genre/5000/device/iphone/country/us/date/2020-04-27)

通过扫描这 102 个 App，最终发现国内的 Swift 占比为 30.4%（31/102），国外的 Swift 占比 82.3%（84/102），相比于 2019 年的数据，国内的 Swift 应用增长了 10% 左右，国外的 Swift 应用增长了 5% 左右。

![IMAGE](08.jpg)

下图是结合 2019 年和 2020 年的百分比趋势变化图

![IMAGE](09.jpg)

下面是扫描的详细结果：

| 国内 App 版本     | 是否使用 Swift | 国外 App 名称 | 是否使用 Swift | 
| ------------------| ---------------| ------------- | -------------- |
| 拼多多             |   NO  | Zoom                    | NO          |
| 腾讯会议           |   NO  | TikTok                  | NO          | 
| 钉钉               |   NO  | Houseparty              | YES         | 
| 个人所得税         |   NO  | Youtube                 | NO          | 
| 剪映               |   YES | Instagram               | YES         | 
| 交管 12123         |   NO  | Facebook                | NO          |
| 抖音               |   NO  | Messenger               | NO          |
| 微信               |   YES | Amazon                  | NO          |
| 微视               |   YES | Cash App                | YES         |
| 支付宝             |   NO  | DoorDash                | YES         |
| QQ                 |   NO  | American Idol           | YES         |
| 快手极速版         |   NO  | Netflix                 | YES         |
| 手机淘宝           |   YES | Snapchat                | YES         |
| 淘宝特价版         |   NO  | PREQUEL                 | YES         |
| 企业微信           |   NO  | Gmail                   | YES         |
| 快手               |   NO  | Wish                    | YES         |
| 和平营地           |   NO  | Hulu                    | YES         |
| QQ 音乐            |   NO  | Disney+                 | YES         |
| 百度               |   YES | Shop                    | YES         |
| 闲鱼               |   NO  | Pinterest               | NO          |
| 网易云音乐         |   NO  | Amazon Prime Video      | YES         |
| 腾讯视频           |   NO  | PayPal                  | YES         |
| 高德地图           |   NO  | Spotify                 | YES         |
| 酷狗音乐           |   NO  | Google Duo              | YES         |
| 小红书             |   YES | Discord                 | YES         |
| 美团               |   NO  | Venmo                   | YES         |
| 京东               |   NO  | Wayfair                 | YES         |
| 夸克               |   NO  | Walmart                 | YES         |
| 哔哩哔哩           |   NO  | Twitter                 | YES         |
| 瑞幸咖啡           |   YES | WhatsApp                | YES         |
| 爱奇艺             |   NO  | Twitch                  | YES         |
| BOSS 直聘          |   NO  | SHEIN                   | YES         |
| 云闪付             |   NO  | Google Chrome           | NO          |
| 百度网盘           |   YES | Roku                    | YES         |
| 中国建设银行       |   NO  | intoLive                | YES         |
| 阿里巴巴           |   NO  | Nike                    | YES         |
| WPS Office         |   NO  | PicsArt                 | YES         |
| 58 同城            |   NO  | Google                  | YES         |
| Keep               |   YES | Calm                    | YES         |
| 微博               |   NO  | eBay                    | YES         |
| 闽政通             |   YES | Norton Secure           | YES         |
| QQ 浏览器          |   NO  | Airtime                 | YES         |
| 中国工商银行       |   YES | YEE                     | YES         |
| 美图秀秀           |   YES | Omegle                  | YES         |
| 安居客             |   NO  | Google Driver           | YES         |
| 七猫小说           |   YES | Tinder                  | YES         |
| 贝壳找房           |   NO  | Target                  | YES         |
| 农行掌上银行       |   NO  | Google Map              | NO          |
| 京东金融           |   NO  | OfferUp                 | YES         |
| 轻颜相机           |   YES | Grubhub                 | YES         |
| 番茄小说           |   NO  | Google Photos           | YES         |
| FaceApp            |   YES | Yubo                    | YES         |
| 得物               |   YES | Google Classroom        | YES         |
| 优酷视频           |   NO  | YOLO                    | YES         |
| 秘乐短视频         |   YES | Pandora                 | YES         |
| 网上国网           |   NO  | Fonts                   | YES         |
| 哈罗出行           |   NO  | Uber Eats               | YES         |
| 中国银行手机银行   |   YES | Mercari                 | YES         |
| 好省               |   YES | Reddit                  | YES         |
| 喜马拉雅           |   NO  | SoundCloud              | YES         |
| WIFI 万能钥匙      |   NO  | Hangouts Meet           | YES         |
| 扫描全能王         |   NO  | Google Docs             | NO          |
| UC 浏览器          |   NO  | Instacart               | YES         |
| 驾考宝典           |   NO  | Fitness Coach           | YES         |
| 天眼查             |   NO  | PictureThis             | NO          |
| 人人视频           |   NO  | Xbox                    | YES         |
| QQ 邮箱            |   NO  | Tubi                    | YES         |
| 淘宝直播           |   NO  | VideoToLive             | YES         |
| 西瓜视频           |   NO  | Amazon Photos           | YES         |
| 最珠海             |   NO  | Zillow                  | YES         |
| 办事通             |   NO  | Robinhood               | YES         |
| 知乎               |   YES | Hangouts                | YES         |
| 搜狗输入法         |   NO  | News Break              | YES         |
| 菜鸟裹裹           |   NO  | Enlight Video           | NO          |
| 招商银行           |   NO  | Youtube Music           | YES         |
| 苏宁易购           |   NO  | letgo                   | YES         |
| 美团外卖           |   NO  | InShot                  | NO          |
| 学习强国           |   NO  | Fetch Rewards           | YES         |
| 饿了么             |   NO  | Splice                  | YES         |
| 全民 K 歌          |   NO  | Postmates               | YES         |
| 今日头条           |   NO  | ESPN                    | YES         |
| 识货               |   YES | Duolingo                | YES         |
| 懂车帝             |   YES | Quibi                   | YES         |
| 全球骑士特购       |   NO  | Amazon Music            | YES         |
| Facetune2          |   YES | Audible                 | YES         |
| 中国联通           |   YES | Microsoft Outlook       | YES         |
| 平安好车主         |   NO  | Esty                    | YES         |
| 腾讯新闻           |   NO  | SONIC Drive-in          | YES         |
| 醒图               |   YES | Funimate Video          | YES         |
| 酷狗铃声           |   NO  | Telegram                | YES         |
| 百度地图           |   NO  | Instacart Shopper       | YES         |
| 探探               |   YES | AliExpress Shopping     | NO          |
| 多闪               |   NO  | YouTube Studio          | NO          |
| 中国移动           |   NO  | Miscorsoft Teams        | YES         |
| 作业帮             |   NO  | PlayStation             | NO          |
| 邮政手机银行       |   NO  | Bumble                  | YES         |
| Zoom               |   NO  | Messenger Kis           | NO          |
| 滴滴               |   YES | Facetune2               | YES         |
| 智联招聘           |   YES | Hoop                    | YES         |
| 虎牙直播           |   NO  | TextNow                 | YES         |
| 芒果 TV            |   YES | Vinkle                  | YES         |
| 百度贴吧           |   YES | McDonald's              | YES         |

通过这组数据，我们可以分析出很多有意思的东西

首先，BAT 三巨头的门户 APP 都已经具备了 Swift 混编的能力，例如百度系的百度主 App，百度网盘，百度贴吧，阿里系的手淘，芒果视频，腾讯系的微信，微视

其次，国内的[独角兽巨头们](http://www.199it.com/archives/864570.html)，似乎也做好了迎接 Swift 的准备，例如字节跳动（剪映，飞书），滴滴出行，自如，网易，小红书，得到，瑞幸咖啡，猿题库，英语流利说等。

最后，一个有意思的现象是，我发现国外 Top 102 的 ipa 总大小为 10G，而国内 Top 102 的 ipa 总大小将近 24G，不知道这是不是用 Swift 编写代码为包大小带来的正向收益，还是我们 ”CMD+C 和 CMD+V“ 的代码复用机制造成的。

## 总结与展望

做完这个调研，能得出什么结论呢？

1. 综合 Swift 的发展历史，语言排名，社区活跃度等因素来看，Swift 的发展是处于上升趋势的，可能相比于一些明星语言和明星技术热点，它的表现不是那么突出，但总趋势上升是不可否认的，这同样适用于 Objective-C 的总趋势下降。
2. Apple 应该会继续增加 Swift 语言在其生态圈的重要性并进行相应的战略部署，即使 Swift 语言还没完全达到 Module Stability，但从推出的 Swift 独占 SDK 已经看出了他们的想法。
3. 国内外各个互联网厂商在转向 Swift 的道路上已经走起来了，即使国内的占比还落后于国外，但总的趋势和涨幅势头已经十分明显了，以 BAT 为首的大部分互联网公司已经完成了 Swift 的混编工作，尤其是它们旗下的明星 App 已经接入了 Swift。

拿着这些结论来看看大家平日里对 Swift 的印象，我们又发现了哪些不一样的地方呢？

虽然国内 iOS 圈里常有人说”Swift 无用“，”Swift 火不了“，”我们不需要用 Swift 开发“，但冷静的分析下来，国内的各大厂商真的抛弃 Swift 了么？ 

通过这份真实的数据，我想答案很明显，它们都没有放弃 Swift，而且都在积极的做准备，这就很像嘴上说着不要不要，但身体却出卖了自己。

国外的 Swift 氛围不用多说，大家都已经可以看出谁会是 iOS 端上未来的主角。这里我们就说说国内的情况：

我们都知道，淘系的主 App ---- 手机淘宝在今年完成了 Swift 混编能力的建设，这意味着什么？对于淘系App 来说，这么庞大，复杂的工程都已经具备了混编的能力，那么阿里旗下的其余 App 转型将不再有什么逾越不了的鸿沟。

号称 App 工厂的字节跳动，在很早就尝试了 Swift 的混编开发，早期还限定在一些小型的项目，现在它们已经具备了中大型项目的 Swift 混编能力，这一点可以在它们的海外明星应用 Helo 上的得到验证，目前这款 App 在字节跳动内部的排名已经跻身到前 5 名了，另外从一些渠道得知，它们的明星应用 - 今日头条也将在近期开展 Swfit 混编能力的建设。

而国内那些还在高速发展的公司，例如美团，京东，拼多多，快手等公司，在 Swift 上的探索还显得比较落后，可能还停留在一些小型内部应用或者 B 端的应用上，甚至也有可能完全没有开展过相关的工作。

这么看来，国内厂商的 Swift 格局已经十分清晰，大体会呈现三个梯队：

第一梯队：以 BAT 为代表的顶端团队已经解决了复杂的，巨型的，历史包袱重的工程项目如何使用 Swift 的问题，对他们而言，未来的问题就是怎么将 Swift 用到真正的业务代码上了，玩的好，玩的溜的将是它们要关注的问题了。

第二梯队：以字节跳动，网易为代表的一些公司，已经解决了部分 Swift 混编的问题，对于其内部工程复杂度最高，历史包袱最重的 App 还没有实现混编的能力，例如字节跳动的 Helo 和抖音，网易的网易公开课和网易云音乐，这些公司在近期面临的问题可能会是如何解决混编。

第三梯队：以美团，京东为代表的一些公司，在这方面还没有开展相应的工作或者开展的还比较少，他们目前可能更关注的还是在解决自身业务发展的问题，例如动态化，中台建设，容器化等方面的技术积累与战略部署。

不管怎么看，Swift 在国内的发展既不是完全停滞，也不是无人问津，只是真正玩的人比较“低调”而已，但该来的一定会来。

不知道看完这篇文章，你认为 Swift 在国内的发展会是什么样子呢？
