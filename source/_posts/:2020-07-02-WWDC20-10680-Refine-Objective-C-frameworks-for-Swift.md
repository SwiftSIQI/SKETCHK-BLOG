---
title: WWDC20 10680 - Refine Objective-C frameworks for Swift
comments: true
date: 2020-07-02 20:25:20
updated:
tags:
  - WWDC
  - Swift
  - Objective-C
categories:
  - Swift
---

每一年的 WWDC 里都会有一些类型 Apple 工程师教你如何写代码的 Session，这些 Seesion 的内容都偏向最佳实践，告诉你如何写出 Apple 风格的代码，解答你对代码里的各种疑惑，甚至给出你如何继续深入研究的方向，这对开发者来说，是一个非常好的学习机会。

在这个 Session 中，Apple 的工程师将告诉我们如何改造现有的 Objective-C 框架，使其能够更符合 Swift 的使用体验，所以你不仅能学习很多实际的技巧，也会进一步了解他们背后的思考。

话不多说，来看正文吧！

<!-- more -->

## 引子

每一年的 WWDC 里都会有一些类型 Apple 工程师教你如何写代码的 Session，这些 Seesion 的内容都偏向最佳实践，告诉你如何写出 Apple 风格的代码，解答你对代码里的各种疑惑，甚至给出你如何继续深入研究的方向，这对开发者来说，是一个非常好的学习机会。

在这个 Session 中，Apple 的工程师将告诉我们如何改造现有的 Objective-C 框架，使其能够更符合 Swift 的使用体验，所以你不仅能学习很多实际的技巧，也会进一步了解他们背后的思考。

话不多说，来看正文吧！

### 背景介绍

相比于六年前推出的 Swift 语言，Objective-C 在 Apple 生态圈的历史更为悠久，导致了历史包袱比较重的或者现有的工程中还会持续存在许多 Objective-C 的框架，这些框架不是孤立的，会与 Swift 的框架产生依赖关系和调用关系。而这种微妙的关系产生了许多棘手的问题。

![IMAGE](01.jpg)

不光社区里的开发者会遇到这样的问题，Apple 公司的工程师也无法例外，在 Session 里，Brent 用这样一段话来形容这个问题，我感觉很贴切:

> We understand that, because Apple is in the same boat. We probably have more Objective-C frameworks than anyone in the world.

所以如何让 Objective-C 框架更好的为 Swift 服务也是他们要解决的问题之一，虽然 Swift 编译器在转换 Objective-C 接口时做了很多不错的优化工作，但很难满足所有开发者的期望，不过这不代表你没有办法去优化它，因为今天的 Session 就是做这个事儿的。

从改变途径来看的话，主要是通过以下几种方式：

* 遵循编译器的某些规则
* 在头文件里进行特殊标注
* 用 Swift 做中间层，重新封装原有代码
* 根据自己的喜好进一步优化

### 知识目录

![IMAGE](02.png)

## Demo 工程

这个 Session 是围绕一个用于描述 NASA 载⼈航天计划的 SDK 展开的，这个 SDK 的名字叫做 SpaceKit，它是 Objective-C 编写的。

现在这个 SDK 会被一些 Swift 代码调用，所以我们要通过一些改造，使其更符合 Swift 的使用习惯。

### 如何查看编译器生成的 Swift 接口

考察一个 Objective-C SDK 是否符合 Swift 的使用习惯，最重要的一点就是看它生成的 Swift API 质量。那么，我们如何查看 Swift Compiler 自动生成的接口呢？

在 Objective-C 的头文件里，点击左上角的 Related Items 按钮，选择 Generated Interface 后，就会出现满足不同 Swift 版本的接口文件。

![IMAGE](03.jpg)

点开后，它的样子大体如下

![IMAGE](04.jpg)

这个功能对于我们理解如何生成更符合 Swift 使用习惯的 API 来说是非常重要的，所以希望你能掌握这个技巧！

## 自动生成的接口利与弊

下面是根据 Objective-C 源码自动生成的 API 接口，我们可以看到 Swift 编译器已经做了不少的优化，例如：

* 将 NSString，NSDate 类型转换成了 String，Date；
* 将 Objective-C 里的初始化方法转换成了 Swift 里的构造器方法；
* 将原有的 `- (NSSet *)previousMissionsFlownByAstronaut:(SKAStronaut *)astronaut` 的方法名优化成了 `previousMissionFlown(by astronaut:)`
* 将原有的 `-(BOOL)saveToURL:(NSURL *)url error:(NSError **)error` 的错误处理 API 改成了 Swift 风格的 API

