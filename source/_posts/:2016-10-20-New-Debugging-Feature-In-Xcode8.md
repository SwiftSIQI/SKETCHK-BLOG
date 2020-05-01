---
title: Xcode 8 的 Debug 新特性 ---- WWDC 2016 Session 410 & 412 学习笔记
comments: true
date: 2016-10-20 17:37:32
updated:
tags:
	- WWDC
	- Xcode
categories:
	- Xcode

---

<!-- more -->

![01](01.jpg)

## Overview
本文针对 WWDC 2016 Session 410 和 412 以及 WWDC 2015 Session 413 中的内容进行了整理.   

文中主要倾向于如何使用,以及这些工具的使用场景, 如果想了解这些工具的工作原理或者细节方面的东西,请大家观看原视频即可.具体链接在文中最后的参考资料里.  

废话少说,接着往下看吧.

## Static Analyzer
Static Analyzer 是一个常见的 debug 的工具, 苹果工程师在 WWDC 中是这样介绍它的:

* 不需要 running code (unlike sanitizers)
* 在处理 edge-case 的 bugs 时, 有着优异的表现
* 支持 C, C++ 和 Objective-C

### 如何使用 Static Analyzer
使用 Static Analyzer 很简单, 你可以通过选择 `Product -> Analyze` 或者 `Cmd + Shit + B` 的方式执行, 如果有错误,就会在 Issue Navigator 上显示出来

![02](02.jpg)

在今年的 Session 412 中, Apple 的工程师告诉我们在 Xcode 8 中, Static Analyzer 能够检测出三种新的错误, 它们分别是:

* Localizability 
* Instance Cleanup
* Nullability 

看英文有点不好理解,不用担心,接着往下看,咱们一个个的说.

### Localizability
Localizability 其实说的是 Static Analyzer 现在能够检测出本地化信息缺失的问题,目前能够检测出来两种类型的错误, 一种是没有使用 NSLocalizeString 这样的 API, 而直接给控件设置 Sting 的情况, 一种是使用了相应的 API, 但在 comment 信息里面赋值为 nil. 如果有错, 就会像下图一样, 在代码下方出现一个蓝色提示条, 告诉开发者具体的错原因.

![03](03.jpg)

在 Xcode 里,检测第二种类型的错误并不是默认开启的,如果想开启,需要在BuildSting 中进行如下设置: 

![04](04.jpg)

### Instance Cleanup
Instance Cleanup 说的是什么呢? 

这说的是在 MRC 的代码中, 尤其在 dealloc 中,我们不应该对 assign 类型的属性进行 release 操作,应该对 retain 或者 copy 类型的属性进行 release 操作, 如果不这样操作的话,会引发一些不必要的麻烦. 不过现在有了 Xcode 8, 这些问题就交给 Static Analyzer 吧,它能够很准确的检测出这样的错误.

![05](05.jpg)

![06](06.jpg)

Anyway, 还是建议你把代码转成 ARC 吧! 不知道怎么转, 看下图

![07](07.jpg)

### Nullability
关于这个话题是说的什么呢?  

首先我们得先说说在 2015 年的 WWDC 大会上, Objective-C 引入的一个新特性就叫做 Nullability, 用于表明一个东西到底可以为 nil 还是不可以为 nil , 这和 Swift 里的 option 类型很相似. 既然知道了这个玩意后,我们再说说 Static Analyzer 在这一块到底能够干点什么?   

通俗的说, Static Analyzer 可以检测出在不同场景下是否做到了 nullability 的一致性.   

那么我们一般什么时候会出现 nullability 方面的错误呢?

* Objective-C 与 Swift 混编的场景
* 在代码中有一些逻辑错误
* 不正确的注释

我想看到这,很多朋友都会对这个功能嗤之以鼻, 并且想着"我只要不使用与 nullability 相关的关键字, Static Analyzer 就肯定不会报错啦.", 确实从某种角度来说, 你这么干以后, Static Analyzer 确实不会报错了,但这样真的好么? 

这就回归到为什么我们需要使用与 nullability 相关的关键字这个问题上, 我认为主要的原因有三个:

* 便于跟 Swift 的交互
* 方便使用者明白开发者的意图
* 能够将一些不必要的问题提前到编码阶段, 而不是到用户使用时才暴露

估计有的朋友会对我的第三个观点不太理解, 不用在这里纠结, 下面的这个例子会解释我的想法.  

首先看这段代码, 我们假设他的使用场景如下, 这是一个类似地理位置的抽象类, 对于这样的类,它可以有一个方法来描述它所在的城市或者国家, 这个方法看起来是没有任务错误的, 但其实里面是有缺陷的, 现在假设我们在大西洋的某个不知名的海域中, 由于这个地方既不属于某个城市, 更不属于某个国家, 那么由于 `name` 的初始值为 `nil` , 那么他的返回值一定为 `nil`, 这就与 API 设计者规定的 `nonnull` 相互冲突了, 万幸的是 Static Analyzer 帮我们检测到了这一切. 

![08](08.jpg)

但假如我们没有使用 nonnull 关键字呢? 那么这段话本来是要用于展示在某个 label 上的,但由于返回值为空, 屏幕上空空如也, 用户好几脸懵逼, PM 和 QA 的同事火速杀到你的工位前......
总之,不用我说,你应该能明白我意思了, 这就是我说的:

>能够将一些不必要的问题提前到编码阶段, 而不是到用户使用时才暴露

我们再来看一个例子说明下不正确的宏注释产生的问题,在`NS_ASSUME_NONNULL_BEGIN` 和 `NS_ASSUME_NONNULL_END` 之间的属性都会被默认为 `nonnull` 类型, 那么看下面的代码: 

![09](09.jpg)

在日常的工作中,我们经常是从某个人手里接过来一段代码进行开发, 假设在这个文件里, 由于整个代码已经到了近 1000 行, 且有好几个类在同一个 `.m` 文件中, 所以两个宏写的非常隐蔽, 你根本没有察觉到它们的存在.  

然后你很自信的声明了一个再普通不过的的 `pressure` 属性, 并重写了它的 `get` 方法, 同时我们的逻辑很清楚, 这是一个人拥有的方法, 如果他有压力计的话就测量一下压力,并返回这个压力值,如果没有压力计的话,就返回 nil , 这个逻辑看起来是如此的正确, 但你一运行就 crash , 是不是很崩溃. 

好在 Static Analyzer 告诉了我们问题的关键, OK, 这个 bug 很快就能解决了.

## Runtime Issue

说完了Static Analyzer, 我们来说说 Runtime Issue 这个东西,就像下面这个图展示的一样, 你可以认为以后见到这个紫色的感叹号标志就是一个 runtime issue , 哦, 顺道说一下左边的两个分别是 Error 和 Warnning 状态, 右边的两个分别是 Static Analyzer Issue 和 UI Test Failed 的状态, 不同于其余这些东西的出现时间, runtime issue 是出现在程序运行期间的

![10](10.jpg)


目前支持 Runtime Issue 的工具有三个, 分别是 Debug View Hierarchy , Thread Sanitizer 和 Debug Memory Graph , 我们会在下面的话题一个个介绍给大家! 

嗯呐, 总之, Runtime Issue 的话题就告一段落啦.

## View Debugging Enhancements
View Debugging 到底指的是什么呢? 我想各位看英文时候可能有点懵逼, 但看完下面两张图是不是瞬间明白我在说神马了!!!

![11](11.jpg)

![12](12.jpg)

嗯呐, 就是这个功能叫做 View Debugging, 也可以叫做 Debug View Hierarchy, anyway, 你喜欢啥就叫啥吧,

### Enhancements
这个功能在 Xcode 6 就有了, 那么在 Xcode 8 上又有了哪些提升呢? 苹果工程师给出答案是这样的, 

* up to 70% faster snapshots
* layout and transform accuracy
* blur rendering
* navigator filtering
* jump to class
* auto layout debugging

