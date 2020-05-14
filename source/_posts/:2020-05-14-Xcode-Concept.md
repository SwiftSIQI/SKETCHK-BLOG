---
title: Xcode Concept 学习笔记
comments: true
date: 2020-05-14 20:04:12
updated:
tags:
  - Xcode
categories:
  - Xcode
---

只怪自己当年学东西不够扎实，这次让我好好理解一下 Xcode 里的相关基础概念吧！

<!-- more -->

如果使用 Xcode 进行开发，我们常常会与这么几个概念打交道：Workspace，Project，Target，Scheme 和 Build Setting。

官方对这些概念的解释可以参考这篇文档 - [Xcode Concepts](https://developer.apple.com/library/archive/featuredarticles/XcodeConcepts/Concept-Projects.html)。

虽然文档本身已经被列为 Archived 的状态，但大毛病没有，还是可以拿来再学习一遍，所以后面的内容将围绕它展开。

另外在学习和调研的过程中，我发现了一个蛮有价值的博客，地址是 [pewpewthespells.com](https://pewpewthespells.com/)，里面有不少关于 Xcode 的文章，估计是个做 CI/CD 工作的妹子，下面的几篇文章可以当做索引手册收藏起来：

* [The Unofficial Guide to xcconfig files]( https://pewpewthespells.com/blog/xcconfig_guide.html)
* [Managing Xcode](https://pewpewthespells.com/blog/managing_xcode.html)
* [Xcode Build Settings Reference](https://pewpewthespells.com/blog/buildsettings.html)

## 概述

* **Target** -一个 target 代表一个产品
* **Project** - 对应 `.xcodeproj` 类型的文件，project 包含了构建产品所需的源文件，一个 project 可以有多个 target
* **Workspace** - 对应 `.xcworkspace` 文件，用来组织管理 project 和其他文档的，workspace 可以包含多个 project，project 可以属于多个 workspace
* **Scheme** - scheme 决定了哪个 target 去运行，它可以针对编译，运行，测试，打包等进行配置
* **Build Setting** - build settings 就是构建产品时的一些设置，target 可以覆盖 project 一些相同的设置


## Xcode Target

一个 Target 确定一个产物（product）的构建，包括一些指令（instructions），例如怎么从一个 Project 或者 Workspace 的一堆文件导出一个产物。简单来说，一个 Target 就定义了一个产物，一个 Target 对应一个 Product，它管理着一个产物的 Build System 的“输入”（一堆源文件和一些处理这些源文件的 Instruction）。 Projects 可以包含一个或者多个 Target，它们代表不同的产物，例如：如果你的产物需要做 Lite 和 Pro 版本，那么你可以考虑采取两个 Target 来处理。

构建一个产物的 Instructions（指令）的表现形式是构建设置(Build Settings)，构建规则(Buidling Rules)和构建参数(Build Phases)，这些都可以在 Xcode 的 Project Editor 中调整。一个 Target 的 Build Settings 继承 Project 的 Build Settings，但是可以重写覆盖 Project Settings。同时间内只能有一个 Active Target，Xcode Scheme 能够指定 Active Target。

一个 Target 可以跟其他 Target 相关联。如果一个 Target 在构建的时候需要另外一个 Target 的输出，我们说前者依赖于后者。

如果两个 Target 在相同的 Workspace 里，Xcode 能够发现它们的依赖关系，它能够以需要的顺序构建产品。这样的关系可以被称为隐形从属依赖(Implicit Dependency)。当然你也可以在 Build Setting 里为两个 Targets 指明显示依赖关系(Explicit Dependency)。

例如：在同一个 workspace 中，可以构建一个 library 和一个链接这个 library 的 application。Xcode 可以发现这种依赖关系，并首先自动构建 library。但是，如果想链接某个版本的 library，就需要在 build settings 明确依赖关系，该依赖项会覆盖隐式依赖项。

## Xcode Project

Xcode Project 是个构建一个或者多个产物所需要的文件，资源，信息等的存储库（repository）。Project 包含用于构建产物的所有元素，并且管理这些元素间的关系。它包含一个或多个 Target，指定怎样去构建产品。Project 在工程里面默认的为所有的 Target 指定 Build Settings，当然每个 Target 可以覆盖 Project 的 Build Settings，去指定自己特有的 Build Settings。

一个 Xcode project 包含下面的信息：

* 源文件的引用：
  * 源码，包括头文件和实现文件
  * Libraries and Frameworks
  * 资源文件(plist等)
  * 图片文件
  * nib 文件(xib, stroyboard等)
* 用于在结构导航器（ structure navigator）中组织源文件（source files）的组（groups），这里又分物理文件和引用文件
* Project 级别的 Build Configurations. 你可以为 Project 指定多个 Build Configuration，例如，Xcode 就默认为我们指定了 Debug 和 Release 的 Build settings，当然你也可以自定义。
* Targets，每个 Target 会指定：
  * 通过 Project 构建的一个产物的引用
  * 构建该产物所需的资源文件的引用
  * 用于构建该产物的构建配置（Build configurations），包括对其他 Targets 和 Settings 的依赖；如果 Targets 的 build configurations 没有配置时，使用 Project 级别的 Build Configurations
* 用来 Debug 和 Test 程序的可执行环境（Executable Environment），包括：
  * 从 Xcode run 或 debug 时启动的可执行文件
  * 要传递给可执行文件的命令行参数
  * 程序 run 时要设置的环境变量

A project 可以单独存在，也可以被包含在 workspace 里面(cocoapods 就是被包含在 workspace 里面)。


## Xcode Workspace

一个 Workspace 是一个 Xcode 文档，组合不同的 Project、文档，所以你可以同时管理多个 Project。一个 Workspace 可以包含任意数量的 Xcode projects 和其他文件。除了组织每个 Xcode Projects 中的所有文件外，Workspace 还维护 projects 与他们各自 Targets 之间的隐式/显示关联。

### Workspace 扩展 workflows 的范围

一个工程文件（Project File）包含指向 project 中所有文件的指针，Build Configurations 和 Project 的其他信息。在 Xcode 3 之前，Projects 之前关联是很复杂的事情，大多数工作流仅限于单个 Project。从 Xcode 4 之后，你可以创建一个 Workspace 去包含多个 Projects 和其他文件。

除了提供被包含在 Xcode Project 中的所有文件的访问外，Workspace 还拓展许多重要的 Xcode Workflows 的范围。例如，由于 indexing（文件索引）遍布整个 Workspace，所以，在 workspace 中， code completion、Jump to Definition 和所有其他的内容感知特性，可以在所有 Projects 中无缝衔接运作。因为 refactoring operations（重构操作）横跨整个 Workspace 的所有内容，所以，你可以在一个 framework project 中重构 API，并且在其他 application projects 中使用这个 framework。构建时，一个 project 可以利用 workspace 中其他 projects 的 products。

workspace 文档包含被囊括的 projects 和其他文件，不再有其他数据。一个 project 可以被多个 workspace 持有。下图展示一个 workspace 包含两个 Xcode projects 以及一个文档 project。

### Workspaces 中的 Projects 共享 Build Directory

默认情况下，Workspace 下面的 projects 都是在同一个目录下构建的，也就是 Workspace 的编译目录(workspace build directory)。由于是在同一个目录下面，Project 的资源文件都彼此都是可见的，可互相引用的。所以，如果你有多个 Projects 使用相同库的时候，不需要将它分别拷贝到各个 Project 中。

Xcode 会在编译目录下检查文件发现它们的隐形从属依赖。例如，如果 Workspace 中的一个 Project 编译的时候需要链接到相同 Workspace 的其他 Project 某个库，Xcode 会自动帮你先编译那个库，即使构建配置没有显式的指定从属依赖关系。如果需要的话，你可以指定显式从属依赖，但是你必须创建 Project 引用。

Workspace 中的每个 Project 仍然有属于它们自己的独立的标识。你能通过 project 的打开方式控制 project 受不受其他 projects 的影响，例如单独打开 Project 而不是通过 Workspace。因为，一个 Project 可以属于多个 Workspace，你可以任意组合 Projects，而无需重新配置 projects 或者 workspaces。

你可以使用 workspace 默认的 build directory，也可以自己指定一个。注意：如果一个 project 指定一个 build directory，这个 build directory 会覆盖全部所在的 workspace 里的默认 build directory。

## Xcode Scheme

一个 Xcode Scheme（方案）定义三样东西：一个要生成的目标（targets to build）的集合、building 时使用的配置（configuration）、以及要执行的测试集合。

你可以拥有任意数量的 scheme，但一次只能有一个是活跃状态（active），你可以指定 scheme 是否储存在 project 中（这种方案下，scheme 在每一个包含这个 project 的 workspace 中都可用），或者储存在 workspace 中（仅在当前 workspace 中可用）。选择要激活的 scheme 时，可以选择运行目标（设备）。

## Build Setting

一个 Build Setting 是一个变量，包含着如何构建产物的信息。例如，可以指定 Xcode 传递给编译器的选项

Build Settings 有 Project 和 Target 两个级别，Project 级别中的 Build Setting 适用项目中所有的 targets，只要该项 setting 没有被 Target 级别的重写覆盖。

每个 Target 管理着创建一个产物的源文件，一个 build configuration 指定一组 build settings，用于以特定的方式构建一个 product。例如，通常有 debug 和 release 俩种分开的 build configurations。

一个 Build Setting 包含两个部分：Setting Title（标题） 和 Definition（定义），类似于 key-value 结构。前者标示该 Build Setting 的名称，后者是一个常量或一个表达式，用于确定 Build Setting 的值。

另外，当你通过 Project 模板新建一个 Project 时，Xcode 会生成一个默认的 Build Settings，你也可以为 Project 或者某个 Target 创建自定义的 Build settings。你还可以设定 Conditional Build Settings，一个 Conditional Build setting 的值取决于是否满足一个或多个先决条件。这个机制也可以被用在指定用于基于目标架构构建产品的SDK。

## 参考资料

1. [Xcode: What is a target and scheme in plain language?]( https://stackoverflow.com/questions/20637435/xcode-what-is-a-target-and-scheme-in-plain-language/20637892#20637892)
2. [Xcode Project vs. Xcode Workspace - Differences](https://stackoverflow.com/questions/21631313/xcode-project-vs-xcode-workspace-differences)
3. [Xcode 相关概念](https://joakimliu.github.io/2016/09/24/Xcode-Concepts/)
4. [iOS项目Project和Target配置详解](http://www.liugangqiang.com/2018/03/22/iOS%E9%A1%B9%E7%9B%AEProject%E5%92%8CTarget%E9%85%8D%E7%BD%AE%E8%AF%A6%E8%A7%A3/)