![IMAGE](05.jpg)

但这样的 API 接口还存在多问题，让我们列举一下：

* SKMission 的问题：
  * 过多的隐式解析可选类型
  * `crew` 属性里的 Any 定义过于模糊
  * `save(to url:)` 可能会在不该抛异常的时机点抛异常
  * `previousMissionFlown(by astronaut:)` 的方法名还不够优雅

* SKAstronaut 的问题：
  * 构造器之间关系不够清晰

* SKErrorDomain 和 SKErrorCode 的问题：
  * NSError 风格的 API 在 Swift 里的使用体验非常不好，尤其在 try catch 中

* SKCapsule 和 SKRocket 类型的常量
  * 用于枚举的字符串常量在 Swift 里更适合使用 enum 类型来描述
  * `SKCapsuleApolloCSM` 的 API 消失了
  * `SKRocketStageCount(_ rocket: String!) -> Unit` 的 API 还有不少潜在的风险

如果你还看不出上面存在的所有问题，也无法提供所有问题的解决方案，那么这篇文章将十分适合你阅读。

所以让我一起来看看 Apple 工程师给出的解决方案吧！

## 改进的方法

如果想解决上面提到的各种问题，可以从下面四个方向入手：

* 提供更丰富的类型信息
* 遵守 Objective-C 的约定
* 解决缺少 API 的问题
* 改善框架在 Swift 里的使用体验

### 提供更丰富的类型信息

#### 增加 nullability 的描述信息

Objective-C 指针既可以是一个有效值，也可以是空值，例如 null 或者 nil，这与 Swift 里的可选值行为十分相似。

如果我们再仔细想一下，就会发现在 Objective-C 里面，每个指针类型实际上都是可选类型，每个非指针类型都是非可选类型。可是大部分时间，一个属性或者方法不会处理输入值是 nil 的情况，或者永远不会返回 nil。

所以，默认情况下 Swift 会把 Objective-C 里的指针当做隐式解析可选类型，因为它认为这个值大部分情况下不会是 nil，但它也不完全确定。

虽说这种转换规则没什么毛病，但大量的隐式解析可选类型让代码变得意图模糊，好在我们有两个关键字注解可以去描述这个意图，他们分别是 nonnull 和 nullable

这两个注解在 Objective-C 里面只是用于记录开发者的意图，不是强制的。但 Swift 会用到这些信息来决定是否转换为可选类型。

另外需要注意的是，在标注完 nullability 后，原有的 Objective-C 代码可能会出现一些新的警告，这里请认真检查并按照提示进行修改，这会让你的代码更健壮。

![IMAGE](06.jpg)

除了 nonnull 和 nullable 意外，还有一对配合使用的宏 `NS_ASSUME_NONNULL_BEGIN` 和 `NS_ASSUME_NONNULL_END` 可以让我们的代码更清爽。

在这两个宏包裹的代码片段中，属性，⽅法参数和返回值的默认注解都是 nonnull 类型的，这样一来，我们就可以删掉许多冗余的代码。

![IMAGE](07.jpg)

但是这些方法并不适用所有的场合，例如你将 nonnull 直接放在常量前会触发编译器错误。还好这种错误是有解决办法的！

nonull 和 nullable 只能在方法和属性上使用，如果想拓展其使用场景，就需要直接调用这两关键字底层的内容，也就是 `_Nonnull` 和 `_Nullable`。

这两种注解除了可以用在全局常量，全局函数的场景外，也适用于任何 Objective-C 任何地方的指针类型，甚至那种指向指针类型的指针。

![IMAGE](08.jpg)

现在我们看到 SKRocketSaturnV 终于如期所愿的摆脱了隐式解析可选类型！

![IMAGE](09.jpg)

然后我们看看下面的 API 可能存在的问题，在这里我们从 API 层面假设 capsule 是一个非空值，但可能这是不合理的，并不是每次的飞行计划都需要载人，不是么？

![IMAGE](10.jpg)

那么，如果 Objective-C 返回了⼀个 nil 值，⽽在 API 层⾯，Swift 认为这是⼀个⾮空值，又会发⽣什么呢？

