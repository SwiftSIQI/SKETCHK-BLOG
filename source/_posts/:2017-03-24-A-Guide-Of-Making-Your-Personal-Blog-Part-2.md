---
title: A Guide Of Making Your Personal Blog - Part 2
comments: true
date: 2017-03-24 23:41:59
updated: 2017-04-03 22:24:29
tags:
	- Blog
categories:
	- DIY

---

<!-- more -->

![01](01.png)

**A Guide Of Making Your Personal Blog 系列**

* [Part 1：概述](http://sketchk.xyz/2017/03/24/A-Guide-Of-Making-Your-Personal-Blog-Part-1/)
* [Part 2：域名与服务器](http://sketchk.xyz/2017/03/24/A-Guide-Of-Making-Your-Personal-Blog-Part-2/)
* [Part 3：域名解析](http://sketchk.xyz/2017/03/24/A-Guide-Of-Making-Your-Personal-Blog-Part-3/)
* [Part 4：博客框架](http://sketchk.xyz/2017/03/24/A-Guide-Of-Making-Your-Personal-Blog-Part-4/)
* [Part 5：博客主题](http://sketchk.xyz/2017/03/24/A-Guide-Of-Making-Your-Personal-Blog-Part-5/)
* [Part 6：自动部署](http://sketchk.xyz/2017/03/24/A-Guide-Of-Making-Your-Personal-Blog-Part-6/)
* [Part 7：总结与参考资料](http://sketchk.xyz/2017/03/24/A-Guide-Of-Making-Your-Personal-Blog-Part-7/)

## 域名
搭建个人博客这件事，首先一定要弄个域名吧，要不然怎么彰显这是你的个人博客呢？

### 在 GoDaddy 上购买域名
购买域名的话，其实仁者见仁，智者见智，我个人是在 [`GoDaddy`](https://sg.godaddy.com/zh/) 上买的, 当然，你要想在 [`阿里云`](https://wanwang.aliyun.com/) 上买也 OK，毕竟条条大路通罗马，不过在这里我就以 `GoDaddy` 为例好了。

1. 注册并登录 `GoDaddy` 这种事我就不细说了！聪明的你一定可以完成的！

2. 在搜索栏里寻找你感兴趣的域名，其实你随便输入一个字母就好，比如 `sketchk`

	![02](02.jpg)


3. 验证域名是否可用, 如果当前域名不可用会有提示，如果可用就会自动加到购物车

	![03](03.jpg)

4. 选好以后点右上角的 `Continue to Cart` 按钮

5. 稍等一会，你会进入到下面的页面，
	
	![04](04.jpg)
	
	* 第一个选项卡是在说，注册域名时，您的姓名、地址、电子邮件地址和电话号码都会自动发布到网上，大家都可以看见。借助 `GoDaddy` 隐私保护来保护自身免遭垃圾邮件和网络欺诈的侵害，它会将您的个人信息替换为 `GoDaddy` 的信息。
	* 第二个选项卡是在说你是否愿意使用 `GoDaddy` 提供的搭建网站服务，也就是 `GoCentral` 服务。
	* 第三个选项卡是说是否需要提供一个与你注册域名一致的邮箱地址。
	* 这些都是附加服务，如果你感觉有必要就买好了，当然如果你觉得都没必要，也完全可以都不选。
	

6. 接下来就是填写一些个人信息了，这一步按照自己的情况填写就好。

7. 最后就是付款了，记得在付款的时候去 [`独特优惠码`](http://www.dute.me/) 上找点折扣，毕竟能省就省点呗。

8. 付款结束后，你的注册邮箱会收到一封确认邮件，恭喜你，你现在拥有一个域名了！

好了，做完这些就代表你已经处理完与域名相关的大部分事情了，剩下的一些事，我们会在 `域名解析` 一节里完成，下面我们来说说服务器！

## 服务器

在前一篇文章中，我们已经说了服务器像一个存放资源的地方，而存放资源的时候，你可以用一个小型仓库，也可以用一个中型仓库，甚至可以用一个大型仓库。那么我们应该选一个什么样的服务器呢？

### 关于服务器的几件事
用于搭建网站的服务器大概有以下几种：虚拟主机，虚拟专享服务器以及专属服务器，它们之间的区别可以参考下图的说明：

![05](05.jpg)

像我们看到的大多数网站，其实都是“放”在服务器上的，然而，这个“放”其实并不像我们想的那么简单。

如果你不太理解的话，让我举个可能不是那么恰当的例子吧！

假如现在你已经拥有了一个域名并用 `HTML`，`CSS`，`JavaScript` 写好了一个堪称完美的网站，现在你把这些文件放在了某个服务器的磁盘中，那么你在另外一个电脑的浏览器里输入自己的域名会看到对应的网站么？

答案肯定是看不到的！

因为我们必须告诉服务器去监控各种各样的网络请求，让它能够对这些浏览器发送的网络请求进行分析，以便服务器能够知道这个请求到底想干嘛，然后再把这个请求想要的信息（例如 `HTML`，`CSS`，`JavaScript`）返回给对应的浏览器，这样你就能在浏览器上就能看到正确的内容了。

![06](06.png)

> 小白：额，什么是网络请求啊？ 
> SketchK：当你在浏览器上输入一个域名并敲击了回车就完成了一次网络请求！

所以说了这么多，你是不是突然意识到买服务器只是“服务器”这个 topic 里的一小部分。不过不要担心，因为我们还有更简单的办法，比如 `GitHub Pages`! 

![07](07.jpg)

`Github Pages` 是 `GitHub` 公司提供的免费静态网站托管服务，所以从严格的角度上说，`GitHub Pages` 并不是一个服务器，它只是提供了一种类似服务器的服务！那么这个服务到底是什么呢？

简单来说就是：如果我们将写好的 `HTML` 等文件放到 `GitHub` 的指定位置时，`GitHub Pages` 就能对这些文件进行处理并把它们展示成一个网站，同时还会自动给这个网站提供一个特定的域名。假如我们对 `GitHub Pages` 提供的域名不喜欢，我们完全可以把域名改成我们自己想要的！

所以仔细理解下这个服务的内容，是不是很符合我们的需求! 

* 我们只需要将资源放到 `GitHub Pages` 要求的地方就等于拥有了一台服务器。
* 假如我们对 `GitHub Pages` 提供的域名不喜欢，我们还能把它换成在 `GoDaddy` 上买到的域名。

好了，现在我们已经有解决方案了，那么我们将按照下面三个步骤来进行操作:

1. 安装 `Git`
2. 注册 `GitHub` 账户
3. 开启 `GitHub Pages` 服务

稍微等一下！`Git`, `GitHub`, `GitHub Pages` 这些都是神马东西！我已经完全不知所云了！Take it easy! 让我来简单的说明下吧：

* `Git` 是一种软件，通过它可以控制代码的版本  
* `GitHub` 是一个代码的托管平台，支持 `Git` 作为其代码的版本控制软件  
* `GitHub Pages` 是...(我刚才不是说过了么，忘了就自己看看上面的解释吧)  

![08](08.png)

OK，废话就此结束了，准备实战！

### 安装 Git

* 登录 `Git` 的官网：[https://git-scm.com/](https://git-scm.com/)
* 点击右边的 `DownLoads For Mac` 按钮

> 当然也可以用 [`brew`](https://brew.sh/) 命令来安装 `Git`, 这里为了减少学习成本就不说  ` homebrew` 了，好奇的同学可以自己尝试

![09](09.jpg)

后面的事，咱就不用废话了。记住，安装完以后，在 terminal 上输入 `git --version` 来查看 Git 版本，只有安装成功才会显示出你的 Git 版本，所以你也可以通过这个方式来确认自己是否安装成功！

![10](10.jpg)

### 注册 GitHub

> 如果你已经注册过一个 `GitHub` 账户了，那就跳过这一节吧！

* 登录 `GitHub` 的官网：[https://github.com/](https://github.com/)

![11](11.jpg)

* 点击 `Sing Up` 进行注册，注册的过程就不细说了，你肯定能搞定的！
* 在注册邮箱里面查收激活邮件并激活 `GitHub` 账户。记住，一定要激活！否则后面无法使用 `GitHub Pages` 服务。 
* 设置 `Git` 的用户信息
	* 在命令行里输入：
	
	```
	$ git config --global user.name "Your GitHub Name"
	$ git config --global user.email "Your-GitHub-Email-Address@example.com"
	```

* 配置 SSH Key
	* 创建 `SSH Key`。在用户主目录下，看看有没有 `.ssh` 目录，如果有，再看看这个目录下有没有 `id_rsa` 和 `id_rsa.pub` 这两个文件，如果已经有了，可直接跳到下一步。如果没有，打开终端，创建SSH Key：

	```bash
	$ ssh-keygen -t rsa -C "Your-GitHub-Email-Address@example.com"
	```

	* 你需要把邮件地址换成你自己的邮件地址，然后一路回车，使用默认值即可，由于这个Key也不是用于军事目的，所以也无需设置密码。

	* 如果一切顺利的话，可以在用户主目录里找到 `.ssh` 目录，里面有 `id_rsa` 和 `id_rsa.pub` 两个文件，这两个就是 `SSH Key` 的秘钥对，`id_rsa` 是私钥，不能泄露出去，`id_rsa.pub` 是公钥，可以放心地告诉任何人。

	* 登陆 `GitHub`，点击自己的头像，选择 `Settings`，再点击左边的 `SSH and GPG keys` 页面：

	* 然后，点 `Add SSH Key`，填上任意 `Title`，在 `Key` 文本框里粘贴 `id_rsa.pub` 文件的内容：

	![12](12.jpg)

* 最后在终端里验证下是否配置成功

	```
	ssh -T Your-GitHub-Email-Address@example.com
	
	```

* 如果在过程中遇到下面的提示, 不要紧张，输入 `yes` 就好。

	```
	The authenticity of host 'github.com (207.97.227.239)' can't be established.
	RSA key fingerprint is 16:27:ac:a5:76:28:2d:36:63:1b:56:4d:eb:df:a6:48.
	Are you sure you want to continue connecting (yes/no)?
	```

### 开启 GitHub Pages 服务 

>温馨提醒：执行这一步时，一定要确保之前的配置都已经完成了哦！

* 创建仓库
	* 点击右上角的 `+` 号后，选择 `New rpoository`，会进入如下界面：
	
	![13](13.jpg)

	* 在 `Repository name` 下填写 `Your-GitHub-Name.github.io`，`Description` 下填写一些简单的描述，另外记得勾选下 `Initialize this repository with a README` 选项，

## 测试一下
在执行完上面的步骤之后，我们可以在浏览器中输入 `https://Your-GitHub-Name.github.io/` 这个网址, 记住 `YourGitHubName` 代表你自己的 `GitHub` 用户名。

如果你能看到一个十分简陋的界面，就像下图所示一样,那么恭喜你。你已经成功开启了 `GitHub Pages` 服务啦。

> 字号大的文字内容是仓库名称，字号小的是仓库信息。

![14](14.jpg)

好了，关于域名和服务器的话题也算告一段落了。下一篇文章我们会说说博客框架的内容！