---
title: 由 Tagged Pointer 联想到的一个问题
comments: true
date: 2018-07-29 11:20:15
updated:
tags:
categories:
---

在某些看似正确的写法中，Tagged Pointer 技术恰巧将本该发生的问题隐藏了起来。今天我们来剖析下这里面的小

<!-- more -->

## 前言

最近和基友 Maize 聊天，他给我普及了一个有意思的知识点，回看唐巧的 [深入理解Tagged Pointer](http://blog.devtang.com/2014/05/30/understand-tagged-pointer/) 的文章，再结合之前在公司看到的代码，突然有了一些灵感，我们先上一段代码。

```objc
@interface NSObject (AssociatedObject) 
@property (nonatomic, assign) CGFloat someProperty; 
@end 

@implementation NSObject (AssociatedObject) 
@dynamic someProperty; 

- (void)setSomeProperty:(CGFloat)someProperty{ 
	return objc_setAssociatedObject(self, @selector(someProperty), @(someProperty), OBJC_ASSOCIATION_ASSIGN); 
} 

- (CGFloat)someProperty{ 
	return [objc_getAssociatedObject(self, @selector(someProperty)) floatValue]; 
} 
@end 
```

如果此时我们给 someProperty 属性赋值并打印该属性的值，你认为它会 crash 么？
我们先说一下结论，如果这个属性的值为100，那么它不会 crash，如果是 100.1 那么就一定会 crash。
如果你马上就明白了其中的原理，那么恭喜你，你完全不用再花时间看此篇文章了。
如果你没有意识到其中的问题，不妨花上 10 分钟时间来仔细读读这篇文章的内容。

## Tagged Pointer 是什么？

> 本节内容是节选自唐巧的文章 - [深入理解Tagged Pointer](http://blog.devtang.com/2014/05/30/understand-tagged-pointer/)

在 2013 年的 WWDC 上，Apple 推出了首个 64 位架构的双核处理器，为了节省内存和提高执行效率，Tagged Pointer 概念诞生了。Apple 宣称引入该技术后，相关逻辑能减少一半的内存占用，以及 3 倍的访问速度提升，100 倍的创建、销毁速度提升。

为了能让大家更好的理解上面代码的问题，我们需要了解 Tagged Pointer 的一些实现细节。

我们知道 NSNumber、NSDate 一类的变量本身的值需要占用的内存大小常常不需要 8 个字节，拿整数来说，4 个字节所能表示的有符号整数就可以达到 20 多亿（注：2^31=2147483648，另外 1 位作为符号位)，对于绝大多数情况都是可以处理的。

所以 Apple 将一个对象的指针拆成两部分，一部分直接保存数据，另一部分作为特殊标记，表示这是一个特别的指针，不指向任何一个地址。所以，引入了 Tagged Pointer 对象之后，64 位 CPU 下 NSNumber 的内存图变成了以下这样：

![01](01.jpg)

对此，我们也可以用 Xcode 做实验来验证。我们的实验代码如下：

```objc
int main(int argc, char * argv[])
{
    @autoreleasepool {
        NSNumber *number1 = @1;
        NSNumber *number2 = @2;
        NSNumber *number3 = @3;
        NSNumber *numberFFFF = @(0xFFFF);
        
        NSLog(@"number1 pointer is %p", number1);
        NSLog(@"number2 pointer is %p", number2);
        NSLog(@"number3 pointer is %p", number3);
        NSLog(@"numberffff pointer is %p", numberFFFF);
        return UIApplicationMain(argc, argv, nil, NSStringFromClass([AppDelegate class]));
    }
}
```

运行之后，我们得到的结果如下，可以看到，除去最后的数字最末尾的 2 以及最开头的 0xb，其它数字刚好表示了相应 NSNumber 的值。

```objc
number1 pointer is 0xb000000000000012
number2 pointer is 0xb000000000000022
number3 pointer is 0xb000000000000032
numberFFFF pointer is 0xb0000000000ffff2
```

可见，苹果确实是将值直接存储到了指针本身里面。我们还可以猜测，数字最末尾的 2 以及最开头的 0xb 是否就是苹果对于 Tagged Pointer 的特殊标记呢？我们尝试放一个 8 字节的长的整数到 NSNumber 实例中，对于这样的实例，由于 Tagged Pointer 无法将其按上面的压缩方式来保存，那么应该就会以普通对象的方式来保存，我们的实验代码如下：

```objc
NSNumber *bigNumber = @(0xEFFFFFFFFFFFFFFF);
NSLog(@"bigNumber pointer is %p", bigNumber);
```

运行之后，结果如下，验证了我们的猜测，bigNumber 的地址更像是一个普通的指针地址，和它本身的值看不出任何关系：

```objc
bigNumber pointer is 0x10921ecc0
```

可见，当 8 字节可以承载用于表示的数值时，系统就会以 Tagged Pointer 的方式生成指针，如果 8 字节承载不了时，则又用以前的方式来生成普通的指针。关于 Tagged Pointer 的存储细节，[Mike Ash 的文章](https://mikeash.com/pyblog/friday-qa-2012-07-27-lets-build-tagged-pointers.html) 中也有一些讨论。

## Tagged Pointer 带来的问题？
我们结合一下刚才的代码，试想如果我们有以下两种使用场景，哪一种会有问题呢？

* 代码片段1
  
  ```objc
  // CodeSnippets 1
  self.view.someProperty = 100;
  NSLog(@"%@", @(self.view.someProperty));
  ```

* 代码片段2
  
  ```objc
  //CodeSnippets 2
  self.view.someProperty = 100.1;
  NSLog(@"%@", @(self.view.someProperty));
  ```

不管你是通过代码实验的，还是已经想明白了，第二个代码片段是会造成 crash 的，而且 Xcode 会提示这是一个野指针方面的问题。

那么结合刚才所说的 Tagged Pointer，我们怎么解释这个问题呢？

**当赋值为 100 时，由于 Tagged Pointer 的优化，`@(100)` 生成的 NSNumber 对象并不是一个严格意义上的对象，所以系统不会在堆上开辟内存存储对象，值是直接保存在地址指针里面，在取出的时候也就不会出现任何释放的问题。**

**而当赋值为 100.1 时，由于 Tagged Pointer 对这种值没法进行优化，`@(100.1)` 生成的 NSNumber 对象是一个真正意义上的对象，所以此时存储下来的是一个地址指针，但由于在关联的时候我们选用了 `OBJC_ASSOCIATION_ASSIGN`，那么此时系统并不会帮我们去进行那些计数引用等操作，所以当我们想再取出的时候，就会出问题了，也就是野指针的问题。**

至此，应该很好的解释了前言中那段代码的问题。

## 结论

那么我们应该如何优化这段代码呢？

* 假设你真的想存一个 CGFloat，由于在存取过程中操作的是一个对象，不妨将 `objc_AssociationPolicy` 选项中的 `OBJC_ASSOCIATION_ASSIGN` 换成 `OBJC_ASSOCIATION_RETAIN_NONATOMIC`。
* 如果可以的话，建议直接保存一个 NSNumber 类型的对象来替换 CGFloat 类型的数值，毕竟这样会更安全，记得同样把关联策略设定为 `OBJC_ASSOCIATION_RETAIN_NONATOMIC`

  > 关于 CGFloat 还需要注意的一点是：在不同的 CPU 下，它的真实类型是不一样的，有时是 float，有时是 double，那么在从 NSNumber 做转换的时候，到底要返回何种类型，仍然需要开发者做好判断。

* 在 Apple 的 API 文档中，`OBJC_ASSOCIATION_ASSIGN` 说是用于 `weak` 类型的引用，但并不是 ARC 范围内的 `weak`，严格来说应该更类似于 `unsafe_unretained` 的概念，如果真的要用 `OBJC_ASSOCIATION_ASSIGN` 策略，请时候考虑好对象的生命周期，避免不必要的 crash 。

  > 参考 NSHipster 的文章 [Associated Objects](https://nshipster.cn/associated-objects/)

* 如果你是声明一个 ARC 范围内的 `weak` 属性，你可能需要一个类似弱引用表概念的东西，不妨阅读下瓜神的这篇文章 [weak 弱引用的实现方式](https://github.com/Desgard/iOS-Source-Probe/blob/master/Objective-C/Runtime/weak%20%E5%BC%B1%E5%BC%95%E7%94%A8%E7%9A%84%E5%AE%9E%E7%8E%B0%E6%96%B9%E5%BC%8F.md)