至于第一个改进点,大家可能需要自己去感受,我也确实没有兴趣做一个数据对比, 毕竟 PM 还找我做需求呢...  
那么我们说下后面五个改进点吧.
#### layout and transform accuracy
这到底是说神马呢? 难道是说以前的 layout 和 transform 不准确么? No, No, No, 并不是说以前不准确,而是说现在比以前更精确了, What, 我说的话是不是好绕,还是直接上图吧...

![13](13.jpg)

![14](14.jpg)

![15](15.jpg)

所以说你看出来神马名堂了么? 在 Xcode 8 里能够更加精确的表明这些约束的意义, 例如是否是等比例缩放(第一张的比例值), 是否是权值较低的约束(第二张的虚线段), 是否是一个不绝对相等的约束(第三张的小于等于). 这些在 Xcode 7 里都是没有体现出来的, 总之通过这些标记, 能够让我们更加清晰的了解到这些约束的意义, 而不只是一根实线而已

#### blur rendering
这是说在新的 debug 模式下,我们能够看到 blur 层了. 是不是很美好

![16](16.jpg)

#### navigator filtering
这个东西我觉得还是蛮好使的, 因为原先在 Navigator 里找某个控件时, 真的很难, 尤其在那种结构复杂的界面里, 就看着自己点着那个三角按钮一遍又一遍的... 

Xcode 8 在今年很好的解决了这些个问题, 我们现在在 Navigator 中可以通过控件的内存地址来定位, 也可以通过它的类名来定位, 甚至可以使用控件中展示的 String 内容来定位. 

这样一来, 找控件就变得很 easy 了, 是不是!!! 不信, 你看下面的这张图. 

![17](17.jpg)

#### jump to class 
这个新增的功能充分体现了苹果工程师的人文关怀, 试想一下, 我们每次在定位到对应的控件后, 如果想要修改其 layout 的相关属性时, 有些人会到左边的 Project Navigator 中的层级结构里找对应的`.m`或者`.h`文件, 熟悉快捷键的人可能会用 `Cmd + Shit
 + O `直接跳转到对应的类中, 总之,你都得想想这个东西的类名, 并输入点字符神马的, 很麻烦.
 
但在Xcode 8 之后, 我们只需要去 UI 控件的 Object Inspector 的界面里点一下右边深灰色的前进按钮, 嗖的一下,我们就跳转到了对应类的文件中

![18](18.jpg)

#### auto layout debugging
这个功能就要结合之前的 runtime issue 话题了, 废话少说, 先上个图给你们瞅瞅.

![19](19.jpg)

我们可以看到,如果我们在布局控件中有错误的话, 我们点击 Debug View Hierarchy 后, Xcode 8 就会报出来一堆 Runtime Issue , 这个功能是不是很吊, 以后写约束再也不怕不怕啦, 毕竟有错, 咱们就按提示改呗. 

### Debug Workflow
这一块的内容并不是某个 Session 里提到的, 而是我在看这些个 WWDC 后总结出来的, 你可以发现苹果的工程师在解决这些问题时, 都是有一个套路的, 套路的英文我也不知道是啥, 就用个 workflow 吧.
他们在解决带有 runtime issue 的问题时, 都会遵循这样一个解决思路

1. Activity Viewer : 查看 Activity Viewer 上是否有错误提示
2. Issue Navigator : 在 Issue Navigator 上初步了解错误的类型
3. Debug Navigator : 在这些界面上了解其层级结构, 调用顺序, 堆栈信息, 对象持有的层级结构图等信息
4. Inspector : 查看具体的细节,并分析错误的原因
5. Source code : 使用 jump to class 功能进入源代码,并修改

至于这个东西, 我觉得可能需要大家自己在实践中慢慢体会, 才会更深入的理解为什么会有这些 debug 工具的产生和为什么他们要在这里提示. 
当然这也是个仁者见仁,智者见智的问题, anyway 你若安好, 便是晴天! 

## Memory Graph Debugging
讲完了 View Debugging Enhancement , 我们来说说今年 Xcode 8 推出的 Memory Graph Debugging.