如果是 NSString 或者 NSArray 的话，Swift 会得到⼀个空的字符串或者数组，这可能会引起一些问题，但对于其他类型，可能会拿到⼀个⽆效的对象，总之，可能与你的预期不⼀样。

如果是 Objective-C 对象，你可能很难注意到这⼀点，因为 Objective-C 会忽略 nil，但在某些 case 下，你可以会因为 null 指针崩溃或者触发异常⾏为。

编译器不会对这种⾏为作出任何的承诺，所以改变 release mode 或者 xcode 版本可能有不同的表现！ 

不论怎样，需要记住的是，当你头⽂件⾥某个东西不会是 nil 的时候，Swift 不会对其强制解包，所以你不会在返回 nil 的地⽅看到崩溃。

![IMAGE](11.jpg)

那么对于这种 case，就没有解决办法了么？

好消息是 Objective-C 编译器和 Clang 的静态检查能够很好的解决这个问题，所以在写好 nullability 的注解后，最好关注⼀下编译器警告和静态分析结果！

就如下图所示一样

![IMAGE](12.jpg)

当然我们知道，开发者可能还会遇到一些特殊的 case，在这些 case 里，他们没法确定代码到底是有值，还是没有值，所以 Apple 还提过了 `_Null_unspecified` 的注解词。

![IMAGE](13.jpg)

被 `_Null_unspecified` 标注的内容会被转换成隐式解析可选类型，这种类型在 Swift 里的使用场景大体如下，例如某个属性在其⽣命周期早期为 nil，之后再不会是 nil 的情况。

当然，在你⽆法确定的 case 里也可以这样使用，因为

* 如果一切按照预期，你可以⽆需解包，继续使⽤ 
* 如果返回的是 nil，你会稳定的复现这个 bug，⽽不是⼀些奇怪的⾏为

#### 利用泛型约束接口

原有的接口中，没有对 crew 这个数组里的元素进行约束，这会使得其转换到 Swift 的 API 时，将其中的元素描述为 Any。

虽然也不是什么大的毛病，但用起来确实会显得有点别扭！

![IMAGE](14.jpg)

我们都知道 Objective-C 也提供了一些泛型的能力，所以我们完全可以将其优化到一个更好的层次上，通过在 Objective-C 里添加相应的语法内容，就可以将其在 Swift 的使用体验改善不少。

当然除了 NSArray，NSDictionary 等基本类型也适用这个技巧！

![IMAGE](15.jpg)

#### 对于数字处理统一使用 Int

我们先看看这样一段代码，下⾯的函数返回⼀个计数值，很显然，你在喊倒计时的时候，数值不会为负，所以在 Objective-C ⾥⾯以 NSUInteger 的形式返回。

这样的声明，意味着在 Swift 里会返回⼀个 UInt，而这意味打破了 Swift 的使用习惯。

![IMAGE](16.jpg)

当我们想对比特位进行运算的时候，我们通常会使用 unsigned 类型的数据，因为 signed 的数据在处理起来会有些麻烦，而且在这种场景下，我们还十分关注数据的位数，但是由于 NSUInteger 的大小会因架构不同而产生一些变化，导致使用它的人并不多。

![IMAGE](17.jpg)

与此初衷不同的是，大多数人使用 NSUInteger 是为了表明这个数值是⾮负的，虽然这种用法是可行的，但它还是会存在一些严重的安全漏洞，所以这种设计思路并没有被 Swift 采用。

Swift 采取的策略是在进⾏有符号运算时，要求开发者必须将⽆符号类型转换为有符号类型，如果 Swift 在处理⽆符号运算时，产⽣了负值，就会直接停⽌运算。

也正是这样的策略，会让 Swift 中的 Int 和 UInt 在混合起来使用的时候变得很麻烦，当然，这在 Objective-C ⾥⾯的也是一个棘手的问题。

所以混合使用 Int 和 UInt 并不是 Swift 里的最佳实践，在 Swift 里面，我们建议将所有进行数值计算的类型声明为 Int，即使它永远不可能为负数。

![IMAGE](18.jpg)

对于 Apple 自己的框架，他们设置了一个白名单用于将 NSUInteger 转换为 Int。

对于开发者而言，决定权在我们自己手里，我们可以⾃⾏选择是否使⽤ NSInteger，但 Apple 的工程师强烈推荐你这么做。

或许在 Objective-C ⾥⾯差距不是很⼤，但在 Swift ⾥⾯很重要！

#### 将字符串类型的常量变得更有条理

