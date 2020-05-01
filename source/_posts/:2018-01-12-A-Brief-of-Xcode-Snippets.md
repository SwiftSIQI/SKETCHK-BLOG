---
title: Xcode 代码块工具简介
comments: true
date: 2018-01-12 12:52:01
updated:
tags:
	- iOS
	- Xcode
categories:
	- iOS

---

<!-- more -->

## Xcode 代码块功能简介
### 工具位置
* 编辑器右侧的工具面板, 点击 { } 图标

![01](01.png)

### 使用方式
* 拖拽 代码块
* 输入 代码块 快捷键

## 内置的代码块类型
* C 的 `enum`，`struct union`，和 `blocks` 的 typedef 声明
* C 控制流语句像 `if`，`if...else`，和 `switch`
* C 循环，像 `for`，`while`，和 `do...while`
* C 内联 block 变量声明
* Objective-C 的 `@interface` 声明（包括类扩展和分类），`@implementation`，`@protocol`
* Objective-C 的 KVO 样板，包括相对模糊的 keyPathsForValuesAffecting<Key>，用来 注册相关的键
* Objective-C 的 Core Data 访问，属性存取，属性验证样板。
* Objective-C 的枚举 NSIndexSet 惯用语
* Objective-C 的 `init`，`initWithCoder:` 和 `initWithFrame:` 实现方法
* Objective-C `@try`/`@catch`/`@finally`和`@autorelease` blocks
* GCD `dispatch_once`和`dispatch_after`惯用语
* C++ 相关
* Swift 相关
* 几个特殊方法
    * test method
    * debugDescription
    * description
>Tips:
>* Enumerated Type Declaration (`NS_ENUM`) 的缩写是 `NSENUM`
>* Enumerated Type Declaration (`NS_OPTIONS`) 的缩写是 `NS_OPTION`
>* `~/Library/Developer/Xcode/UserData/CodeSnippets/`目录存放了所有 Xcode 代码段的文件表示

## 自定义代码块
* 拖拽代码块至 {} 面板即可， 可以参考下图
![02](02.png)

* 用户将自定义的代码块添加到库里面后，可以双击列表中的块去编辑。
![03](03.jpg)

* 每个块都有以下内容：
    * `Title` 标题 - 代码块的名字（出现在代码补全和代码块库列表中）
    * `Summary` 简介 - 简单描述下它是干嘛的（只出现在代码块库列表中）
    * `Platform` 平台 - 限制可访问该代码块的平台。macOS，iOS，tvOS, watchOS 或者（“全部”）
    * `Language` 语言 - 限制可访问该代码块的语言。常见的有 C，Objective-C，C++，或 Objective-C++
    * `Completion Shortcut` 输入码 - 快捷输入码。常用块的输入码应该非常简练。Xcode 不会警告冲突 / 重复的输入码，所以一定要确保新添加的不要和已有的冲突。
    * `Completion Scopes` 有效范围 - 限制可访问该代码块的范围。if / else 语句的自动补全应该只在方法或者函数的实现中有效。下面这些选项可以任意组合：
        * All 全部
        * Class Implementation 类实现
        * Class Interface Methods 类接口方法
        * Class Interface Variables 类接口变量
        * Code Expression 代码表达式
        * Function or Method 函数或方法
        * Preprocessor Directive 预处理指令
        * String or Comment 字符串或注释
        * Top Level 最高层
    * 占位符
      * 在 Xcode 中，占位符使用`<#`和`#>`来分隔，中间是占位文本。
        ![04](04.jpg)

## Reference
* Xcode Snippets: http://nshipster.cn/xcode-snippets/
* 使用 Git 来管理 Xcode 中的代码片段: http://blog.devtang.com/2012/02/04/use-git-to-manage-code-snippets/


