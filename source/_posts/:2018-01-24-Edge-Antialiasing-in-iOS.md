---
title: iOS 中的抗锯齿处理思路小结
comments: true
date: 2018-01-24 12:46:10
updated:
tags:
	- iOS
	- UIKit
categories:
	- iOS

---

<!-- more -->

当 UI 控件由于旋转或者图片自身问题而产生锯齿现象的时候，我们通常有三种解决方案：

1. 开启UI控件的抗锯齿功能
2. 绘制带有 1 像素透明边的图片
3. 使用更清晰的素材

以下面的例子为例，左侧为非抗锯齿视图，右侧为抗锯齿视图：

![01](01.jpg)

锯齿现象的细节

![02](02.jpg)

## 开启抗锯齿功能
当锯齿现象是由于 UI 控件自身引起的时候，例如旋转，形变等，我们就可以使用iOS 自带的抗锯齿功能来解决这一问题。

如果想开启全局抗锯齿的话，可以在 info.plist 中加入以下 key-value。

![03](03.jpg)


但由于该功能会造成性能上的损耗，苹果官方并不推荐使用这种全局开启的方式，我们只需要对特定的视图开启即可

```objc
layer.allowsEdgeAntialiasing = YES
```

## 绘制带有 1 像素透明边的图片
如果有问题的视图是显示一个静态资源或者动画过程中可以呈现为静态内容（例如在 imageView 里模拟 gif），那么我们就可以使用该方法。

将视图渲染为 UIImageView 并且将边界值为设置为 1，此时 UIImageView 就会自动处理它

我们看一下使用该方法处理后的效果图

![04](04.jpg)

代码实现起来较为简单，可以写成 UIImage 的 Category 方法，示例代码如下

```objc
- (UIImage *)antiAlias
{
    CGFloat border = 1.0f;
    CGRect rect = CGRectMake(border, border, self.size.width-2*border, self.size.height-2*border);
	
    UIImage *img = nil;
    
    UIGraphicsBeginImageContext(CGSizeMake(rect.size.width,rect.size.height));
    [self drawInRect:CGRectMake(-1, -1, self.size.width, self.size.height)];
    img = UIGraphicsGetImageFromCurrentImageContext();
    UIGraphicsEndImageContext();
    
    UIGraphicsBeginImageContext(self.size);
    [img drawInRect:rect];
    UIImage* antiImage = UIGraphicsGetImageFromCurrentImageContext();
    UIGraphicsEndImageContext();
    
    return antiImage;
}
```

## 使用更清晰的素材
这个思路比较简单，咱们就不细说了吧。

## 案例分析
近日，有部分同事反馈首页浮动弹框图片的锯齿现象比较严重，希望我们能改进，我们看一下原始效果:

![05](05.gif)

根据当前的使用场景，我们应该使用方案二进行处理，效果如下：

动态效果图
![06](06.gif)


静态效果图
![07](07.jpg)

乍一看，似乎没有任何改变，让我们把图片放大来看看细节

![08](08.png)

我们发现使用方案二确实生效了，而且在锯齿边缘产生了新的像素点，但方案受限于图片自身大小和分辨率的问题，导致效果不够明显。

## 参考资料：
Efficient Edge Antialiasing: https://markpospesel.wordpress.com/2012/03/30/efficient-edge-antialiasing/
StackOverFlow: https://stackoverflow.com/questions/1136110/any-quick-and-dirty-anti-aliasing-techniques-for-a-rotated-uiimageview/5057451#5057451