下面我们来看看这样一段代码，从某种角度上来说，`SKRocketStageCount` 这个 API 很容易被滥用，因为只要传一个字符串就行了，但其实我们希望传入的是以 `SKRocket` 为前缀的常量字符串。

可惜 Swift 无法感知这一切，它能看到的只是函数需要的是字符串⽽已，如果传了其他值的字符串，就会出现不符合预期的情况。

![IMAGE](19.jpg)

在 Swift ⾥通常会把这些常量变成⼀个具有字符串原始值的枚举或者结构体，然后改变函数的入参类型，使其接受相应的枚举或者结构体类型。

那么我们怎么去改造这个接口呢？我们先说个最简单的方法：

使⽤ typedef 将常量分组，并将涉及此常量的地⽅改为新的类型。而在 Swift 中，typedef 会被转换成 typealias

![IMAGE](20.jpg)

这已经使得代码发生了一些变化，不过这还不是最终效果！

此时，你只需要在 typedef 后⾯加上 `NS_STRING_ENUM` 即可, 此时，原有的字符串常量将以结构体的⽅式导⼊到 Swift 中, 而且，你注意到没有，`SKRocketStageCount` 的⼊参类型彻底的变了！

![IMAGE](21.jpg)

怎么样，一共就 2 步，就能得到原汁原味的 Swift API，是不是还不错！

这样的使用方式，可以在 Apple 的框架里看到不少实实在在的例子，例如 NSAttributedStringKey，NSCalendarIdentifier，NSNotificationName，NSNotificationUserInfoKey 等。

所以放心使用它吧！

### 遵守 Objective-C 的约定

#### 关于构造器的相关约定

接下来，我们来看看构造器方面的问题。

下面的代码中，SKAstronaut 有两个初始化构造器，⼀个入参类型为 PersonNameComponents, ⼀个入参类型为字符串，

这就意味着，如果要声明⼀个 SKAstronaut 的⼦类，就需要重写两个⽅法，但 NSPersonNameComponents 本质也代表一个字符串，所以这样的工作显得有点多于。

![IMAGE](22.jpg)

同时，你还会在使用的时候发现，莫名其妙的多出来一个构造器，它没有任何的入参，我们在源⽂件⾥找不到任何与此相关的定义，但其实它来⾃ superclass。 因为 SKAstronaut 继承⾃ NSObject。

虽然有这么一个方法，你也能调⽤，但很可能它⽆法正常⼯作！

![IMAGE](23.jpg)

如果你深入分析上面的两个问题会发现，它们的内核是一样的。

在 Objective-C 中，有⼀个关于初始化器的约定，它确保开发者知道如何写⼀个总是能被正确初始化的⼦类。

这个约定的大体内容是这样的，将初始化器分为两类，designated 和 convenience。 你需要覆盖所有 designated 初始化器，以便安全地继承 convenience 的初始化器。

![IMAGE](24.jpg)

这个约定和 Swift 里面的构造器约定十分相似，但它们有个本质的区别！

Objective-C 的这种构造器约定不是语⾔级别的强制规则，更多的是⼀个开发者之间的约定，例如 convenience 必须选择⼀个 designated 的接口，但实际上很多 Objective-C 的类并没这么做，这也意味着如果有⼦类的话，如何正确构造它会成为⼀个头⼤的问题！

这是⼀个⾮常⾮常不好的事情，尤其对框架使⽤者⽽⾔，如果想写出⾼质量的代码，就必须阅读源码，或者逆向来观察它的行为，甚至通过猜测的方式， ⽽这都会导致⼦类出现异常的概率变⼤。

![IMAGE](25.jpg)

如果你忘了重写⼀些必要的构造器，作为框架的维护者是不会收到警告的，⽽使⽤者恰巧使⽤了这个 API，那就意味着这个类的初始化可能出现了问题，使⽤者会感觉很痛苦，为什么写个构造器这么难？

所以作为框架的维护者，我们需要去直面这个问题！

通常 designated 构造器会调⽤ [super init] 这个方法，而 convenience 构造器会调⽤⾃⾝的某个 designated 构造器

![IMAGE](26.jpg)

所以，我们需要在 designated 构造器后面添加 `NS_DESIGNATED_INITIALIZER`, 对于 convenience 类型的构造器，你不需要做任何事情

![IMAGE](27.jpg)