最近看到很多公众号和微博都有朋友在说这个特性, 我在这里就不花费太多的篇章去讲它, 更多的说说我觉得在其他文章里没提到的东西吧.

在说这个东西之前, 不知道大家是否知道以下三个命令, 如果没有大家不妨在自己的机子上试一试

```s
$ heap YourAppName
$ leaks YourAppName
$ malloc_history YourAppName Address
```

好吧,假设看到这时, 你已经按照我说的那样,按使用了上面的几个命令, 那么下面我就得告诉你一个真相.  

其实 Memory Graph Debugging 就是把这样的一套东西变成了UI界面而已.

### How to use
那么我们接着说说如何使用它吧, 它的使用方式很简单, 在 app 运行的时候, 点击 Debug View Hierarchy 按钮旁边的 Debug Memory Graph 按钮即可, 对就是那个三个圆圈两个线的按钮.

![20](20.jpg)

哦, 对了如果你想看到对象的 malloc_history, 记得在 Diagnostic Scheme Tab 页面里面选择 Malloc Stack , 否则你是看不到任何信息的, 命令行也是如此, 另外, 苹果的工程师还说如果勾选了 Malloc Scribble, 整个结果会更加精确

![21](21.jpg)

那么我们来看看点击 Debug Memory Graph 按钮后的效果吧

![22](22.jpg)

通过这段时间的使用呢, 大致总结出来这样的一些规律

* 绿色的一般都是 UIKit 控件及其子类
* 蓝色一般 NSObject 类及其子类
* 黄色一般都是容器类型及其子类
* 灰色括号是指 block

当然还有很多一些其他的类型,具体的大家去看右上角的 Memory Inspect 界面就好,上面都会有详细的信息

另外这一块还要跟大家交流的就是在 Session 410 中, 苹果的工程师说了一些内容, 希望开发者们在使用 Memory Graph Debug Tool 时能够知道:

1. 为了避免误报内存泄漏的问题, 苹果在展示 Memory Graph 时, 增加了一些引用, 这些引用只是为了避免误报
2. 在 Memory Graph 所有的强引用都是黑色实线, 而灰色实线并不是弱引用, 只是一些系统级别的引用或者苹果为了优化显示效果而添加的, 就像上面第一条说的那样, 所以在看到灰色的引用时, 就自动忽略它吧
3. 关于运行效率方面, 苹果工程师也说了,如果只是为了看 Memory Graph , 在 Malloc Stack 选项中, 直接选择 `Live Allocations Only` 即可, 这样会避免过多的性能消耗
4. 另外, Swift 3 在 Memory Graph 上的表现要相对较好一些
5. Sanitizer 与这个功能不能同时开启, 至于为什么, 你自己看完这几个 session 就会明白了


### `.memgraph` file
另外 Xcode 8 专门提供了一个文件格式来保存某一时刻 app 的 Memory Graph, 当然这个文件你是没法 run 起来的, 它只是个graph, 你要明确这一点.

对于喜欢命令行的小伙伴来说, 苹果还提供了一下的操作指令

```s
$ leaks --outputGraph=<path> <process>                    //creates .memgraph file
$ {leaks|vmmap|heap} <path/to/file.memgraph> [options]    //operate on .memgraph file
```

## Sanitizer
### What does Sanitizer mean?
Santize在英文里面有美化, 优化的意思, 可想而知 Sanitizer 就是一个用于优化的工具.  
那么 Xcode 中的 Sanitizer 到底是什么呢? 在 WWDC 2015 
Session 413 中, 苹果的工程师给出以下条目来介绍 Sanitizer:

* Find bugs at run time
* Similar to Valgrind
* Low overhead
* Work with Swift 3 and C/C++/OC
* Integrated into Xcode IDE

那么到底 Sanitizer 在 Xcode 里怎么使用呢? 其实很简单, 打开 Product -> Scheme -> Edit Scheme, 就会弹出如下的界面, 我们在 Diagnostics 中能够看到这样一个标题 `Runtime Sanitization`, 在它下面有 `Address Sanitize` 和 `Thread Sanitizer` 两个选项, 我们只需要勾选相应的 Sanitizer 即可.

