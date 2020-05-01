---
title: A Guide Of Making Your Personal Blog - Part 4
comments: true
date: 2017-03-24 23:42:23
updated:
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

## 博客框架
我们在前面说过 `GitHub Pages` 服务允许我们访问已经写好的 `HTML` 等静态文件，所以我们对博客框架的需求就是产出静态文件。

幸运的是能完成这个需求的框架有很多，例如 `GitHub Pages` 官方推荐的 [`Jeklly`](https://jekyllrb.com/), 也有基于 `Jeklly` 开发的 [`Octopreess`](http://octopress.org/), 不过我今天要说的是另外一个博客框架：[`Hexo`](https://hexo.io/zh-cn/)。

所以废话少说，直接来看看 `Hexo` 吧。

## 安装 Hexo 的前提

安装 `Hexo` 需要电脑里面安装 `Node.js` 和 `Git`。

`Git` 我们已经在前面安装过了，所以现在我们只需要安装 `Node.js` 了，具体怎么安装就不废话了，就列出官网链接吧：[https://nodejs.org/en/](https://nodejs.org/en/)

如果你不知道自己是否安装了，在终端里面输入 `$ node --version` 来检查就好。 
 
## 安装 Hexo

所有必备的应用程序安装完成后，在终端下面输入以下命令来安装 `Hexo`。

```bash
$ npm install -g hexo-cli
```

## 创建我们的博客吧！

安装 `Hexo` 完成后，请执行下列命令，`Hex`o 将会在指定文件夹中新建所需要的文件，下面的代码中 `<folder>` 指的是文件夹的地址。

```bash
$ hexo init <folder>
$ cd <folder>
$ npm install
新建完成后，指定文件夹的目录如下：
```

> 在之后的文章里，`<folder>` 文件夹会统一称为**站点根目录**

创建完毕后，你大概会看到这样一个目录结构。

![02](02.jpg)

>实际生成的目录结构和 Hexo 官网给出的目录结果会有一些出入，不过估计是作者许久没有更新了，总之按照操作来就好。

### 预览我们的博客

在站点根目录中，输入`$ hexo server` 后会出现如下的提示

![03](03.jpg)

所以让我们在浏览器里面输入一下 `http://localhost:4000/` 吧，然后你看到了什么？

![04](04.jpg)

以后我们在实际的使用过程中，每当修改完网站里的东西就可以通过这种方式来预览，而不是直接发布到网上。

### 生成静态网站

好了,假设你已经修改好了网站，我们现在就要生成静态文件了，我们应该怎么做呢？

我们只需要在站点根目录中输入 `$ hexo generate` 即可，执行完以后大概是下图这个样子。

![05](05.jpg)

此时，你会发现站点根目录里多了一个 `public` 文件夹，这个文件夹里的东西就是博客的静态文件，所以下一步，不用我说，你也应该知道我们要干什么了吧？

![06](06.jpg)

### 部署我们的博客

你是不是想起来我们要干什么了啊？ 如果忘记的话，也没事，我再重说一遍就好：

**那就是把刚才生成的静态文件上传到 GitHub 仓库中。**

如果你对 `Git` 的使用比较熟练，你完全可以通过 `Git` 把这些本地文件 push 到对应的仓库中。

如果你对 `Git` 并不熟悉，那该怎么办呢？ 

还好 `Hexo` 给我们提供了另一种选择，通过它我们可以很方便的把静态文件部署到对应的仓库中，不过在使用这个功能前，我们需要做一点设置。

* 首先，打卡站点根目录中的 `_config.yml` 文件，找到 `Deployment` 字段，填写下面的内容，

```yml
deploy:
  type: git
  repo: git@github.com:yourname/yourname.github.io.git
  branch: master
```

* 然后，在终端里安装一个用于帮助我们上传文件的扩展工具：

```bash
$ npm install hexo-deployer-git --save
```

* 最后，在站点根目录中输入 `$ hexo deploy`

![07](07.jpg)

> 眼尖的朋友一定发现，我在终端里面实际输入的是 `hexo d` 而不是 `hexo deploy`, 这是因为 `Hexo` 对这些命令做了优化，大部分命令我们都可以使用简写的方式来执行，例如 `hexo g` 完全等同与 `hexo generate`，是不是对 `Hexo` 又产生了一点好感呢？

现在在你的浏览器里面输入自己购买的域名吧！What !? 为什么是 404 页面。

其实道理很简单，打开你的 `GitHub` 仓库，发现有什么不对的地方么？ 

你会发现库里面原有的 `CNAME` 文件没了，这也就是说 `Settings` 里面的 `Custom Domain` 没了，所以就会 404 喽。

是不是知道原理以后，出现这些问题也不会怕怕了，现在我们赶紧填写好 `Custom Domain` 吧，然后再次在浏览器里输入购买的域名！ 嘿嘿嘿！

![08](08.jpg)

恭喜你！你的个人博客终于有个雏形了！

## 改进一下

### 每次都要填写 Custom Domain 参数？ 
我想聪明的你马上会发现一个问题，似乎我们每次 `hexo deploy` 后，都要去修改 `Custom Domain` 这个参数，是不是感觉心好累 ?

解决这个问题并不难，我们只需要在 `source` 文件夹下面创建一个名为 `CNAME` 的文件，然后在这个文件里写上我们购买的域名就可以了。

为什么呢？因为 `Hexo` 的工作原理是将 `source` ，`theme` 等文件夹里的内容进行一顿处理，就变成 `public` 文件夹里的内容，如果我们在 `source` 里面放入了一个它无法处理的文件，Hexo 就会原封不动的把它拷贝到 `public` 文件夹中，这样也就解决了我们每次要修改 `Custom Domain` 的问题。

> 如果你不明白为什么加了一个 `CNAME` 文件就等于修改了 `Custom Domain`，请再阅读上一篇文章里 `解释下在 GitHub 里的操作` 一节的内容

### hexo deploy 不好使了？

有时候你会发现 `hexo deploy` 不好使了。我该怎么办？很简单，老老实实的用 `Git` 吧! 

什么，还要我教一下 `Git` 的使用，我建议你还是看看[廖雪峰老师的 git 教程](http://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000)。

如果你对每次敲击这些无聊且重复的 `Git` 代码感到厌烦，那我们来说说如何用一个脚本替我们完成这些工作吧!

Talk is cheap , Show me the code!

```sh
$ #!/bin/sh
$ [ -d .deploy/<Name>.github.io ] && rm -rf .deploy/<Name>.github.io
$ git clone git://github.com/<Name>/<Name>.github.io.git .deploy/<Name>.github.io
$ hexo generate
$ cp -R public/* .deploy/<Name>.github.io 
$ cd .deploy/<Name>.github.io 
$ git add .
$ git commit -m "update"
$ git push origin master
```

需要说明的是，你要把 `<Name>` 换成你的 `GitHub` 账户名，然后把它保存为 `deploy.sh` 并放到你的站点根目录下。

有了它，我们只需要每次在终端里面输入 `sh deploy.sh` 即可以完成部署了，是不是很完美，让我们看一下它的实际运行效果。

> 不用纠结你终端里面展示的内容和我的不太一样，重要的是最后的那句 **Everything up-to-date**。

![09](09.jpg)

我说过凡事还是要搞明白，要不以后出问题都不知道该怎么办，所以各位还是耐着性子让我把这段 `shell` 脚本解释下。

* 第一行是对 shell 的声明，说明你所用的是哪种类型的 `shell` 及其路径所在。
* 第二行里 `&&` 左边是在判断是否有 `.deploy/<Name>.github.io` 文件夹，`&&` 右边是在说删除 `.deploy/<Name>.github.io` 文件夹，所以这行的意思就是如果有这个文件夹，就删掉它
* 第三行的意思是把你 `<Name>.github.io` 仓库的文件拷贝到本地的 `.deploy/<Name>.github.io` 文件夹下面
* 第四行就是让 `hexo` 生成静态文件
* 第五行就是说把 `public` 文件夹下面的东西放到 `.deploy/<Name>.github.io`
* 第六行就是跳转到 `.deploy/<Name>.github.io` 目录下
* 第七，八，九行就是正常的 `git` 操作了。

所以现在你明白了每一步的作用，如果在使用过程中出现了问题就不必慌张了。

好了，这两个头疼的问题终于解决了！让我们继续下面的内容。

## 发布一篇新文章
现在是不是迫不及待的想发布一篇博客了？

你只需要输入 `hexo new <名称>`， 然后你会发现在 `source/_post/` 里多一个文件，文件名与你输入的 `<名称>` 相同。

![10](10.jpg)

不过，你很快会发现这是一个以 `md` 为后缀名的文件，这说明它是一个支持 `Markdown` 语言的文件，那么什么是 `Markdown` 呢？

> `Markdown` 是一种轻量级的「标记语言」，它的优点很多，目前也被越来越多的写作爱好者，撰稿者广泛使用。看到这里请不要被「标记」、「语言」所迷惑，`Markdown` 的语法十分简单。常用的标记符号也不超过十个，这种相对于更为复杂的 `HTML` 标记语言来说，`Markdown` 可谓是十分轻量的，学习成本也不需要太多，且一旦熟悉这种语法规则，会有一劳永逸的效果。

所以你大概已经明白了，`MarkDown` 是一种用于写作的语言，你如果想使用目前的这套技术方案来搭建自己的博客，就必须使用这个语言，其实说真的，这个没啥难度，可以参考下面的几个资料学习下：

* `Markdown` 发明者写的说明文档：[http://daringfireball.net/projects/markdown/syntax]()
* 热心群众翻译的中文版：[https://github.com/othree/markdown-syntax-zhtw/blob/master/syntax.md]()
* `GitHub` 提供的 `Markdown` 游戏，边学边玩：[http://www.markdowntutorial.com/]()
* `Markdown` 语法的 `cheatsheet`: [https://guides.github.com/pdfs/markdown-cheatsheet-online.pdf]()

至于用什么来编辑 `md` 文件，我自己是目前是在用 [`MacDown`](https://macdown.uranusjr.com/)，受限于 `MacDown` 的归纳能力，我目前在观望 [`Quiver`](http://happenapps.com/#quiver)和 [`MWeb`](http://zh.mweb.im/)。

总之，这些都不重要，哪个看着顺眼就用哪个！

![11](11.png)

### 关于创建文章的一点补充

如果仔细看 `Hexo` 的文档，我们就会知道创建文章的完整命令如下：

```bash
$ hexo new [layout] <title>
```

我想跟大家说的有两点：

* 第一点，就是 `title` 参数为什么我会用 `“ ”` 号包起来，因为不用这个双引号包起来的话，我们文章的标题如果有空格的话，就会出问题了，用这种方式的话，即使你再双引号里面有空格，`Hexo` 在实际生成文件名的时候也会用 `-` 来替代它，所以这只是一个避免错误的小技巧而已。

* 第二点，就是 `Hexo` 支持模板这个概念，对应到这个命令里面就是的 `layout` 参数，对应到我们的文件里就是指 `scaffolds` 文件夹，那么这个概念是什么意思呢，也就是说我们不同的文章会有一些固定的格式，我们可以为这一类固定格式的文章建立通用的模板，这样我们就不用在每次创建文章的时候输入相同的内容。关于这个话题的一些高级使用技巧我就不在这里展示了，你完全可以直接查看 `Hexo` 的官方文档：[https://hexo.io/zh-cn/docs/](https://hexo.io/zh-cn/docs/)

现在你已经搭建好了博客，也知道如何简单的使用这个博客系统了，那么这一篇的内容也就到此结束了。

下一篇我们会来讨论一下如何让我们的博客看起来更漂亮点。