在添加完 `NS_DESIGNATED_INITIALIZER` 以后，可能会遇到一些错误提示，它会要求你重写⽗类的 designated 构造器，因为这是一个潜在的 bug，如果有⼈使⽤了⽗类的 designated 构造器，而你没有对此进行处理，对象的构造就可能会出现问题。

所以

如果你想支持这些父类构造器，就去实现它！

如果你不想支持这些父类构造器，就需要完成如下的工作

* 在 `.m` 文件里重写父类构造器，并调用 `doesNotRecognizeSelector` 方法

![IMAGE](28.jpg)

* 在 `.h` 文件里用 `NS_UNAVAILABLE` 声明对应的 API

![IMAGE](29.jpg)

除此之外，还需要提醒的有两点

* 除了关注父类的 designated 构造器，开发者也需要关注 convenience 类型的构造器。
* `NS_UNAVAILABLE` 的 API 不会被继承

通过这些改造，你的 Swift 接口将会变得清晰明了！

#### 关于错误处理的相关约定

让我们在看看下面的代码

在整个文章的开始部分，我们提到这段代码可能会在不该抛出异常的时候抛出异常，至于原因，其实在这个 API 的注释里就已经说明了。

![IMAGE](30.jpg)

可能有人会问，看起来没什么问题啊？

这其实就是问题本身，可能我们会认为如果⼀个⽅法要发出失败的信号，就意味着必须要返回 false 且为 error 设置⼀个 non-nil 值；如果只返回⼀个 false 并不是真的失败。

但事实上，通常的约定是：返回 false 就是失败！即使 error 是 nil。

![IMAGE](31.jpg)

Apple 的工程师并不建议把 error 设置为 nil，因为这样的话，使用者就无法感知发⽣了什么，但如果你坚持这样做，也就是返回 false 的同时，返回了一个 error 为 nil 的值，从惯用的约定来看，它仍然代表失败！

所以在 swift ⾥调⽤这种 Objective-C 的⽅法时，它会⾃动导⼊ throws，而且 swift 会认为你遵循刚才提到的约定，所以只要⽅法返回 false 它就会抛出异常！

但是 Swift 又不允许你抛出异常的时候，提供的信息是 nil。如果没有 error，swift 会抛出⼀个基础库里⾮公开的 error 类型，由于这是⼀个⾮公开的类型，你是⽆法 catch 这些信息的，但是你可以在 logs ，debugger 或者 error message ⾥看到相关的信息。 

这意味着⼀些 Obective-C 代码即使没有失败也返回了 false，或者失败了但没有告诉你原因。

![IMAGE](32.jpg)

所以回到一开始的那段代码上，⽂档注释⾥说明了在某些情况下，例如有东西需要保存的时候，会直接返回 false，虽然其实这不能算是失败，但是 Swift 会因为 false 抛出异常。由于⽅法没有设置正确的 Error 信息，就会提到我们前⾯说的情况，抛出⼀个基础库里⾮公开的 error 信息。

所以如何解决它呢？

最简单的方法就是去掉特殊情况，让 false 总是意味着失败，⽽且让该⽅法遵守前⾯提到的约定。

![IMAGE](33.jpg)

不过这也引入了新的问题，如果你的用户需要判断那种特殊情况的时候，这种⽅法就⾏不通了！

另外一种方式是，在 API 后面标注 `NS_SWIFT_NOTHROW` 来告诉 Swift 你不想遵守那个约定，这样的话，Swift 会让你⼿动处理错误相关的代码

虽然这解决了问题，但并不是最好的方式，建议将这个 API 废弃，此时你只需要在后面添加 `DEPRECATED_ATTRIBUTE` 即可

![IMAGE](34.jpg)

在 Objective-C 里最好的解决方案是重新写一个 API 并返回额外的信息来表示刚才提到的特殊场景。

例如可以添加⼀个布尔输出参数来说明⽂件是否真的被保存了，然后返回值就可以遵守之前的约定了。

![IMAGE](35.jpg)

目前来看，这是在 Objective-C 里的最佳解决方案了！

如果你还想进一步优化，我们还是有办法的，只是我们需要用 Swift 将其包裹一下并对外保留 Swift 接口，而非原先的 Objective-C 接口。

在 Swift 里，我们可以提供这样的一段代码来优化使用体验

在 Objective-C 里面，我们要小心返回值，因为他的惯用约定会认为返回 false 即是失败，而在 Swift 里，我们则可以忽略这一点，返回值与抛出异常或者失败是完全没有关系的！