![23](23.jpg)

说到这里还必须多说几句, 此处如果你只是勾选了相应的选项并不代表你就能使用 Sanitizer 来 Check 代码了, 你还必须重新 run 一下代码, 为什么呢? 

这就必须说说整个代码 build flow 了. 如下图所示, 通过勾选了对应的选项, Xcode 会向 clang 传递一个特定的参数, 然后生成一个独特的 binary, 然后这个 binary 会和 Thread Sanitizer 或者 Address Sanitizer 的 dylib 链接在一起. 这样 Sanitizer 就实现了它想要达到的功能. 

![24](24.jpg)

至于每个 Sanitizer 的实现原理, 我这里就不过多描述了, 建议大家直接观看 WWDC 2015 Session 413 ( Address Sanitizer ) 和 WWDC 2016 Session 412 ( Thread Sanitizer ) , 我们这里还是着重介绍它们的使用方法和使用场景.

总之, 你需要记住的就是, 在使用 Sanitizer 的时候, 要重新 Run 一下代码哦.

### Address Sanitizer ( ASan ) 
ASan 其实是 Xcode 在去年新增的一个功能, 它主要用于检测一些内存方面的错误, 在 Xcode 8 里, ASan 已经全面支持了 Swift, 这应该是它唯一新增的一个功能.

那么 ASan 到底能检查哪些类型的错误呢? 苹果工程师列举了以下六种:

* Use after free 
* Heap buﬀer overﬂow 
* Stack buﬀer overﬂow 
* Global variable overﬂow 
* Overﬂows in C++ containers 
* Use after return

哦对了, 苹果的工程师还说后面四种是 ASan 独有的功能, 当然说这话的时候是 2015 年, 不知道 2016 年的时候, 其他的 debug 工具有啥进步没.

说了这么多,咱们来看看下面这段代码吧.

![25](25.jpg)

大家应该能够看出来如果用 `buffer[80]` 的话是会产生数组越界的问题, 虽然 malloc 了 80 个位置,但起始位置是从 0 开始的.   

但现实呢, 这段代码在不开启 ASan 的状况下, 百分之九十九都不会产生 crash , 而且产生 crash 的时候也不会像图中红色文字那样明确的告诉你这是一个 Heap buffer Overflow 问题.

这就是 ASan 的作用, 所以如果再遇到内存问题, 不用再诚惶诚恐的改完代码后使用哎弥陀佛 Cmd + R 大法了, 是不是很 nice!

### Thread Sanitizer ( TSan ) 
最近发现不少公众号和微博在说 Xcode 8 的新特性时, 都在说 Debug View Hierarchy 和 Debug Memory Graph 的相关内容, 但说实话, 我觉得今年 Xcode 8 最令人兴奋的就是添加了 Thread Sanitizer 这个功能, 说真的, 这个功能太有用了, 为什么呢?

让我们想想自己在调试线程方面的 bug 时, 有哪些令人记忆深刻的东西:

* 线程方面的 bug 对时间很敏感, 这就导致很多线程的 bug 极难复现, 复现都成问题, 还怎么改 bug 
* 由于线程的抽象概念导致在 debug 时候也比一般的 debug 更费劲儿, 这时候总觉自己脑子不够使
* 有时候, 由于线程引起的 crash 或者 error ,让我们根本意识不到这其实是线程出了问题

相信上面的三点总会有一个让你刻骨铭心......

那么我们赶紧说说怎么开启 TSan 来帮我们检查线程问题吧. 喂喂, 这个咱们就不再说一遍了吧, 记得看 ASan 章节里的那个图片, 在 Address Sanitizer 下面就行 Thread Sanitizer 啦.

至于 Thread Sanitizer 下面的那个 `Pause on Issues` 的选项就是说, 如果你想一个一个看 runtime issue 就勾选它, 如果你不想这样, 就不要勾选它, 具体是个神马感觉, 你自己试试喽.

如果你喜欢使用 Comman-Line ,那么请记住下面的代码

