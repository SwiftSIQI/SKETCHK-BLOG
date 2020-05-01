---
title: A Guide Of Making Your Personal Blog - Part 5
comments: true
date: 2017-03-24 23:42:29
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

## 博客主题

使用 `Hexo` 框架可以使用不同样式的主题，它们被放在了[这里](https://hexo.io/themes/)，你可以随意挑选一个自己喜欢的。

估计这时候你已经选出了自己喜欢的主题，但你很快就会发现大部分主题的说明文档并不是那么友好，所以我们应该怎么办呢？

幸运的是 [`NexT`](https://theme-next.iissnan.com/) 主题拥有一套完整的说明文档，所以今天就拿它举例了。

### _config.yml 文件

在 `Hexo` 中有两份主要的配置文件，其名称都是 `_config.yml`。 其中，一份位于站点根目录下，主要包含 `Hexo` 本身的配置；另一份位于主题目录下，这份配置由主题作者提供，主要用于配置主题相关的选项。

为了描述方便，在以下说明中，将前者称为**站点配置文件**， 后者称为**主题配置文件**。

### 下载主题

在终端窗口下，定位到站点根目录下。输入以下代码：

```bash
$ cd your-hexo-site
$ git clone https://github.com/iissnan/hexo-theme-next themes/next
```

### 启用主题

与所有 `Hexo` 主题启用的模式一样。 当完成 `git cloen` 后，打开站点配置文件， 找到 `theme` 字段，并将其值更改为 `next`。

```
theme: next

```

到此，`NexT` 主题安装完成。下一步我们将验证主题是否正确启用。在切换主题之后、验证之前，我们最好使用 `hexo clean` 来清除 `Hexo` 的缓存。

### 验证一下

启动 `Hexo` 本地站点，并开启调试模式（即加上 `--debug`），整个命令是 `hexo s --debug`。 在服务启动的过程，注意观察命令行输出是否有任何异常信息，如果你碰到问题，这些信息将帮助他人更好的定位错误。 当命令行输出中提示出：

```
INFO  Hexo is running at http://0.0.0.0:4000/. Press Ctrl+C to stop.
```

此时即可使用浏览器访问 [`http://localhost:4000`](http://localhost:4000)，检查站点是否正确运行。

![02](02.jpg)


## 一些说明文档上没有说的事儿

估计你马上就会发现上面一节的内容和 `NexT` 官网上的内容一模一样，没错，确实是从官方文档上面 `copy` 过来的，因为官方文档已经把大多数问题都说清楚了，所以我就不把说明文档里的内容再重复一遍了，让我们说点文档里面没提到的吧。

### 创建 about 界面

在终端输入

```bash
hexo new page about
```

在主题的 `_configy.yml` 设置中将 `menu` 中 `about` 前面的注释去掉即可。

```bash
menu:
  home: /
  archives: /archives
  tags: /tags
  about: /about
```

需要注意的是，在 `about` 的页面中，`title` 是不会自动对齐的，如果想达到对齐的方式可以直接把 `title` 放到正文中，然后用 `hexo` 的居中对齐标签来实现该效果。

还有就是记得在这个 `md` 文件的 `front-matter` 里加入 `comments: false` 的语句，否则`about` 页面里面还会出现评论组件，如果你没有在博客系统里面添加过评论组件或者本意就想显示评论组件，这一条可以忽略。

### 博客 favicon 图标

先说说什么是 `favicon` 图标，看到下图左边蓝色的 `logo` 了么，它就是 `favicon` 图标。

![03](03.jpg)

制作这个东西很简单，将你喜欢的图片放到 [RealFaviconGenerator](http://realfavicongenerator.net/) 网站中进行转换，然后把获得的 `ico` 文件放到 `source` 文件夹中

最后在主题配置文件 `_config.yml` 中的 `favicon` 字段里填写 `ico` 图片的地址

```
favicon: /YourIcoPath
```

好了，重新部署一下你的网站就可以看到自己的 `favicon` 了！

如果没有看到，请不要着急，等上一会再看看，如果还是不行的话，请查看一下图片的路径和文件的格式是否有误。

另外需要补充的一点是 `favicon` 支持的图片格式有 `ico`，`png` 和 `gif`, 至于图片的大小一般是 `16 * 16` 或者 `32 * 32`, 详细信息可以查看 W3C 的[文档](https://www.w3.org/2005/10/howto-favicon)。

### sitemap 设置和蜘蛛协议

在开始之前，还是先说说 `sitemap` 是什么东西。

你可以把 `sitemap` 当做一个网站的说明书，它用于告诉搜索引擎这个网站里面有什么内容，以便用户在使用搜索引擎的时候，更容易找到这个网站的相关资源。

所以说做这件事的目的就是为了让他人能够更好的找到你博客里的内容，如果你并不希望他人找到的话，那么你完全可以忽略这节的内容。

下面我会具体演示一遍向 Google 添加 `sitemap` 的过程，走起！

1. 向 Google 证明你对自己博客的所有权，参考 `NexT` 文档中的 [`Google Webmaster tools`](https://theme-next.iissnan.com/third-party-services.html#google-webmaster-tools) 一节完成对博客的验证。
2. 在站点根目录下安装插件 [`hexo-generator-sitemap`](https://github.com/hexojs/hexo-generator-sitemap)
3. 在站点配置文件 `_config.yml` 中加入以下字段

	```bash
	sitemap:
	    path: sitemap.xml
	    template: ./sitemap_template.xml
	```

4. 重新生成博客文件，并确认 `public` 文件夹下是否多出一个 `sitemap.xml` 文件
5. 在 Google 的 `Search Console` 里填写 `sitemap` 地址，看看是否检测成功

	![04](04.jpg)

6. 大概过几分钟，在 `Google` 的搜索栏里输入 `site:YourDomain`,例如我的就是`site:sketchk.xyz`, 如果能够像下图一样展示出你的网站，那么就说明你的网站已经被正确收录了。

	![05](05.jpg)

弄完了 `sitemap` 的设置，我们来说说蜘蛛协议。

蜘蛛协议就是和蜘蛛之间的约定，哈哈，是不是很想死，不过这里的蜘蛛并不是真正意义上的蜘蛛，而是存在于网络中的网络爬虫，通过这个协议来规定这些网络爬虫到底能访问哪些资源，不能访问哪些资源。如果你对我说的东西还是感觉太不理解，我们来看看更官方的一种解释？

> 网络爬虫是一种按照一定的规则，自动地抓取网络信息的程序或者脚本。它是搜索引擎中最重要的一项技术，通过网络爬虫，我们可以将互联网中数以百亿计的网页信息保存到本地，形成一个镜像文件，为整个搜索引擎提供数据支撑。

好了，知道蜘蛛协议是什么了，那我们赶紧操作起来吧！

* 在站点根目录的 `source` 文件下新建 `robots.txt` 文件，添加以下文件内容

```
# hexo robots.txt
User-agent: *
Allow: /
Allow: /archives/
Allow: /categories/
Allow: /tags/
Allow: /about/

Disallow: /vendors/
Disallow: /js/
Disallow: /css/
Disallow: /fonts/
Disallow: /vendors/
Disallow: /fancybox/

Sitemap: http://www.sketchk.xyz/sitemap.xml
```

* 在 `Google Search Console` 的 `robots.txt` 中测试你的协议是否被正确识别！

好了，补充的内容就到这了，加上 `Next` 官网上的说明文档，我相信这个博客的功能已经基本够用了。

如果你想试试别的主题也未尝不可，生活嘛，就要多折腾才有意思！

下一章我们来说说自动部署的话题吧！