所以代码会变成下面的样子！

![IMAGE](36.jpg)

> 为了让这个文章的内容更连贯，我将原 Session 里这部分的内容做了精简，此处还提到了以下几个知识点:
>
> * 在框架的头文件里，也就是 umbrella header 里，不要导入 generated header，也就是系统自动生成的 `<framework>-Swift.h`，这个头文件会声明 swift ⽂件⾥所有被 @objc 标记的内容
>   * 不这么做的原因是会产生循环依赖，因为不导⼊ umbrella header ⾥的内容，Swift ⽆法⽣成 `<framework>-Swift.h`，但如果 umbrella header 导⼊了 `<framework>-Swift.h`，那么 swift 就会试图读取这个还没⽣成的⽂件，而这就会造成问题！
> * 在某些平台上，Swift 的 bool 和 Objective-C 的 bool 略有不同，主要是在内存表示方面的问题。
>   * 通常，swift 会做⼀层转换，但是现在的代码⾥你操作的是指针，也就是将 Swift 的 bool 指针直接扔给 Objective-C 使⽤，而对于这种 case 来说，Swift 还没覆盖，所以我们需要做⼿动转换，声明⼀个 ObjcBool 类型即可

现在我们的 Swift 使用者终于可以使用到非常舒服的 API 了，不过还有一个问题就是，他在这种场景下，依然能接触到 Objective—C 里面提供的新 API，`-(BOOL)saveToURL:(nonnull NSURL *)url wasDirty:(nullable BOOL *)wasDirty error:(NSError **)error`。

他一定会面临这样的处境，我该用哪个呢？

所以作为框架的维护者，你现在的情况是 ：

* 希望 Objective—C 的使⽤者用到原先的 API， 
* 希望 Swift 的使用者用到 Swift 里提供的 API

针对这种情况，你也有相应的解决办法，此时你需要

* 在原有的头⽂件⾥对相应的 Objective—C 的⽅法标记 `NS_REFINED_FOR_SWIFT`

![IMAGE](37.jpg)

* 修改 Swift 里的代码实现

![IMAGE](38.jpg)

可能有人会好奇，`NS_REFINED_FOR_SWIFT` 到底干了什么？

其实这个标记做的事情很简单，它把对应的 Swift 版本 API 进行了改造，改造的内容就是在其开头增加了两个下划线在开头，

当 Xcode 看到这样的 api 时，会让编辑器将其隐藏起来，例如代码补全的时候，但它不代表你不可以调用，所以在刚才的 Swift 文件里，我们看到了 `self.__save(to:url, wasDirty: &wasDirty)` 的代码。

此时，我们仍然使⽤了 Objective—C 的实现，⽽且使⽤者也⽆法直接调取那些我们不想让他获取的 API 了！真是一个让人开心的结果！

### 解决缺少 API 的问题

通常 Swift 编译器会导⼊ Objective—C 头⽂件⾥的⼀切，但如果它⽆法识别如何导⼊的时候，就会忽略这些内容，进而导致某些 API 没有在 Swift 里展示。

这些场景会是如下的这些情况：

![IMAGE](39.jpg)

回到代码上，我们可以看到 SKCapsuleApolloCSM 这个 API 消失了！

![IMAGE](40.jpg)

在这里，我们用宏定义了一些字符串，在这里宏本质上只是⼀个⽂本⽚段，你可以在 Objective—C 源码的任何地⽅使⽤它。

同⼀个宏在不同的地⽅可能有不同含义，而 Swift 本⾝是没法搞清楚这些的，所以 Swift 只能识别符合某些特定模式的宏，这种模式主要是⽤来声明常量的。

它允许你为另外⼀个宏命名，或者给宏设置某个值，但是两者同时使用就会出现识别问题，

所以来看第四个宏，它替换了另外⼀个宏，并在原本的内容上增加了 ".csm"，这对 Swift 来说有点超出其能力范围了！

有许多⽅法解决这个问题，最简单的就是⽤完整的字符串来表达这个宏，而不是用相对复杂的宏拼接。

![IMAGE](41.jpg)

但如果你是⽤这些字符串做枚举，建议你把它转成真正的字符串常量，这样就可以像前面的 SKRocket 常量⼀样进⾏字符 串枚举了！那才是好的最佳实践！

### 改善框架在 Swift 里的使用体验

#### 如何改善 Swift 的 API 名称