```
//Compile and Link with TSan
$ clang -fsanitize=thread source.c -o executable
$ swift -sanitize=thread source.swift -o executable
$ xcodebuild -enableThreadSanitizer YES

//Stop after the first error
$ TSAN_OPTIONS=halt_on_error=1 ./executable
```


哦, 还要加一句, TSan 现在只支持 64为 macOS, 以及 64位的 iOS 和 tvOS 的模拟器, 并不支持真机调试和 watchOS.

那么 TSan 作为一个能够检查线程错误的工具, 它现在能检查哪些类型的错误呢? 苹果给出的答案如下:

* Use of uninitialized mutexes
* Thread leaks (missing `phread_johin`)
* Unsafe calls in signal handlers (ex:`malloc`)
* Unlock from wrong thread 
* Data race

那么我们拿下面的这段代码来举例:


```objc
- (void)viewDidLoad {
    [super viewDidLoad];
    [self resetStatue];
    pthread_mutex_init(&(_mutex), NULL);
}

- (void)resetStatue{
    [self acquireLock];
    self.dataArray = nil;
    [self releaseLock];
}

- (void)acquireLock{
    pthread_mutex_lock(&_mutex);
}

- (void)releaseLock{
    pthread_mutex_unlock(&_mutex);
}
```

这段代码的意思是,我们在 viewDidLoad 方法里面重新 reset 自己的状态, 为了防止多个线程去访问同一个 dataArray 属性, 造成 data race 的状态, 我们在 resetStatus 的时候需要加锁, 但当前代码中,我们实际上调用的是一个没有初始化的锁 ( init 方法在 resetStatus 方法下面哦) , 但这段代码在实际运行的过程中,百分之九十九也不会出现 crash, 但有了 TSan 后, 我们来看看发生了什么变化

![26](26.jpg)

发现 runtime issue 的标志了么! 看不清啊,那我们把左边的 Issue Navigator 放大一下

![27](27.jpg)

发现没有,在 Issue Navigator 中, TSan 明确的告诉了我们错误的类型, 而且把线程中的历史信息都记录了下来以便我们分析并解决这个问题, 有没有很贴心!

通过这个问题, 你发现了什么问题么?

* 首先 TSan 并没有触发 Crash 并且成功捕获到了这个线程问题, 这说明 TSan 不需要我们去复现 crash 来捕获错误
* 然后你在不同机型, 不同环境下运行带有线程错误的程序, 你会发现每次 TSan 都能捕获到这个 bug, 这说明 TSan 对时间是不敏感的.

这样安全可靠的 debug 工具你怎能不爱呢! 

说道这, 不妨我们多说一点, 大家都知道 data race 是线程中最常出现的问题, 造成 data race 的情况无非就是两种, 一种是逻辑错误, 一种是没有加锁. 在这里我特别想分享一个我自己在使用 TSan 时编写的小 Demo. 

我们先看一下代码的使用场景, 我们假设有三个售票员在卖票, 票的数量由 `ticketsCount` 决定, 同时我们将售票员抽象成一个线程类:

```objc
    self.ticketsCount = 100;
    
    NSThread *thread1 = [[NSThread alloc] initWithTarget:self selector:@selector(saleTicket) object:nil];
    thread1.name = @"售票员1";
    self.thread1 = thread1;
    
    NSThread *thread2 = [[NSThread alloc] initWithTarget:self selector:@selector(saleTicket) object:nil];
    thread2.name = @"售票员2";
    self.thread2 = thread2;
    
    
    NSThread *thread3 = [[NSThread alloc] initWithTarget:self selector:@selector(saleTicket) object:nil];
    thread3.name = @"售票员3";
    self.thread3 = thread3;
```

然后我们执行下面的这段代码

```objc
    for (NSUInteger count = self.ticketsCount; count > 0; count--) {
        if (count > 0) {
            [NSThread sleepForTimeInterval:0.1];
            NSString *name = [NSThread currentThread].name;
            dispatch_async(dispatch_get_main_queue(), ^{
                self.ticketsCount = count - 1;
                NSString *string = [NSString stringWithFormat:@"%@卖了一张票, 还剩%zd张票", name, self.ticketsCount];
                self.ticketsCountLabel.text = string;
                NSLog(@"%@", string);
            });
        } else {
            dispatch_async(dispatch_get_main_queue(), ^{
                NSLog(@"票卖完了");
            });
            return;
        }
    }
```


