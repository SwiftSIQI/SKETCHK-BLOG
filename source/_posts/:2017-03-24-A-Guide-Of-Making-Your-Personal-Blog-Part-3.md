---
title: A Guide Of Making Your Personal Blog - Part 3
comments: true
date: 2017-03-24 23:42:18
updated: 2019-08-29 20:51:43
tags:
	- Blog
categories:
	- DIY

---

<!-- more -->

> 2019-08-29 20:51:43
>  
> * 更新了 Github 服务器的 IP 地址，详情见 `解释下在 DNSPods 里的操作` 里的备注。

![01](01.png)

**A Guide Of Making Your Personal Blog 系列**

* [Part 1：概述](http://sketchk.xyz/2017/03/24/A-Guide-Of-Making-Your-Personal-Blog-Part-1/)
* [Part 2：域名与服务器](http://sketchk.xyz/2017/03/24/A-Guide-Of-Making-Your-Personal-Blog-Part-2/)
* [Part 3：域名解析](http://sketchk.xyz/2017/03/24/A-Guide-Of-Making-Your-Personal-Blog-Part-3/)
* [Part 4：博客框架](http://sketchk.xyz/2017/03/24/A-Guide-Of-Making-Your-Personal-Blog-Part-4/)
* [Part 5：博客主题](http://sketchk.xyz/2017/03/24/A-Guide-Of-Making-Your-Personal-Blog-Part-5/)
* [Part 6：自动部署](http://sketchk.xyz/2017/03/24/A-Guide-Of-Making-Your-Personal-Blog-Part-6/)
* [Part 7：总结与参考资料](http://sketchk.xyz/2017/03/24/A-Guide-Of-Making-Your-Personal-Blog-Part-7/)

## 域名解析

估计你已经十分讨厌我每次在开干前的废话，所以我决定这次咱们先开干！走起

### 在 DNSPods上进行操作

* 登录 `DNSPods` 官网： [https://www.dnspod.cn/](https://www.dnspod.cn/)
* 注册 `DNSPods` 账户，这一步咱就不废话了
* 注册成功后，用用户名登录，点击左边的“域名解析”, 然后点击“添加域名”, 点击域名添加以下两个记录. 另外 `name.github.io` 中的 `name` 是你 `GitHub` 的用户名。

| 主机记录	|记录类型|	线路类型| 记录值| MX优先级| TTL|
|---- |---- |---- |---- |---- |---- |
|  @	| CNAME |	默认 |	name.github.io.| 无输入值 | 10 |
| www | CNAME | 默认 |	name.github.io.| 无输入值 | 10 |

### 在 GoDaddy 上进行操作

* 登录 `GoDaddy` 的账户，点击自己账户里的 `My Products` 选项。
* 在弹出的界面里面选择 `Domains` 一行的 `Manage` 按钮

![02](02.jpg)

* 点击左边卡片中右上角的齿轮按钮，然后再点击 `Manage DNS` 按钮

![03](03.jpg)

* 在 `Nameservers` 卡片中修改 `NameServe` 为 `f1g1ns1.dnspod.net`, `f1g1ns2.dnspod.net`

![04](04.jpg)

### 在 GitHub 上进行操作

* 打开我们在 `GitHub` 上的仓库，点击 `Settings` 按钮

![05](05.jpg)

* 在 `GitHub Pages` 的选项卡中的 `Custom domain` 中填写你自己购买的域名

![06](06.jpg)

## 测试一下

好了，现在所有的操作都做完了，在浏览器里输入一下你自己购买的域名吧，那个熟悉的页面又出现了，只是这次的网址变成了你之前购买的域名。

![07](07.jpg)

## 我们都干了些什么？

不过，你明白刚才做的这些事情都代表什么意思么？如果出了问题应该去寻找那方面的资料呢？是不是感觉有点不知所措。所以我们还是静下心来看看我们这些操作到底都代表着什么意思。

### 解释下在 DNSPods 里的操作

首先，我们注册了一个 `DNSPods` 账号，并做了一些操作。

在解释这些操作之前，我们得先说明一下 `DNS` 和 `DNSPods`。

`DNS` 的全称是 `Domain Name System`，所以通过字面就可以理解出它其实是一个域名解析系统，用于解析域名对应的服务器。

`DNSPods` 是一个提供 `DNS` 服务的公司。当然还有其他提供 `DNS` 服务的公司，如果你想用别的也完全 OK。

我们在 `DNSPods` 里面进行域名解析的时候到底又做了哪些工作呢？让我们重新看下刚才填写的表格：

![08](08.jpg)

这里面的名词包括，主机记录，记录类型，线路类型，记录值，权重，MX 优先级，TTL ，它们分别代表着什么意思呢，我们来一个个解释。

* 主机记录用于设置域名的前缀，用于告诉 `DNS` 服务器把 “域名前缀 + 域名” 的链接与记录值中的地址关联到一起，例如我们在主机记录里输入 `www` ，记录值里输入 `baidu.com` ，就代表我们把 `www.sketchk.xyz` 与 `www.baidu.com` 关联到一起。当然不同的前缀有不同的意思，常见的有：
	* `www`：表示解析域名 `www.skethck.xyz` 
	* `@`：表示解析主域名 `sketchk.xyz`


* 记录值是用来填完我们购买的域名地址或者服务器的 `IP` 地址。
	* 如果我们在记录类型里选择了 `A` 记录就在记录值里填写服务器的 `IP`。
	* 如果我们在记录类型里选择了 `CNAME` 记录就在记录值里填写我们购买的域名。


* 记录类型是用于区分记录值的类型，不同的记录类型要填写不同样式的记录值

* 线路类型是说我们让哪些线路的用户可以访问这个域名。举个例子，如果我们指定了电信用户，那么联通用户就无法访问我们这个域名，通常我们选默认就好。

* TTL 是 Time To Live 的缩写，它表示一条域名解析记录在 `DNS` 服务器中的存留时间，单位是秒，所以 600 是表示 10 分钟的意思。

* 权重和 MX 优先级 跟我们做博客这件事没有太大关系，就不废话了

在这里估计有人会问，我们使用的是 `GitHub Pages` 服务，并没有实际去购买一个服务器，那么我们怎么去获得这个 `IP` 地址呢？ 其实在 `GitHub` 的官网上就有解释，具体详情请点击[这里](https://help.github.com/articles/setting-up-an-apex-domain/)。

如果你仔细读了 `Customizing GitHub Pages` 的这篇[文档](https://help.github.com/articles/about-supported-custom-domains/)，就会发现 `GitHub` 推荐在 `DNS` 里面使用 `A` 记录和 `CNAME` 结合的方式解析，而我在之前使用的是 `CNAME` 记录的方式，那么我们就改一下吧。

![09](09.jpg)

> 备注：于 2019 年 8 月发现 Github 将服务器 IP 地址更新了，相关说明的链接： [DNS Configuration errors](https://help.github.com/en/articles/troubleshooting-custom-domains#dns-configuration-errors) ，所以上图里的记录值需要改为：
>  
> * 185.199.108.153  
> * 185.199.109.153  
> * 185.199.110.153  
> * 185.199.111.153  

### 解释下在 GoDaddy 里的操作

现在把我们在 `DNSPods` 里面干的事情算是搞明白了，我们是告诉了 `DNS` 服务器应该如何解析我们购买的域名，那么下一步就是告诉我们的域名在解析的时候去找刚才配置过的 `DNS` 服务器。

好了，让我们回顾下刚才填写的内容。

![10](10.jpg)

估计有些好奇的小伙伴一定会好奇，为什么我们要填写 `f1g1ns1.dnspod.net`, `f1g1ns2.dnspod.net`，而不是其他的值，原因很简单，`DNSPods` 在它的使用手册里面说明了它们的地址就是这样的。如果有任何疑问可以看它们提供的[帮助文档](https://support.dnspod.cn/Kb/showarticle/tsid/42/)。

所以如果你没有使用 `DNSPods` 作为你的 `DNS` 服务商，只需要找到对应的地址并填写到 `Nameservers` 里面即可。

### 解释下在 GitHub 里的操作

回顾下我们前面做的事，我们在 `DNSPods` 里面给出了域名和服务器的映射关系，然后我们告诉了域名商在解析时候去 `DNSPods` 里进行解析，这样它们就可以找到对应的服务器 `IP` 地址了，用一个流程图来表示的话，就是下面的样子：

![11](11.jpg)

> 如果你对上面这幅图有疑问，可以在终端里面输入 `dig sketchk.xyz` ，`dig www.sketchk.xyz`，`dig github.sketchk.io` 来查看具体信息

这一切看起来都能解释清楚，但我们想想 `Github` 的服务器上不光有 `sketchk.github.io` 的仓库，也可能有 `abc.github.io` 的仓库, 还会有 Pop , AsyncDisplayKit 等一堆第三方仓库，那当我们通过自定义域名（例如 `sketchk.xyz`）找到这台服务器的时候，服务器是怎么把正确的资源返回给我们的呢？

如果你不太明白我在说什么的话，请回顾 `在 GitHub 上进行操作` 一节的内容并删除 `Custom Domain` 中的内容，然后你再访问下自己购买的域名？ 是不是 404 了！ 但如果你访问 `GitHub` 自动分配的那个域名（例如给我分配的 `sketchk.github.io`），你会发现一切正常。是不是很有意思？

所以我们需要确定一个 `GitHub` 和 自定义域名的对应关系，这样当你访问 `GitHub` 的服务器的时候，`GitHub` 就知道应该返回给你什么样子的资源了。

心细的朋友可能会问，这个对应关系到底是怎么进行的呢？ 我自己做了一个实验，将 `Custom Domain` 改为了 `sketchktestforfun.xyz`, 此时无论你访问 `sketchktestforfun.xyz` 还是 `sketchk.xyz` 都会发现网站无法正常展示。

![12](12.jpg)

![13](13.jpg)

不过我们可以尝试用下面的方式来访问域名，比如把请求的 `Host` 改为 `sketchktestforfun.xyz` 

```bash
$ curl -v sketchk.xyz -H 'Host: sketchktestforfun.xyz'
```

![14](14.jpg)

通过上面这个图，我们发现使用 `sketchk.xyz` 这个域名又可以访问到 `sketchk.github.io` 的资源了，这说明如果一个域名最终指向 `GitHub` 的服务器，当我们修改其 Host 为自己的仓库名时，它们最终都会访问到我们自己的那个仓库，这也又一次证明了在 `Custom Domain` 中填写的内容是为了让服务器知道如何响应不同的网络请求。

估计这时候一些朋友会惊慌失措的说道：“如果我在 `Custom Domain` 里填写的域名跟人重复了的话，该怎么办？” 

对于这个问题，我们必须先搞清楚一件事，就是你在购买域名的时候，域名商一定不会卖给你一个已经被人买走的域名，Don't panic！

这里还有一些朋友会说我在一些文章里面看到的操作不是这样的，他们都是在对应的仓库里面添加了一个 `CNAME` 文件并在里面写上自己购买的域名。

其实吧，这两种做法是一样的，只不过 `GitHub` 以前没有提供这样一个简便的操作，所以之前的人会用创建 `CNAME` 的方式完成这个对应关系，但现在 `GitHub` 提供了这个功能，咱就怎么简单怎么来吧！

好了，终于把这一章的内容写完了，让我们休息一下，开始下一章的内容吧！