---
title: UIAppearance 使用指北
comments: true
date: 2018-01-25 12:48:11
updated:
tags:
	- iOS
	- UIKit
categories:
	- iOS

---

<!-- more -->

## UIAppearance 使用指北

### UIAppearance 的简介
在 UIAppearance 出现之前，开发者如果想统一修改 app 内某一个控件的 UI 样式时，只能通过去修改每个控件的实例属性，对于只有几个实例的 UI 控件来说，这样的修改还可以接受，但如果整个 app 中有几十个，甚至上百个实例的时候，这样的修改就显得相当笨拙了，当然你也可以考虑使用一些黑魔法来实现，不过这或多或少都给开发者带来了不少麻烦。

除了上面提到的场景外，还有一种场景就是在 app 内提供多种多样的主题来满足用户的需求，例如手淘在 app 内提供的主题切换功能。

上面这两个实际开发中的应用场景都映射出这样一个问题：如何在整个 app 中**高效，统一，即时**的定制 UI 控件样式。

在 iOS 5.0 之后，Apple 为开发者提供了名为 UIAppearance 和 UIAppearanceContainer 的类，它们就是苹果为开发者提供的一种官方解决方案。

为了在现有的 UIKit 框架里面去做这个事，苹果的思路是: 让 UIAppearance 成为一个可以返回代理的协议，通过它可以把任何配置转发给特定类的实例。

但为什么不直接在 UIView 里面搞个属性或者方法来做这件事儿呢？

因为诸如 UIBarButtonItem 这样的控件并不是 UIView 的子类，它只是持有一个视图实例而已。通过这种设计，UIAppearance 可以处理所有类型的 UI 控件，无论它是 UIView 的子类，还是包含了视图实例的非 UIView 控件。

### 阅读 UIAppearance 和 UIAppearanceContainer 的头文件
我们刚才说过 UIApearance 实际上是一个协议，该协议需实现以下几个方法：

```objc
+ (instancetype)appearance;
+ (instancetype)appearanceWhenContainedInInstancesOfClasses:(NSArray<Class <UIAppearanceContainer>> *)containerTypes NS_AVAILABLE_IOS(9_0);
+ (instancetype)appearanceForTraitCollection:(UITraitCollection *)trait NS_AVAILABLE_IOS(8_0);
+ (instancetype)appearanceForTraitCollection:(UITraitCollection *)trait whenContainedInInstancesOfClasses:(NSArray<Class <UIAppearanceContainer>> *)containerTypes  NS_AVAILABLE_IOS(9_0);

// 已经废弃的两个方法
+ (instancetype)appearanceWhenContainedIn:(nullable Class <UIAppearanceContainer>)ContainerClass, ... NS_REQUIRES_NIL_TERMINATION NS_DEPRECATED_IOS(5_0, 9_0, "Use +appearanceWhenContainedInInstancesOfClasses: instead") __TVOS_PROHIBITED;
+ (instancetype)appearanceForTraitCollection:(UITraitCollection *)trait whenContainedIn:(nullable Class <UIAppearanceContainer>)ContainerClass, ... NS_REQUIRES_NIL_TERMINATION NS_DEPRECATED_IOS(8_0, 9_0, "Use +appearanceForTraitCollection:whenContainedInInstancesOfClasses: instead") __TVOS_PROHIBITED;

```

在这个协议中，需要简单解释一下第 3 和第 4 个方法，这个两个方法是用于解决 Size Classes 的问题而诞生的，通过这两个 API，我们可以控制在不同屏幕尺寸下的样式。

另外一个与之对应的协议是 UIAppearanceContainer，该协议并没有任何约定方法。因为它只是作为一个容器。

常见的，如 UIView 实现了 UIAppearance 这两种协议，既可以获取外观代理，也可以作为外观容器。而 UIViewController 则是仅实现了 UIAppearanceContainer 协议，很简单，它本身是控制器而不是 view，作为容器，为 UIView 等服务。

### UIAppearance 的使用

#### 哪些属性可以被 UIAppearance 调用？
在 UIKit 中被 `UI_APPEARANCE_SELECTOR` 宏标注的属性可以被 UIAppearance 调用。 例如 UIBarButtonItem 里的 tintColor 属性。

```objc
@property(nullable, nonatomic,strong) UIColor *tintColor NS_AVAILABLE_IOS(5_0) UI_APPEARANCE_SELECTOR;
```

> 在 swift 中没有宏的概念，所以属性无法被 `UI_APPEARANCE_SELECTOR` 标注，如果想让某个属性支持 UIAppearance 可以为该属性使用 `dynamic` 关键字

#### 如何使用 UIAppearance 修改 UI 控件的属性？
UIAppearance 不仅可以修改某一类型控件的全部实例，也可以修改部分实例，开发者只需要使用正确的 API 即可。还是以 UIBarButtonItem 为例。

当我们想改变所有 UIBarButtonItem 实例的 tintColor 时，代码如下:

```objc
[[UIBarButtonItem appearance] setTintColor:myColor];
```

当我们想在某些指定容器类里改变 UIBarButtonItem 的 tintColor 时，我们可以这么做:

```objc
[[UIBarButtonItem appearanceWhenContainedInInstancesOfClasses:@[[UINavigationBar class]]] setTintColor:myColor];
```

UIAppearance 里与 UITraitCollection 相关的两个方法与上面两个方法的使用规则相似，在这里就不做赘述了。

#### 如何让 UIAppearance 生效？

使用 UIAppearance 只有在视图添加到 window 时才会生效，对于已经在 window 中的视图并不会生效。所以，对于已经在 window 里的视图，可以采用从视图里移除并再次添加回去的方法使得 UIAppearance 的设置生效。

所以如果你发现自己的设置没有生效的话，不妨参考下这个规则。

#### 如何让自定义 view 的属性支持 UIAppearance

在实际使用过程中，我们绝大多数的视图类都继承自 UIView，UIView 的容器也基本上是 UIView 或 UIController，所以基本不需要开发者去实现这两个协议。

对于需要支持使用 UIAppearance 来设置的属性，在属性后增加 `UI_APPEARANCE_SELECTOR` 宏声明即可。

需要说明的是，在遵循 UIAppearanceContainer 协议的类中，声明与 UIAppearance 相关属性的方法时，要遵循两个代码规范：

1. 在方法的最后用 `UI_APPEARANCE_SELECTOR` 标注
2. 方法命名要遵守下面的格式


```objc
//You may have no axes or as many as you like for any property.
- (void)setProperty:(PropertyType)property forAxis1:(IntegerType)axis1 axis2:(IntegerType)axis2 axisN:(IntegerType)axisN;
- (PropertyType)propertyForAxis1:(IntegerType)axis1 axis2:(IntegerType)axis2 axisN:(IntegerType)axisN;

//Example
- (void)setBackgroundImage:(nullable UIImage *)backgroundImage forState:(UIControlState)state barMetrics:(UIBarMetrics)barMetrics NS_AVAILABLE_IOS(5_0) UI_APPEARANCE_SELECTOR;
- (nullable UIImage *)backgroundImageForState:(UIControlState)state barMetrics:(UIBarMetrics)barMetrics NS_AVAILABLE_IOS(5_0) UI_APPEARANCE_SELECTOR;
```

### UIAppearance 的探讨
Apple 官方对于 UIAppearance 的介绍并不多，网上的文章也大多都停留在使用层面上，今天我们来深入讨论一下 UIAppearance 内部的秘密。

#### 没有什么卵用的 `UI_APPEARANCE_SELECTOR` 

查看 Apple 的官方文档后，你就会发现它其实什么也没干.....

```objc
#define UI_APPEARANCE_SELECTOR

```

既然它什么都没干，那么对于没有被 `UI_APPEARANCE_SELECTOR` 标记的属性，我们是否也能用 UIAppearance 进行统一设置呢？

答案是可以的。

此时你内心充满了疑惑，

我们不妨将这个问题分成两个部分：

* 第一部分就是为什么会有这个宏定义，如果真的没有它会有什么问题么？
* 第二部分就是 UIAppearance 是如何实现统一修改控件样式的呢？ 

#### 真没问题么？

事实证明， UIAppearance 确实可以对一些没有被 `UI_APPEARANCE_SELECTOR` 标记的属性进行设置，甚至某些方法也是支持的 UIAppearance，例如 UISegmentedControl 的 setWidth:forSegmentAtIndex 方法。

但这种做法应该被避免，原因很简单：**这些没有被标记的属性不一定会在未来的 iOS 版本中适用。**

在 [Understanding UIAppearance](http://johnpetitto.com/understanding-uiappearance) 这篇文章里也给出了同样的观点。

#### 如何实现的？
那么 UIAppearance 到底是如何实现的呢？国外的大神们已经对此有了研究结论，我直接给出链接，感兴趣的朋友可以阅读一下 [UIAppearance and Custom Views](http://www.logicality.co/2012/10/08/UIAppearance-and-Custom-Views/)。

#### 友善的提醒！
在自定义 View 中使用 UIAppearance 还是有一些需要注意的事项，具体的内容可以参考 [UIAppearance for Custom Views](http://petersteinberger.com/blog/2013/uiappearance-for-custom-views/)

### 参考文献
* UIAppearance: [http://nshipster.com/uiappearance/](http://nshipster.com/uiappearance/)
* UIAppearance for Custom Views: [http://petersteinberger.com/blog/2013/uiappearance-for-custom-views/](http://petersteinberger.com/blog/2013/uiappearance-for-custom-views/)
* Understanding UIAppearance: [http://johnpetitto.com/understanding-uiappearance](http://johnpetitto.com/understanding-uiappearance)
* UIAppearance and Custom Views: [http://www.logicality.co/2012/10/08/UIAppearance-and-Custom-Views/](http://www.logicality.co/2012/10/08/UIAppearance-and-Custom-Views/)
* iOS UIAppearance 探秘: [https://hyancat.com/posts/2016/04/13/UIAppearance/](https://hyancat.com/posts/2016/04/13/UIAppearance/)

