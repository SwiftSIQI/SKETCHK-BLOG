---
title: A Guide Of Making Your Personal Blog - Part 6
comments: true
date: 2017-03-24 23:42:33
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

## 自动部署
使用 `Hexo` 写博客是一件十分幸福的事，但让很不爽的是每次写完 `markdown` 文档后都要手动生成静态文件并重新部署到 `Github` 上，反复的操作不禁让人感到厌烦。

虽然上面的问题可以用脚本来解决，但这并不能解决所有的问题，假如我们换了一台电脑就需要重新安装 `Git`, `Node.js`，`Hexo`等等，这其实也挺麻烦的。

那么有什么好的解决方案呢？

## Travis CI 

在说 `Travis CI` 之前，我们先说说持续集成，持续交付和持续部署的概念。

* 持续集成（Continuous integration）就是频繁地（一天多次）将代码集成到主干。它的目的就是让产品可以快速迭代，同时还能保持高质量。它的核心措施是，代码集成到主干之前，必须通过自动化测试。只要有一个测试用例失败，就不能集成。

![02](02.png)

* 持续交付（Continuous delivery）在持续集成的基础上，将集成后的代码部署到更贴近真实运行环境的 "类生产环境"（production-like environments）中。比如，我们完成单元测试后，可以把代码部署到连接数据库的测试环境中。如果代码没有问题，可以继续手动部署到生产环境中。

![03](03.jpg)

* 持续部署（continuous deployment）是持续交付的下一步，指的是代码通过评审以后，自动部署到生产环境。持续部署的目标是，代码在任何时刻都是可部署的，可以进入生产阶段。持续部署的前提是能自动化完成测试、构建、部署等步骤。

![04](04.png)