Swift 和 Objective—C 的命名风格是有所不同，例如 Swift 的 API 是由基名（previousMissionsFlown）和参数标签（by）组成的，⽽ Objective—C 基本上只有参数标签(previousMissionsFlownByAstronaut)，没有单独的基名，所以基名的信息会包含在第⼀个参数标签⾥，这也导致了 Objective—C 的方法名会显得略长一些。

为了解决 API 风格上的问题，Swift 会根据一些规则重命名，通常这个结果还不错，但这毕竟是计算机的审美结果，很难满足开发者的诉求。

![IMAGE](42.jpg)

例如某些开发者会认为 flown 应该是参数标签⾥的⼀部分，⽽不是 base name，因为这个⽅ 法获取的是以前的任务列表，它们是某个宇航员所执⾏的任务！

> 当然这不是绝对的，这⾥只是个假设。

所以为了解决这个问题，我们使⽤ `NS_SWIFT_NAME` 重新命名这个⽅法

![IMAGE](43.jpg)

好了！这个 API 终于满足你的诉求了！

或许你会说我知道 `NS_SWIFT_NAME` 能重命名方法名，但 `NS_SWIFT_NAME` 的能力还有很多施展空间！

例如下面的枚举！

![IMAGE](44.jpg)

或许，乍一看，这个枚举其实已经写得挺好的了，但其实也有不少改进的空间。

可能你会想到⽤ `NS_SWIFT_NAME` 删除其前缀 SK，因为在 Swift 里面没有这种做法，但不推荐这么做， ⼤部分 Objective—C 的类都会将框架前缀与⼀个像 query 或者 record 的词组合起来，例如 SKFuleKind ⾥的 SK 和 FuleKind。

所以我们需要用别的方法来优化使用体验，针对目前框架，我们有一个 SKFule 类，此时我们可以让这个枚举和 SKFule 联合使⽤，所以我们将其改为 SKFuel.Kind

![IMAGE](45.jpg)

除了重命名枚举外，`NS_SWIFT_NAME` 的另外一个常见使⽤场景是处理那些与 Swift 风格差异过大的 API，这在许多 C 语言的库里十分常见，例如全局函数，全局变量等。

像上面的例子中，SKFuelKindToString 就是一个全局函数，我们不仅可以⽤ `NS_SWIFT_NAME` 对其重命名，还可以去掉额外的信息，添加⼀个参数标签。

![IMAGE](46.jpg)

不过刚才处理全局函数的例子只是展示了 `NS_SWIFT_NAME` 能力的冰山一角！下面我们会再展开几个例子：

⾸先你可以将 global function 转换成 static method，做法是在 `NS_SWIFT_NAME` 里指明 Objective—C 的类型并在 类型后面使用点语法声明⽅法名

![IMAGE](47.jpg)

然后，你还可以将其变为实例⽅法！

![IMAGE](48.jpg)

最后，你也可以将某个⽅法变为⼀个属性，只需要在前⾯增加⼀个 getter，同理 setter

![IMAGE](49.jpg)

将这些技术应⽤在充满 C 函数的框架⾥，可以很好的重塑 API，如果你用过 Core Graphics 的话，你会深有体会！

下⾯要说说 `NS_SWIFT_NAME` 的能⼒边界，例如刚才的那个 getter 例子，即使你将⽅法名改为 description，你也⽆法让这个类型遵守 string convertible protocol。

但是我们可以通过在 Swift 文件里添加扩展来使其满足 protocol conformance！

如下图所示，给 SKFuel.Kind 写了⼀个扩展，并使其符合⾃定义字符串转换协议，由于 Objective-C 头文件已经⽤ `NS_SWIFT_NAME` 提供了相应的属性，所以写成这样，我们的 SKFuel.Kind 已经遵守了相应的协议并满足其使用要求。

![IMAGE](50.jpg)