会发生什么呢? 显而易见,由于没有加锁, 售票员会卖出去不该卖出去的票

![28](28.jpg)

那我们加个锁试试?

```objc
    @synchronized (self) {
        for (NSUInteger count = self.ticketsCount; count > 0; count--) {
            if (count > 0) {
                [NSThread sleepForTimeInterval:0.1];
                NSString *name = [NSThread currentThread].name;
                dispatch_async(dispatch_get_main_queue(), ^{
                    self.ticketsCount = count - 1;
                    NSString *string = [NSString stringWithFormat:@"%@卖了一张票, 还剩%zd张票", name, self.ticketsCount];
                    self.ticketsCountLabel.text = string;
                    NSLog(@"%@", string);
                });
            } else {
                dispatch_async(dispatch_get_main_queue(), ^{
                    NSLog(@"票卖完了");
                });
                return;
            }
        }
    }
```

加锁后,我们发现售票员确实没有再卖出去不该卖的票,但是好像只有一个售票员在卖票.  

![29](29.jpg)

这显然是一个逻辑错误, 我们锁住更像是线程, 而不是资源, 所以我们再改进一下

```objc
    while (1) {
        @synchronized (self) {
            if (self.ticketsCount > 0) {
                [NSThread sleepForTimeInterval:0.1];
                NSString *name = [NSThread currentThread].name;
                self.ticketsCount = self.ticketsCount - 1;
                dispatch_async(dispatch_get_main_queue(), ^{
                    NSString *string = [NSString stringWithFormat:@"%@卖了一张票, 还剩%zd张票", name, self.ticketsCount];
                    self.ticketsCountLabel.text = string;
                    NSLog(@"%@", string);
                });
            } else {
                dispatch_sync(dispatch_get_main_queue(), ^{
                    NSLog(@"票卖完了");
                });
                return;
            }
        }
    }
```

来看看打印台的结果

![30](30.jpg)

看起来是不是特别完美! 不同的售票员在卖票, 而且也没有出现卖出去不该卖的票, 但是其实这段代码还是有潜在风险的. 

What? What? What? 怎么会有错...

这时候我希望你能打开 TSan 来分析下潜在的风险在哪里吧, 

PS: 如果你比较懒的话, 好好想想 `dispatch_async` 用的对么? 如果我在这里么加入了一些危险操作 ?

说到这, 你是不是又一次被 TSan 强大的功能震撼住了, 说真的, TSan 真的是 Xcode 8 里一个非常强大的新功能, 它能够帮我们察觉到很多很多我们自己完全意识不到的细小问题, 而在这些问题经常会弄得我们在 debug 时候苦不堪言, 所以从今天开始, 在你编程的时候, 用一下 TSan 吧

## Recap

总结一下今天我们到底说了些什么:

* 一种新的文件格式 : `.memgraph`  
* 两个新的概念 : `Debug workflow` 和 `Runtime issue`  
* 三类 debug 工具 : `Sanitizer`, `View Hierarchy Debug Tool` , `Memory Graph Debug Tool`

最后呢, 想说的很简单,  Use this tools on our project and Make our app better than ever

## Reference Material
[WWDC 2016 Session 410 - Visual Debugging with Xcode](https://developer.apple.com/videos/play/wwdc2016/410/) 

[WWDC 2016 Session 412 - Thread Sanitizer and Static Analysis](https://developer.apple.com/videos/play/wwdc2016/412/)

[WWDC 2015 Session 413 - Advanced Debugging and the Address Sanitizer](https://developer.apple.com/videos/play/wwdc2015/413/)

## Demo Code
示例代码可以在[这里](https://github.com/SketchK/SketchK.github.io/tree/blog-code/2016-10-20-NewDebuggingFeatureInXcode8)获得。