说完这三个概念，我们来说说 `Travis CI`，你可以把它理解为一种服务。总之它的主要任务就是处理持续集成，持续交付和持续部署的相关事宜，而且很重要的一点就是它对 `GitHub` 的支持十分友好，加上它的免费策略，我们没有理由不去选择它，当然你也完全可以用 [Jenkins](https://jenkins.io/) 等类似的服务。

![05](05.jpg)

好了，`Travis CI` 的介绍到此结束了，我们下面就来说说如何利用 `Travis CI` 来实现自动部署吧。

### 解决方案

`Travis CI` 的工作原理是在每次提交 `commit` 到 `Github` 后，它会自动检测到我们的提交，此时 `Travis CI` 会生成一个虚拟机，它会根据我们写好的指令进行相关的操作。

那么我们到底要怎么干呢？我们来罗列下相应的步骤

* 我们在 `Github` 的博客仓库里新建一个 `blog-source` 分支，然后把博客的源码托管到这个分支
* 每当我们在本地写好了博文之后，把修改 `push` 到该分支
* `Travis CI` 上可以对这个项目的 `blog-source` 分支设置 `webhook`，每当检测到 `push` 的时候就去仓库 `clone` 源代码
* `Travis CI` 执行构建脚本
* `Travis CI` 把构建结果通过 push 部署到 master 分支的仓库里

如果看文字比较晕，可以看看我画的这张图。

![06](06.jpg)

我想结合着文字和图片，你已经明白我们要干什么了，所以如果我们实现了这个方案，在这样的自动化流程下，我们唯一需要做的事情就是 `push` 文章到 `blog-source` 分支，其他的事情交给 Travis CI 了。

是不是突然觉得世界都美好了呢，好了，开始干活吧！

> 这次的实践需要掌握一些基本的 `Git` 命令，如果对 `Git` 命令不熟悉，建议看看[廖雪峰的 Git 教程](http://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000)

## 创建 blog-source 分支

如果你对 `Git` 的相关操作比较熟悉，这一步应该相当简单，不过为了便于大家的理解和保持文章的一致性，我们还是把这个步骤中的相关操作写下来。

```bash 
$ cd /path/of/your/blog/source/
$ git init
$ git checkout -b blog-source
$ git add .
$ git commit -m "XXX"
$ git remote add origin git@github.com:XXX/XXX.github.io.git
$ git push -u origin blog-source
```

简单的解释下吧：

* 第 1 行是利用 `cd` 命令跳到站点根目录下
* 第 2 行是在这个文件夹下利用 `git init` 命令初始化仓库
* 第 3 行是在当前仓库创建一个 `blog-source` 分支
* 第 4，5 行是在进行一次 `commit` 操作，用来保存当前的修改，`XXX` 是用于记录此次 `commit` 的信息
* 第 6 行是将本地仓库与 `github` 中的仓库关联到一起，这里的 `XXX` 是你的 `GitHub ID`
* 第 7 行是将本地的分支推送到远端的 `blog-source` 分支中

最后的效果就是，当你点击 `branch` 按钮的时候，你会发现多了一个 `blog-source` 分支。

![07](07.jpg)

不过在结束这一步之前，让我们来看一下 `blog-source` 分支中的 `themes` 文件夹，你会发现 `next` 文件夹是一个空文件夹，里面并没有任何文件，这是为什么呢？

![08](08.jpg)

原因很简单，就是之前我们使用了 `git clone https://github.com/iissnan/hexo-theme-next themes/next` 的方式引入第三方主题，这个命令会在本地电脑的 `next` 文件夹下生成 `.git` 文件夹，这也就意味着 `next` 文件夹也使用了 `Git` 来进行版本管理，自然而然的这个 `next` 文件夹就不会纳入到站点根目录的版本管理中。

![09](09.jpg)

所以把 `next` 文件夹下的 `.git` 文件删除，重新进行一次提交。

> 如果在切换分支的时候留下来 master 分支的无用文件，可以直接删除。

## 配置 Travis CI

### 开启仓库的 Travis CI 功能
登录 [`Travis CI`](https://travis-ci.org/) 的网站，使用 `GitHub` 账户登录，成功登录后点击右边的头像，会弹出来个下拉菜单，点击 `Accounts` 按钮

![10](10.jpg)

这时候你会在界面上看到你在 `GitHub` 里的所有仓库，我们只需要打开存有博客资源的仓库即可。

![11](11.jpg)

这时候点击 `SketchK/SketchK.github.io` 会跳入这个项目的详情页，点击右上角的 `More options` 里的 `Setting` 按钮，就会进入下面的页面，我们可以打开 `build only if .travvis.yml is present` 和 `build branch updates`， 至于 `build pull request updates` 可以根据自己的实际情况来选择是否开启。

> 我完全不知道这四个选项代表啥意思啊 ？

好吧，如果你不愿意看 `Travis CI` 的[文档](https://docs.travis-ci.com/)话，我简单说一下这四个选项的意思

* `Build only if .travis.yml is present`：是否在 `.travis.yml` 文件存在的情况下开始才构建；
* `Build pushes`：是否在推送完这个分支后开始构建。
* `Limit concurrent jobs`: 是否限制同时执行的操作数量
* `Build pull request updates`：是否在 PR 合入后执行构建

![12](12.jpg)

到这一步， 我们已经开启了要构建的仓库，但此时 `Travis CI` 还没法帮助我们自动构建并部署。


### push 权限

我们在使用 `Github` 的时候，首先需要在 `Github` 上配置公钥，这是最基础的。那么，存在我们本地的私钥就是你的个人身份标示，如果你的项目 `git` 地址配置的是 `git@github.com:username/projectname.git`（相对的还有 `https://github.com/username/projectname.git`），当你对仓库进行一些操作（比如 `push` 等），则需要私钥进行身份验证了（这是自动验证的，如果是使用 `https` 的配置，则需要提供用户名和密码）。

我们在 `Travis CI` 上自动部署代码，就牵扯到了 `push` 操作，那么就需要提供私钥了。所以我们该怎么解决这个问题呢？

你会说把我们本地的私钥上传到 `blog-source` 里就 ok 啦，当然这么做理论上是没有任何错误的。

但你得知道，我们在 `GitHub` 的仓库可是公开的，也就是说谁都可以看到你的私钥了，这样的话，原本用来保密的私钥就失去了意义，更可怕的是，如果被一些居心不良的人盯上你的私钥，你可能就会遇上麻烦了。

那么我们该怎么办呢？

### 解决方案
目前网上主流的做法都是基于 `Travis` 提供了加密文件的支持，但具体的实现会有一些不同，什么意思呢？

一种方式就是我们可以在本地对文件（这里指私钥，比如 `id_rsa` 文件）进行加密，然后把加密过后的文件放在项目里，那么别人就无法获取里面的真实内容。然后我们在让 `Travis` 执行脚本的时候，在读取加密文件之前对文件进行解密（使用的解密密码需要提前在 `Travis` 上配置好），这样就可以达到不将文件内容暴露，并且让 `Travis` 获取到真实内容的目的了，大概流程如下：

![13](13.jpg)

第二种方式是利用了 `GitHub` 的 `Personal Access Token`，由于这个 `Token` 与 帐号密码 以及 `SSH Keys` 同样具有 `Github` 写入能力，因此只要使用 `Travis CI` 提供的加密工具来加密这个 `Token` 即可。 它的流程如下

![14](14.jpg)

虽然这两种方式都可以解决问题，但实际操作起来都很麻烦，难道就没有什么别的方式了么 ？

好在 `Travis CI` 提供了一个 `Environment Variables` 这个功能，那么我们完全可以使用一种简单的方式来实现我们的目的。

* 在 `GitHub` 的 `Personal Access Token 中创建 Token`
* 将 `Token` 的 `value` 值添加到 `Travis CI` 的 `Environment Variables`
* 对远程仓库的 `push` 操作使用如下方式来完成。

```
git push -f -q https://<username>:$GITHUB_TOKEN@github.com/<username>/<repo>
```

是不是看起来，还不错，下面就让我们开干吧！

> 这种 push 方法出自另一家 CI 服务公司 AppVeyor 的文档，详情请点击[这里](https://www.appveyor.com/docs/how-to/git-push)。

### 生成 Personal Access Token
登录 `GitHub` 的 `Setting` 页面，点击右边侧边栏的 `Personal access tokens` 后，进入下图所示的界面，此时点击右上角的 `Generate new token` 按钮

![15](15.jpg)

点击完以后会要求你重新输入一遍 `GitHub` 的密码，然后就会进入下面的界面，这个界面大概是在说我们应该赋予这个 `token` 的权限有哪些，根据目前的要求，我们需要把 `repo` 选项框里的权限勾上即可，如果你的博客有一些特殊的需求，就根据自己的情况选择吧。

![16](16.jpg)

勾选完后，点击最底下的 `Generate Token` 按钮，你就会得到一个 `token`，唯一需要注意的是 `token` 的值只会显示一次，所以记得把这个值保存一下。

![17](17.jpg)

### 在 Travis CI 上设置 Environment Variables

现在让我们回到 `Travis CI` 的界面中，把 `Token` 的 `Name` 和 `Token` 的 `Valve` 填到 `Environment Variables` 中，然后再点击下 `Add` 按钮。

![18](18.jpg)

现在 `Travis CI` 通过我们的设置已经与 `Personal Access Token` 产生了关联。

### 神奇的 .travis.yml 文件

在 Travis CI 的[官方文档里有这么一段话](https://docs.travis-ci.com/user/customizing-the-build/#The-Build-Lifecycle):

> Travis CI provides a default build environment and a default set of steps for each programming language. You can customize any step in this process in `.travis.yml`. Travis CI uses `.travis.yml` file in the root of your repository to learn about your project and how you want your builds to be executed. `.travis.yml` can be very minimalistic or have a lot of customization in it.

它的大体意思是说 `Travis CI` 为不同的编程语言提供了不同的编程环境，我们可以用一个叫 `.travis.yml` 的文件定义 `Travis CI` 需要执行的任务，同时我们需要把这个文件保存到站点根目录中。

既然 `.travis.yml` 是用来控制 Travis CI 的，那么我们应该如何编写这个文件呢？

在编写这个文件之前，我们不妨先说说 `Travis C`I 的构建周期，如果你不太理解构建周期的意思，我们来打个比方吧。比如我们办事的时候总会有个第一步，第二步，直到最后一步，我们可以把这个过程理解为构建周期，而第一步，第二步之类的概念理解为这个构建周期的时间节点。

那么 `Travis CI` 的构建周期主要分两步：

1. `install` 用于安装构建所需要的一些依赖
2. `script` 运行构建脚本

而 `Travis CI` 提供了下面 12 种标签来描述一次构建周期的各个时间节点：

* `Install apt addons`: 在这个节点中，可以使用 `apt` 包管理器来安装一些依赖包
* `Install cache components`: 这个节点的作用是缓存一些不常改变的东西，可以加快编译速度
* `before_install` 这个节点是指在 `install` 步骤之前的动作
* `install`: 这个节点的作用是用来安装依赖或者工具
* `before_script`: 这个节点是指在 `script` 执行之前的动作
* `script`: 这个节点下的动作是由 `bash` 来解析的，这里放构建的主要步骤
* `before_cache`: 这个节点主要用于清除 `cache`
* `after_success` or `after_failure`: 这个节点是指在构建成功或者构建失败后的动作
* `before_deploy`: 这个节点是指在部署之前的动作
* `deploy`: 这个节点是执行相应的部署操作
* `after_deploy`: 这个节点是指部署之后的动作
* `after_script`: 这个节点就是在 `script` 执行之后的动作

看了这么多的标签是不是有点懵，其实在实际的使用过程中，我们并不会用那么多的标签。

说的有点多了，直接把 `.travis.yml` 的代码放上来吧。

```yaml
language: node_js

node_js: stable

branches:
  only:
  - blog-source

cache:
  directories:
  - node_modules

before_install:
  - export TZ=Asia/Beijing

install:
  - npm install

before_script:
  - git config --global user.name "SketchK"
  - git config --global user.email "zhangsiqi1988@gmail.com"
  - git config --global push.default simple

script:
  - git clone --branch master https://github.com/sketchk/sketchk.github.io.git .deploy/sketchk.github.io
  - hexo generate
  - cp -R public/* .deploy/sketchk.github.io
  - cd .deploy/sketchk.github.io
  - git add .
  - MESSAGE=`date +\ %Y-%m-%d\ %H:%M:%S`
  - git commit -m "Travic CI Auto Build & Blog updated:$MESSAGE"
  - git push --quiet https://SketchK:${TraviS CI - Hexo}@github.com/SketchK/SketchK.github.io.git

```

现在我们对这个文件分段说明下：

#### install 标签之前的操作

`install` 标签之前我们填写了 `language`, `node_js`, `cache` 以及 `branches` 标签，

`language` 和 `node_js` 分别告诉 `Travis CI` 我们需要使用 `node.js`，它的版本是 `stable` 的。

`branch` 部分的代码是告诉 `Travis CI` 只监控 `blog-source` 分支的变化，只有这个分支产生变化了才会执行下面的操作。或许你会好奇 `blog-source` 分支具体产生何种变化才会开始构建？其实我们在前面已经设定过了，如果忘了，请仔细看看 `开启仓库的 Travis CI 功能` 一节的内容。

#### before_install 标签中的操作

在这里我们引入了一个 `TZ` 变量，它的值是当前的北京时间。

#### install 标签中的操作

你肯定会好奇 `Travis CI` 是如何生成静态网页的，因为我们好像似乎没有告诉它应该去安装 `Hexo` 和其他相应的 `Hexo` 插件，只是在 `.travis.yml` 中输入了一句 `npm install` 的命令。

这时候我们需要看看几个没说过东西，下图是我们站点根目录的截图：

![19](19.jpg)

第一个就是 `package.json`, 打开它会发现该文件指定了我们所需要的 `Hexo` 和相应插件，而它是怎么知道的呢？其实这个文件是我们在本地配置 Hexo 及其相关插件时自动生成的，所以通过这个文件，`Travis CI` 在 执行 `npm install` 的时候就知道应该去下载何种软件，保证我们在执行 `hexo g` 的时候不会出错。

![20](20.jpg)

第二个就是 `node_modules`，Travis CI 通过 `package.json` 文件的内容去下载 `Hexo` 和相应插件，但下载完的东西需要放到哪里呢？`node_modules` 就是用来解决这个问题的。不过我们在 `blog-source` 分支中并没有看到这个文件夹，一方面是 `.gitignore` 里标明了排除这个文件夹，另一方面就是我们把 `node_modules` 文件夹里的内容 `push` 到 `blog-source` 上是没有意义的, 安装 `Hexo` 的工作可不是简单的把 `node_modules` 文件上传到 `Travis CI` 上就完事的。

#### before_script 标签中的操作
在执行具体的工作前，我们需要告诉 `Travis CI` 执行 `Git` 命令的操作者是谁，否则你的操作会出问题哦！另外记得加 `--global` 参数

#### script 标签中的操作
这一段的代码与我们之前的 `deploy.sh` 文件的代码十分相似。不过需要需要注意的是我们这次 `push` 的地址是与之前的方式不一样的，它的通用样式如下：

```sh
git push --quiet https://<username>:$GITHUB_TOKEN@github.com/<username>/<repo>
```

## 几个需要注意的地方

* 如果你的 `Travis CI` 没有提示任何错误，但打开网站发现一片空白，那么八成就是你的 `themes` 文件夹出现了问题，可以仔细排查下这一块的代码和逻辑。

* `- git config --global push.default simple` 是 `Git 2.0` 带来的改变，如果我们不写这句话，在 `Travis CI` 里会看到一些 `warning` ，不过没有它似乎也不会有什么问题，只是后面的参数为 `simple` 的时候，执行 `git push` 且 没有指定分支时，只有当前分支会被 `push` 到对应的远程仓库。

* 如果你的 `Token` 中有一些特殊操作符，那么在进行 push 操作时，你可以像我的示例代码中一样使用 `{ }` 的方式把 `Token` 包起来，如果没有空格直接按照通用样式就可以了。

* 在调试的时候可以不用加 `--quiet` 参数，调试成功后，一定要在 `push` 后面加 `--quiet` 参数并换一个新的 token ，因为不加这个参数的时候，Travis CI 会把 `$GITHUB_TOKEN` 的值显示出来。你问我为什么会显示出来？因为你这个仓库 Travis CI 记录是任何人都可以看到的啊。

* 至于为什么把 `push` 操作放到 `script` 中，这是因为 `Travis CI` 判断构建项目的成功与失败主要在 `script` 中完成，如果我们把 `push` 操作放到 `after_script` 或者 `after_success` 中，即使 `push` 失败了，`Travis CI` 也会显示此次的构建成功。所以为了能够保证自动部署的成功，我把这个步骤放到了 `script` 中。

## 测试一下
做完这些工作后，我们把修改好的文件 `push` 到 `blog-source` 分支吧。然后再回到 `Travis CI` 的控制台看看发生了什么？

![21](21.jpg)

果不其然，`Travis CI` 已经检测到变化并进行构建部署了！ 点击下面的 `Job Log` 可以查看整个构建过程。

如果构建失败的话，也不用害怕，`Job Log` 里会有详细的信息。这些信息足够你去解决它们了。

现在，你终于完成了个人博客的自动部署工作了！