> 为了让这个文章的内容更连贯，我将原 Session 里这部分的内容做了精简，此处还提到了以下几个知识点:
>
> * 我们应当注意到刚才的 Swift 文件，在那里我们可以写出任何你想提供的 Swift 版本专⽤的 API。例如整合了 UIView 的 SwiftUI 组件，或者将原有 API 的 completion handler 换成 Combine 的 API.
> * 如果对 SwiftUI 和 Combine 感兴趣，可以查看去年的两个相关 Session，[Integrating SwiftUI](https://developer.apple.com/videos/play/wwdc2019/231/) 和 [Combine in Practice](https://developer.apple.com/videos/play/wwdc2019/721/)
> ![IMAGE](51.jpg)

#### 如何提升 Error Code 在 Swift 里的使用体验

error code 枚举在许多框架里都能见到，它通常会和 NSError ⼀起使⽤。

通常这类代码会分为两个部分

* 声明⼀个带有特定错误代码的 `NS_ENUM`
* 为了防⽌错误码与其他框架的错误码发⽣冲突，还会声明了⼀个用于表明作用域的字符串常量。

下⾯的代码在接口层⾯是没有问题的，但在使⽤时，就会出现⽐较明显的问题！

![IMAGE](52.jpg)

假设我们有这样⼀个场景：我们需要考虑在执⾏某次飞⾏任务的过程中，如果发射终⽌了，我们要确保救援队能去营救宇航员！

所以代码会是如下的样⼦：

![IMAGE](53.jpg)

先调⽤发射任务的代码，如果是发射终⽌错误，我们需要对齐进行 catch，然后就是如何 catch 到特定的 case 了, 这里需要 error 中的作用域（domain）和错误码（code）信息

* ⾸先我们要把它作为⼀个 NSError 来捕获。
* 接下来，我们需要确保错误是 SKError 的作用域，⽽不是其他可能使⽤相同错误代码的作用域。
* 然后我们需要将错误代码从⼀个 Int 转换成⼀个 SKErrorCode
* 最后我们才可以检查它是否是我们想要的情况

看看上面这一坨代码，这只是为了匹配一个错误而已，真的有点复杂了！

毕竟这在 Swift 里可以通过一行模式匹配就搞定！所以我们有办法解决这个问题么？

答案很简单，将 `NS_ENUM` 替换成 `NS_ERROR_ENUM`, ⽤错误的作用域替（SKErrorDomain）换原始类型（NSInteger）

此时，我们再看 Swift 接口文件的话，我们会发现它已经不再是静态常量，⽽是结构体了！

![IMAGE](54.jpg)

它里面的具体内容会如下所示，是不是变得很 Swift 了？

![IMAGE](55.jpg)

除了上⾯的优化，Swift 编译器还做了如下的调整

* SKError 自动遵循了 Error 协议，使得其使用习惯更贴近 Swift 的⽤法。
* 提供一个 tilde equal 操作符，这就是 case 和 catch 语句匹配时使⽤的匹配操作符，

SKError 的这部分内容在⽣成界⾯是不可见的，但编译器真的会合成他们，并供你使⽤！⽽你要做的就是改⼀⾏代码！

![IMAGE](56.jpg)

至此，我们终于打造出来一个比较不错的 API 接口了！

## 总结

现在回到文章最开始的地方，思考⼀个问题，在⽣成的 Swift 接口中，并不是所有的 API 都有明显的缺陷，例如最后一个 Error Code 的案例，可能只有当我们看到 SKErrorCode 被使⽤的时候，才意识到这里有改进的空间。

虽然查看编译器生成的 Swift 头文件是一个好的方法。但⽣ 成的接口并不是全部，真正重要的是使用者在实际使⽤过程中写出的调⽤代码。

所以当我们在思考如何打造一个更适合 Swift 使用的接口是，不光要看看⽣成的接口。也应该考虑实际的使用场景。

![IMAGE](57.jpg)

总之，在 Session 中，Apple 的工程师提供给了我们很多在 Swift 里改善 Objective-C 框架体验的方法，也着重提醒了我们，相比于关注头文件，我们要更关注实际的使用体验和使用场景。

相信大家一定都有不少的收货，可能有人会问，这就是所有的手段了么？当然，Session 里面并没有展示全部的细节，如果你想进一步了解相关的内容，可以阅读以下内容：

* Swift Document 里的 “Language Interoperability” 章节，[传送门在此](https://developer.apple.com/documentation/swift#2984901)

如果你对 Swift API 本身的风格还不够了解，建议先阅读下面的文档：

* Swift API Design Guidelines: [中文传送门](https://github.com/SketchK/the-swift-api-design-guidelines-in-chinese)，[英文传送门](https://swift.org/documentation/api-design-guidelines/)

当然如果你想了解这种混编过程的细节，建议您观看 [WWDC 18 Behind the Scenes of Xcode Build Process](https://developer.apple.com/videos/play/wwdc2018/415/)
