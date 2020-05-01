---
title: UIBarButtonItem 在 iOS 11 上的改变及应对方案
comments: true
date: 2018-02-23 12:44:58
updated:
tags:
	- iOS
	- UIKit
categories:
	- iOS

---

<!-- more -->

## 总览

在 iOS 11 之后，Apple 在导航栏中启用了自动布局的相关特性，这使得导航栏的使用方式发生了一些变化，今天我们着重说说导航栏中 UIBarButtonItem 在 iOS 11 中的几点变化。

## 主要变化

* 视图层级的变化
* 点击区域的变化
* 与屏幕间距的变化

## 视图层级变化

### 表现形式
在 WWDC 2018 的 [Updating Your App for iOS 11](https://developer.apple.com/videos/play/wwdc2017/204/)中，我们可以知道 UINavigationBar 开始支持 Auto Layout 了。

这对于 UIBarButtonItem 来讲，意味着什么呢？通过 view debug 工具我们可以发现，所有的 item 会被一个内置的 stack view 所管理

![01](01.png)

当 Custom View 正确的实现了 sizeThatFits 或者 intrinsicContentSize 时，UI 的展现将不会出现问题。

### 注意事项
在 iOS 11 中，为了充分发挥 Auto Layout 特性，不免需要将 UIBarButtonItem 里 Custom View 的 translatesAutoresizingMaskIntoConstraints 属性设置为 no，这就可能会造成它在 iOS 11 以下的系统中发生布局错乱，因此我们需要在相应的地方写上如下代码

```objc
UIView *view = [UIView new];
if(@available(iOS 11, *)){
  view.translatesAutoresizingMaskIntoConstraints = NO;
}
UIBarButtonItem *item = [[UIBarButtonItem alloc] initWithCustomView:view];
self.navigationItem.rightBarButtonItem = item;
```

## 点击区域的变化

### 表现形式
我们以 Custom View 的方式创建两个 UIBarButtonItem，通过 view debug 工具可以查看 Custom View 的真实大小

![02](02.png)

在 iOS 10 之前的版本中，它的点击区域如红色区域所示：

![03](03.png)

在 iOS 11 中，它的点击区域发生了改变，改变的规则就是点击区域与 Custom View 自身大小保持一致

![04](04.png)

### 解决方案
对于这个问题，可以有两种解决方案:

一种方式是通过重写 `- (BOOL)pointInside:(CGPoint)point withEvent:(UIEvent *)event` 方法来修改控件的点击区域，保证其范围控制在 44 * 44 pt 以上。例如下面的示例代码

```objc
- (BOOL)pointInside:(CGPoint)point withEvent:(UIEvent *)event{
    CGSize acturalSize = self.frame.size;
    CGSize minimumSize = kBarButtonMinimumTapAreaSize; // 44 * 44 pt
    CGFloat verticalMargin = acturalSize.height - minimumSize.height >= 0 ? 0 : ((minimumSize.height - acturalSize.height ) / 2);
    CGFloat horizontalMargin = acturalSize.width - minimumSize.width >= 0 ? 0 : ((minimumSize.width - acturalSize.width ) / 2);
    CGRect newArea = CGRectMake(self.bounds.origin.x - horizontalMargin, self.bounds.origin.y - verticalMargin, self.bounds.size.width + 2 * horizontalMargin, self.bounds.size.height + 2 * verticalMargin);
    return CGRectContainsPoint(newArea, point);
}
```

另一种方式是创建一个中间层视图，保证其视图的大小不小于 44 * 44 即可，例如下面的代码

```objc
@interface WrapperView : UIView
@property (nonatomic, assign) CGSize minimumSize;
@property (nonatomic, strong) UIView *underlyingView;
@end

@implementation WrapperView
- (instancetype)initWithUnderlyingView:(UIView *)underlyingView {
    self = [super initWithFrame:underlyingView.bounds];
    if (self) {
        _underlyingView = underlyingView;
        _minimumSize = CGSizeMake(44.0f, 44.0f);
        underlyingView.translatesAutoresizingMaskIntoConstraints = NO;
        [self addSubview:underlyingView];
        NSLayoutConstraint *leadingConstraint = [underlyingView.leadingAnchor constraintEqualToAnchor:self.leadingAnchor];
        NSLayoutConstraint *trailingConstraint = [underlyingView.trailingAnchor constraintEqualToAnchor:self.trailingAnchor];
        NSLayoutConstraint *topConstraint = [underlyingView.topAnchor constraintEqualToAnchor:self.topAnchor];
        NSLayoutConstraint *bottomConstraint = [underlyingView.bottomAnchor constraintEqualToAnchor:self.bottomAnchor];
        NSLayoutConstraint *heightConstraint = [self.heightAnchor constraintGreaterThanOrEqualToConstant:self.minimumSize.height];
        NSLayoutConstraint *widthConstraint = [self.widthAnchor constraintGreaterThanOrEqualToConstant:self.minimumSize.width];
        [NSLayoutConstraint activateConstraints:@[leadingConstraint, trailingConstraint, topConstraint, bottomConstraint, heightConstraint, widthConstraint]];
    }
    return self;
}
@end
```

![05](05.png)

> 当然最简单的方法就是让你的图片变大一些，比如增大留白......

## 与屏幕间距的变化
在 iOS 11 之前，我们有两种方式来修改 item 与屏幕的间距

1. 设置一个宽带为负值，类型为 Fixed Space 的 item
2. 重写 Custom View 的 alignmentRectInsets 方法
3. 如果 Custom View 为 UIButton 类型，可以重写其 contentEdgeInsets/imageEdgeInsets/titleEdgeInsets 并修改其 hitTest 区域

### iOS 10 中的解决方案一：使用宽度为负值，类型为 Fixed Space 的 item 

当我们使用 Custom View 类型的 item 时，item 与屏幕的边距默认为 16 或者 20 pt（PS：当屏幕为5.5寸屏时，边距为 20 pt），下图以 16 pt 为例

![06](06.png)

如果我们希望 item 与屏幕边距为 8pt 时，我们的解决方案通常是这样的

```objc
UIBarButtonItem *spacer = [[UIBarButtonItem alloc] initWithBarButtonSystemItem:UIBarButtonSystemItemFixedSpace target:nil action:nil];
spacer.width = -8;
self.navigationItem.rightBarButtonItems = @[spacer, item1, item2];
```

这个方案在 iOS 10 及以下的系统是有效的，效果如下图所示

![07](07.png)

但是同样的代码在 iOS 11 上就是失效的......

![08](08.png)

仔细研究 iOS 11 中的视图布局可以发现，fixed space 类型的 item 在 Stack View 的 leading 和 trailling 时的行为与其在 item 之间的行为保持一致，它的最小宽度都为 8pt，这也就说明了设置负数为什么会不生效了。

![09](09.png)

### iOS 10 中的解决方案二：重写 Custom View 的 alignmentRectInsets 方法

我们可以重写某个视图控件的 alignmentRectInsets 属性，例如下面的代码

```objc
- (UIEdgeInsets)alignmentRectInsets {
    return self.position == right ? UIEdgeInsetsMake(0, -8, 0, 8) : UIEdgeInsetsMake(0, 8, 0, -8);
}
```

当我们设置 UIEdgeInsetsMake(0, -8, 0, 8) ，并将这段代码用到前面的例子时，你会看到这样的结果

![10](10.png)

乍一看，感觉这个方法似乎解决了所有的问题，但仔细观察，你就会发现这个方案也存在不完美的地方，例如超出 stack view 的部分将无法响应点击事件

![11](11.png)


### iOS 10 中的解决方案三：利用 XXXEdgeInsets 属性来修改

方案三和方案二比较相似，通过修改 XXXEdgeInsets 属性确实能让控件在视觉上达到预期，但由于 stack View 的存在，即使修改了 Custom View 的 hitTest 区域，也会存在无法响应点击事件的问题，所以这个方案也不是一个完美的解决方案

![12](12.png)

### 解决方案

虽然刚才提到的三种解决方案在 iOS 11 中都无法完美解决问题，但我们还是发现了一个有意思的现象

下图是使用宽度为负值，类型为 Fixed Space 的 item 时的视图布局，虽然 fixed space 的存在 让它在视觉上看起来离屏幕边距还是有点远，但 item 控件自身与屏幕的边距确实变小了。

![13](13.png)

这样说可能让人有点摸不着头脑，我们不妨来看看这中间的区别，下图是使用了非 customView 创建的 UIBarButtonItem，与屏幕的间距为 8 pt（当屏幕为 5.5 寸时，为 12 pt）

![14](14.png)

下图是使用了 customView 创建的 UIBarButtonItem，与屏幕的间距为 16 pt（当屏幕为 5.5 寸时，为 20 pt）

![15](15.png)

正是基于以上的观察，我们发现只要在 stack view 中添加一个 非 customView 创建的 UIBarButtonItem， 系统就会给我们减少 item 与屏幕间的距离，如果此时我们再用 alignmentRectInsets 提供一个视觉上的偏移，就可以完美解决当前的问题。

```objc
UIButton *customButton = [UIButton buttonWithType:UIButtonTypeCustom];
customButton.overrideAlignmentRectInsets = UIEdgeInsetsMake(0, x, 0, -x); // you should do this in your own custom class
customButton.translatesAutoresizingMaskIntoConstraints = NO;
UIBarButtonItem *item = [[UIBarButtonItem alloc] initWithCustomView:customButton]

UIBarButtonItem *positiveSeparator = [[UIBarButtonItem alloc] initWithBarButtonSystemItem:UIBarButtonSystemItemFixedSpace target:nil action:nil];
positiveSeparator.width = 8;

self.navigationItem.leftBarButtonItems = @{positiveSeparator, item, ...}
```

![16](16.png)

不过需要注意的是，这个方法能只能保证 item 距离屏幕的边缘为 8 - 12 pt，如果你想让 item 与屏幕的距离更近一些的话，就可能会出现其他的问题。

## 结论
UIBarButtonItem 在 iOS 11 上的变化让不少开发者都头疼不已，我们在[Apple Developer Forums](https://forums.developer.apple.com/message/247762#259317)上也可以看到不少开发者的解决方案，大体都是去对 NavigationBar 做修改，比如修改约束，自己实现一个导航栏基类等，但对于 我们现有的工程来说，这些方案的实现代价都太大了。

这篇文章提出的解决方案不需要对导航栏本身做任何修改，只需要在 UIBarButtonItem 上做一些处理即可，对比社区里现有的方案，该适配方案对历史包袱比较沉重的项目来说是值得尝试的。

