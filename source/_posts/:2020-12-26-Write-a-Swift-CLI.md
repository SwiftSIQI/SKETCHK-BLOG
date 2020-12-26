---
title: 使用 Swift 编写 CLI 工具的入门教程
comments: true
date: 2020-12-26 15:28:00
updated:
tags:
  - Swift
  - CLI
categories:
  - Swift

---

通过这篇文章，你将了解到为何要使用 Swift 编写脚本工具, 同时我会通过 Step-By-Step 的方式，指导你从头搭建一个 Swift CLI 项目，并写下了一个简单的命令，除此之外, 还会告诉你如何在 Xcode 里调试 Swift CLI 类型的工程项目，在文章的最后，我向你介绍了一些开源社区的优秀资源来加速你的开发。

总之，我们十分期待你使用 Swift 编写属于自己的命令行工具！

<!-- more -->

## 概述

最近在工作中过程中基于 Swift 开发了两款命令行工具：

* Nezuko(未开源) ，一款面向美团壳工程的，类 `create-react-app` 的脚手架工具。
* [ImportSanitizer](https://github.com/SketchK/import-sanitizer)，一款能够修复不规范头文件引用方式的自动化工具。

在整个开发过程中，我们体会到了 Swift 带来的一些变化，当然这里既有好的，也有坏的，不过总的来说，使用 Swift 进行脚本开发还是一件让人愉悦的事情，所以我们迫不及待的邀请你，也就是这篇文章的读者，和我们一起加入 Swift 开发的大军中！

这篇文章通过 Step-By-Step 的方式，指导你完成一个基于 Swift 的命令行工具，不过在开始之前，我们先聊聊为什么要写脚本，以及为什么选用 Swift。

### Why Scripting

对于软件工程师来说，我们经常会遇到这样的工作，例如重命名设计师提供的图片素材，在海量的数据中提取一些特定信息。这种工作有一些共性，就是它的逻辑很简单，就像代码里的 `if-else` 一样，只要你够严谨，就一定能得到想要的答案，

但这种机械性的工作很容易因为人为的因素导致错误，例如手抖，眼瞎，以及间歇性失忆，哈哈，而这时候，机器会显得比人靠谱多了！

同时，重复同样的工作是一件低效的事情，我们完全可以通过编写相应的代码将任务自动化，这将极大的提升我们的工作效率。

当然可能有人会说，我还是自己弄吧，但你不觉得这种重复的，机械性的工作很无聊么，我们可是要改变世界的工程师啊！

### Why Swift

至于为什么使用 Swift 写脚本，我想有人可能会给出这样的答案:

> swift 真的太棒了，我喜欢 Swift，它是最好的语言！我要用它写后端，要用它写前端，用它写 iOS，写 Android，用它写 CLI，总之，我要用它解决一切的编程问题！

但说实话，这是一个很主观的判断，并不应该成为我们使用 Swift 写 CLI 的客观因素，在实际使用了一段时间后，我认为下面几个因素才是我们使用 Swift 编写脚本的主要原因：

* 降低了 App 开发者编写脚本的门槛，减少了上下文切换的负担
* Swift 提供了一些真的很不错的内置库，例如 combine，core graphics，urlsession 等
* 可以将 App 里的代码引入脚本，避免重复工作

关于前两点，我想大家应该会比较好理解，毕竟每次在 Swift 代码和 Bash，Ruby，Python，JavaScript 中切换，会让我产生一种深深的抗拒感，另外在 iOS 上已经得到证明的 core graphics 有理由让我们相信它的品质，同时天然内置的 Combine 让我们在异步编程上有了很好的体验，这让我想起被 JavaScript 里 yield 支配的恐惧，问我为什么要用 yield，不妨来试试 React + Redux + Redux-Saga 的大礼包呀!

关于最后一点，我想展开说说，一方面是因为它基于我真实的开发体验，另一方面是它确实让我真正意识到 Swift 写脚本的优势所在。

在年初的时候，我曾经一度痴迷 SpriteKit，并尝试开发一款属于自己的横版过关游戏，这其中涉及到大量的图片处理，例如使用 Animation 的方式将多张静态图片整合成动态效果，大体的效果就像下面的 gif 一样

![01](1.gif)

这背后的代码大概如下所示，通过读取相应顺序和数量的图片构建 gif 动画

```swift
Animation(
    name: "Units/swordsman/male/attack/right",
    frameCount: 8,
    duration: 1.12,
    tintColor: .blue
)
```

虽然看起来代码不多，手写一下就 ok 了，但你得知道，就这样一个杂兵角色，就会有攻击，跳跃，行走，跑动等等等动作，再加上各个方向，以及特殊效果，如果再考虑到，我们的游戏里面大概有 20 多个兵种和 4 个英雄，我想你大概已经体会到这个工作的痛苦了！

在一开始，我用的是 Ruby 来解决重复代码的生成工作，但这里为了减少上下文切换，我将采用 Swift 类型的伪代码来做展示，方便大家快速理解

```swift
let unitKinds = ["swordsman", "archer", "knight", "catapult"]

for kind in unitKinds {
    guard let config = try? File(path: "\(kind.identifer)/Config") else {
        continue
    }
    try codeGenerator.generateCode(from: config)
}
```

虽然这么写已经帮我节省了不少时间，但我还是得每次手动维护 unitKinds 数组，并保持它与游戏中的模型数据同步，这其实也挺烦人的，不是么？

```swift
class Unit {
    enum Kind: Int {
        case swordsman
        case archer
        case knight
        case catapult
    }
}
```

某日，看着上面的游戏模型数据，我突然顿悟，如果我用 Swift 编写脚本的话，我完全可以复用游戏里的数据模型啊！于是乎，便有了下面的代码：

```swift
import GameModels

for kind in EnumSequence<Unit.Kind>() {
    guard let config = try? File(path: "\(kind.identifer)/Config") else {
        continue
    }
    try codeGenerator.generateCode(from: config)
}
```

至此，我终于做到了不用再手动维护脚本里的任何代码，就可以直接与 App 里的源码保持一致，这样是不是很 cool！同样的道理，我们还可以应用到很多方面，例如直接利用网络层的 modle 文件生成 mock 数据，而不用在 JSON 编辑器上小心翼翼的粘贴复制了！

所以还在等什么呢，让我们开始写一个 Swift CLI 吧！

## 使用 SPM 搭建开发框架

为了开发 command line tool（CLI），我们需要创建一个新的文件夹，并使用 swift package manager（SPM）来初始化项目

```sh
$ mkdir CommandLineTool
$ cd CommandLineTool
$ swift package init --type executable
```

最后一行的 `type executable` 参数将告诉 SPM，我们想创建一个 CLI，而不是一个 Framework。

### 项目里的文件

在 SPM 初始化项目后，我们会得到如下的一个文件夹结构

```sh
.
├── Package.swift
├── README.md
├── Sources
│   └── CommandLineTool
│       └── main.swift
└── Tests
    ├── CommandLineToolTests
    │   ├── CommandLineToolTests.swift
    │   └── XCTestManifests.swift
    └── LinuxMain.swift
```

其中有几个需要关心的文件

* `Package.swift` 文件： 用于描述当前 Package 的信息及其依赖，需要记住的是，在 SPM 的世界里，不再有 Pod 的概念，与之对应概念是 Package，而 CLI 本身也是一个 Package
* `main.swift` 文件：这个文件在 Sources 目录下，它代表整个命令行工具的入口，另外记住不要更换这个文件的名字！
* `Tests` 文件夹：这个文件夹是用于放置测试代码的。
* `.gitignore` 文件：通过这个文件，git 会自动忽略 SPM 生成的 build 文件夹（`.build` 目录）以及 Xcode Project 

### 将代码划分为 framework 和 executable

我的一个个人建议是，在一开始就最好将源代码分成两个模块，一个是 framework 模块，一个是 executable 模块。

这样做的原因有 2 点：

* 会让测试变得更加容易
* 让你的命令行工具也可以作为其他工具依赖的 Package

具体怎么做呢？

首先，我们要保证 Sources 目录下有两个文件夹，一个用于存放 executable 相关的逻辑，一个用于存放 framework 相关的逻辑，就像下面一样：

```sh
$ cd Sources
$ mkdir CommandLineToolCore
```

SPM 的一个非常好的方面是，它使用文件系统作为它的处理依据，也就是说，只要采用上述操作提供的文件结构，就等于定义了两个模块。

紧接着，我们在 `Package.swift` 里定义了两个target， 一个是 `CommandLineTool` 模块，一个是 `CommandLineToolCore`

```swift
import PackageDescription

let package = Package(
    name: "CommandLineTool",
    targets: [
        .target(
            name: "CommandLineTool",
            dependencies: ["CommandLineToolCore"]
        ),
        .target(name: "CommandLineToolCore")
    ]
)
```

通过上面这种方式，我们让 executable 模块依赖了 framework 模块。

### 构建 Xcode 项目

为了能方便的运行和调试代码，我们还需要使用配套的开发工具！

好消息是 SPM 可以根据文件信息自动创建 Xcode 工程，这意味着我们可以使用 Xcode 来开发 CLI 了。

而且在 `.gitignore` 中会自动忽略这个工程项目，这同时意味着，我们不需要更新 Xcode Project 文件，也不需要担心这类文件的冲突问题，只需要通过下面的命令即可完成工程文件的生成。

```sh
$ swift package generate-xcodeproj
```

记得需要在根目录下执行上面的命令，另外在执行过程中会得到一个 warning，让我们暂且忽略它，在后面我们会将其修复！

## 开始动手

### 定义程序入口

为了能够在命令行和测试用例中方便的运行我们的代码，我们最好不要在 `main.swift` 中添加过多的逻辑，而是通过程序调用的方式唤起 framework 中的主逻辑。

为了实现这样的目的，我们需要创建一个名为 `CommandLineTool.swift` 的文件，将其放在 framework 模块中（`Sources/CommandLineToolCore`），它里面的内容如下

```swift
import Foundation

public final class CommandLineTool {
    private let arguments: [String]

    public init(arguments: [String] = CommandLine.arguments) { 
        self.arguments = arguments
    }

    public func run() throws {
        print("Hello world")
    }
}
```

同时在 `main.swift` 中添加 `run()` 方法

```swift
import CommandLineToolCore

let tool = CommandLineTool()

do {
    try tool.run()
} catch {
    print("Whoops! An error occurred: \(error)")
}
```

### Hello，World

让我们用命令行看看执行效果吧！不在在真正的运行前，我们还需要完成编译工作，让我们在根目录下执行 `swift build` 吧，然后再执行 `swift run` 命令。

```sh
$ swift build
$ swift run
> Hello world
```

> 我们其实可以通过直接调用 `swift run` 命令来达到运行程序的目的，因为如果需要的话，它会自动编译我们的项目，但学习一下底层命令的工作原理总是有益的。

### 增加依赖

除非你正在构建一些十分“特殊”的东西，否则你会发现自己需要为你的命令行工具添加一些依赖关系，毕竟有好用的轮子为啥不用呢？

任何 Swift Package 都可以被添加为依赖项，只需在 `Package.swift` 中指定它。

```swift
import PackageDescription

let package = Package(
    name: "CommandLineTool",
    dependencies: [
        .package(
            name: "Files",
            url: "https://github.com/johnsundell/files.git",
            from: "4.2.0"
        )
    ],
    targets: [
        .target(
            name: "CommandLineTool",
            dependencies: ["CommandLineToolCore"]
        ),
        .target(
            name: "CommandLineToolCore",
            dependencies: ["Files"])
    ]
)
```

上面我添加了对 `Files` 组件的依赖，它可以让我们在 Swift 中轻松处理文件和文件夹的相关操作。在后面的教程中，我们将使用它在当前文件夹中创建一个文件。

### 安装/更新依赖

一旦我们声明了新的依赖关系，只需要求 SPM 解析新的依赖关系并安装它们，然后重新生成 Xcode 项目即可。

```sh
$ swift package update
$ swift package generate-xcodeproj
```

### 参数解析

让我们修改一下 `CommandLineTool.swift` 里的内容。

将其从打印 `Hello, World` 的逻辑变为根据命令行参数创建文件的逻辑

```swift
import Foundation
import Files

public final class CommandLineTool {
    private let arguments: [String]

    public init(arguments: [String] = CommandLine.arguments) { 
        self.arguments = arguments
    }

    public func run() throws {
        guard arguments.count > 1 else {
            throw Error.missingFileName
        }
        
        // The first argument is the execution path
        let fileName = arguments[1]

        do {
            try Folder.current.createFile(at: fileName)
        } catch {
            throw Error.failedToCreateFile
        }
    }
}

public extension CommandLineTool {
    enum Error: Swift.Error {
        case missingFileName
        case failedToCreateFile
    }
}
```

如上所述，我们把对 `Folder.current.createFile()` 的调用包装在自己的 `do、try、catch` 中，以便为用户提供统一的，自定义的错误 API。

#### Argument Parser

除了刚才提到的参数解析方式，Apple 官方还提过了一个更优化的解决方案 - [Swift Argument Parser](https://github.com/apple/swift-argument-parser)

这里我们做一下简单的介绍，以官方代码为参考示例：

```swift
import ArgumentParser

struct Repeat: ParsableCommand {
    @Flag(help: "Include a counter with each repetition.")
    var includeCounter = false

    @Option(name: .shortAndLong, help: "The number of times to repeat 'phrase'.")
    var count: Int?

    @Argument(help: "The phrase to repeat.")
    var phrase: String

    mutating func run() throws {
        let repeatCount = count ?? .max

        for i in 1...repeatCount {
            if includeCounter {
                print("\(i): \(phrase)")
            } else {
                print(phrase)
            }
        }
    }
}

Repeat.main()
```

我们可以看到，它的使用方式并不麻烦。

* 首先遵守 ParsableCommand 协议
* 其次声明一个参数类型（Flag，Option，Argument），定义你需要从命令行中收集的信息，并用 ArgumentParser 的属性包装器来装饰每个存储属性
* 最后在 `run()` 方法中实现核心逻辑。

在实际的运行过程中，ArgumentParser 会解析命令行参数，实例化你的命令类型。同时 ArgumentParser 会使用属性名，类型信息，以及你在属性包装器里提供的细节，来提供有用的错误信息和帮助信息，具体效果如下所示。

```sh
$ repeat hello --count 3
hello
hello
hello
$ repeat --count 3
Error: Missing expected argument 'phrase'.
Usage: repeat [--count <count>] [--include-counter] <phrase>
  See 'repeat --help' for more information.
$ repeat --help
USAGE: repeat [--count <count>] [--include-counter] <phrase>

ARGUMENTS:
  <phrase>                The phrase to repeat.

OPTIONS:
  --include-counter       Include a counter with each repetition.
  -c, --count <count>     The number of times to repeat 'phrase'.
  -h, --help              Show help for this command.
```

由于本文的例子较为简单，我们这里就不增加 ArgumentParser 的依赖来增加项目的复杂度了。

即使如此，我相信通过上面的介绍，你也大致了解到了 ArgumentParser 的使用方式了，记得将其用在你自己的项目中吧！

### 编写单测

我们几乎已经准备好发布这个命令行工具了，但在这样做之前，我们还是需要通过编写一些测试来确保它真正的按照预期工作。

由于我们之前将整个项目划分成了 framework 和 executable 的结果，所以测试将变得十分容易。我们所要做的就是以程序调用的方式运行它，并断言它创建了一个具有指定名称的文件。

首先在 `Package.swift` 文件中添加一个测试模块，在你的 `target` 数组中添加以下内容。

```swift
.testTarget(
    name: "CommandLineToolTests",
    dependencies: ["CommandLineToolCore", "Files"]
)
```

最后，重新生成 Xcode 项目。

```swift
$ swift package generate-xcodeproj
```

再次打开 Xcode 项目，跳到 `CommandLineToolTests.swift` 中，添加以下内容。

```swift
import Foundation
import XCTest
import Files
import CommandLineToolCore

class CommandLineToolTests: XCTestCase {
    func testCreatingFile() throws {
        // Setup a temp test folder that can be used as a sandbox
        let tempFolder = Folder.temporary
        let testFolder = try tempFolder.createSubfolderIfNeeded(
            withName: "CommandLineToolTests"
        )

        // Empty the test folder to ensure a clean state
        try testFolder.empty()

        // Make the temp folder the current working folder
        let fileManager = FileManager.default
        fileManager.changeCurrentDirectoryPath(testFolder.path)

        // Create an instance of the command line tool
        let arguments = [testFolder.path, "Hello.swift"]
        let tool = CommandLineTool(arguments: arguments)

        // Run the tool and assert that the file was created
        try tool.run()
        XCTAssertNotNil(try? testFolder.file(named: "Hello.swift"))
    }
}
```

此外，还可以添加另一个测试，以验证在没有给定文件名或文件创建失败时是否抛出了正确的错误。

要运行测试，只需在命令行上运行 `swift test` 即可。

### 安装工具

现在我们已经构建并测试了我们的命令行工具！下面开始，我们会尝试安装它，并使它能够在任何地方运行。

要做到这一点，需要在 `swift build` 后面增加 release 的配置，也就是 `-c relase` 参数，然后将编译后的二进制文件移到 `/usr/local/bin`。

```sh
$ swift build -c release
$ cd .build/release
$ cp -f CommandLineTool /usr/local/bin/commandlinetool
```

## 调试技巧

在实际的开发过程中，不免会遇到一些 bug，这部分内容我们将讨论一些针对 CLI 的 Xcode 调试技巧。

### 通过 Xcode 添加入参

首先，命令行大多是需要输入参数的，所以我们在 Xcode 里如何为命令行添加入参呢？

首先，在 Xcode 的 Toolbar 中，我们点击 choose scheme 面板中的 `Edit Scheme...` 按钮

![02](2.jpg)

在弹出的界面中点击左侧 Run 面板，并继续点击右侧的 Argument 的 Tab 按钮，我们会看到如下的界面，此时我们可以在 Arguments Passed On Launch 中添加命令行所需的参数，例如这里我们添加了一个 `Hello.swift` 的参数。

![03](3.jpg)

此时，我们再次通过 `CMD+R` 的方式运行程序，就会在构建产物的目录中，看到生成的 `Hello.swift` 文件

![02](4.jpg)

## 让开源社区加速你的开发

除了 Foundation 自带的 Combine，CoreGraphes，URLSession 等，在官方的技术社区还有很多不错的组件能够加速你的开发，例如

* [Swift Tool Support Core](https://github.com/apple/swift-tools-support-core)：SPM 和 llbuild 里通用的基础架构代码，可以将其看做是 Foundation 在 CLI 方向的加强库。
* [Swift NIO](https://github.com/apple/swift-nio)：如果 URLSession 不能满足你的需求或者你需要进行跨平台开发，SwiftNIO 这个网络库应该是能满足你的诉求，它是一个跨平台异步事件驱动的网络应用框架，用于快速开发可维护的高性能协议服务器和客户端。
* [Swift Log](https://github.com/apple/swift-log)：一个用于做日志记录工作的组件。
* [Swift Metrics](https://github.com/apple/swift-metrics)：当我们需要为某个系统，某个服务做监控，做统计的时候，Swift Metrics 就是你的不二之选！
* [Swift Crypto](https://github.com/apple/swift-crypto)：一个跨平台的加密库，基于 Apple 自身的 CryptoKit 改造而来。
* [Swift numerics](https://github.com/apple/swift-numerics)：这个库为 Swift 提供了许多与数值计算相关的功能模块。
* [Swift Protobuf](https://github.com/apple/swift-protobuf)：如果你在网络通信的过程中传输的是 Protobuf 类型的文件，可以通过这个库进行解析
* [Swift Atomics](https://github.com/apple/swift-atomics)：为各种 Swift 类型提供原子操作，包括整数和指针值。
* [Swift Backtrace](https://github.com/swift-server/swift-backtrace)：这个 Package 为项目提供了自动打印程序崩溃信息的能力。

总之，这个社区在不断的发展中，很多新的官方库也在如火如荼的建设中，如果你发现这里的内容还不够用，可以关注一下 [Vapor](https://github.com/vapor)，[PerfectlySoft Inc](https://github.com/PerfectlySoft) 和 [SwiftWasm](https://github.com/swiftwasm) [Crossroad Labs](https://github.com/crossroadlabs)的 Group，也能找到很多不错的 package 资源。

当然，也有很多个人开发者提供了不错的 Package 资源，例如：

* [Guitar](https://github.com/artsabintsev/guitar)：这绝对会是你在开发中需要到的东西，一个正则匹配加强库！
* [Rainbow](https://github.com/onevcat/Rainbow)：可以对命令行里的输出内容增加文本颜色，背景样式等。
* [SwiftShell](https://github.com/kareman/SwiftShell)：可以在 Swift 里调用 Shell 命令的 Package
* [Swift-SH](https://github.com/mxcl/swift-sh)：同样是一个 Swift 里调用 Shell 的 Package，介绍它的原因是因为它的作者是 Homebrew 的开发者 mxcl
* [Files](https://github.com/JohnSundell/Files)：与文件操作相关的 Package，在教程里已经提及过。
* [Path](https://github.com/mxcl/Path.swift)：Files 在处理一些路径上还是有短板，mxcl 开发的这个组件很好的补充了 Files 的功能。
* [Release](https://github.com/johnsundell/releases)：可以通过 Swift 脚本或命令行工具轻松解析 Git 仓库中的发布版本，支持远程仓库和本地仓库。
* [XGen](https://github.com/johnsundell/xgen)：通过 Swift 脚本或命令行工具中轻松地生成 Xcode Project 和 Playground。

## 总结

通过这篇文章，你了解到了如何从头搭建一个 Swift CLI 项目，并写下了一个简单的命令，同时你也知道了如何在 Xcode 里调试工程项目，在文章的最后，我们向你介绍了一些开源社区的优秀资源来加速你的开发。

总之，我们十分期待你使用 Swift 编写属于自己的命令行工具！
