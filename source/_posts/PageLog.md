---
title: Github Pages(二)：个人网站的功能插件
date: 2017-10-22 01:55:42
tags:
- 个人网站
categories: 工作
---

在[Github Pages(一)：一个最基础的个人网站](https://hans2936.github.io/2017/10/21/PageBasic/)中，我们已经介绍了怎样用Github Pages做出一个网站框架，例如完全重复出 [HTML5 UP](https://html5up.net) 网站上的某个模版。在熟悉该模版之后，你就已经可以编写博客页(html)，并且链接到首页了。甚至可以有出色的排版与配图。

但对一个博客网站来说这样还不够，很多互动功能也是需要手动添加、单独理解的，包括评论框、搜索栏、点击量计数、分享按钮、网站导航、网页排版和图片滚动等。

这一篇文章是一个大合集，总结我曾经见到的个人网站互动功能，以及各个解决方案的优劣。

注意：虽然列出了这么多插件，但是这些只是技术储备，如果你是一个网站新手，建议先阅读[Github Pages(三)：使用Hexo博客生成工具](https://hans2936.github.io/2018/06/06/HexoLog/)。

<!--more-->

## 搜索功能
搜索功能包括两部分，一是在站内快速找到某一篇博客，这个方便网站的使用；二是在站外的搜索引擎上可以查到你的博客，这个有助于网站的推广。然而我在选择搜索工具的时候发现静态网站是无法自己写站内引擎的，所以我的搜索功能就暂时借助第三方（谷歌）实现了。

> 注：这篇文章写的早，后来我在[Github Pages(四)：Hexo进阶-NexT本地搜索引擎的移植](https://hans2936.github.io/2018/06/15/HexoSearch/)搞定了站内搜索。

### php站内搜索的失败尝试
在站内搜索到某个文件，直接写一段PHP遍历本站文件是最靠谱的，所以我就搜到了一位大神写的code（[月光软件文章](http://www.moon-soft.com/download/info/1492.htm)）。但是这段code有两个函数（`eregi`和`split`）在PHP7 过期了。所以我修改了一下（使用`preg_match`和`explode`），成了这个文件[search.php](https://github.com/hans2936/Hans_Blog_Old/blob/master/search.php)。
使用的时候，只要在网页中调用这个search.php就可以了，比如说

```
<form method="GET" action="search.php">
<input type="text" name="keyword" placeholder="PHP站内搜索" />
</form>
```

这个搜索引擎目前的效果是简单的列出全部搜出来的文件链接，在这个效果上美化一下就可以了。

在测试PHP文件的时候，必须要打开本地Web服务，启动方法（Mac）参见本文的“网页编辑后的预览”一节。

不成功的原因是显然的，Github Pages是静态的，不能运行php程序。不过这段学习经历挺有意思。

### 谷歌自定义搜索
引入谷歌自定义搜索有两个工作，一是建立一个自定义引擎（[谷歌自定义搜索](https://cse.google.com)），二是给谷歌提供网站信息（[Google Search Console](https://www.google.com/webmasters/tools)）。
点击[谷歌自定义搜索](https://cse.google.com)，创建一个搜索引擎，然后在“外观”选项卡里就可以找到“保存并获取代码”按钮了。把这个代码加入你要显示结果的网页即可。
谷歌自定义引擎的搜索框有点丑，十有八九跟你的主题不一样，好在这个引擎也支持分页显示。本站的搜索功能就是用分页的方式实现的：首先选择一个分页的引擎主题，然后单独建立一个Search.html，加入刚才说的自定义引擎代码，然后在其他网页需要加入搜索框的地方输入一个原生主题的搜索框。

```
<form method="Get" action="blogs/Search.html">
<input type="text" name="q" id="s" placeholder="站内Google" />
</form>
```

这样每次搜索就会跳转到搜索结果页了。当然也可以用“弹出”的主题，把`action`直接放到本页，就可以有直接在本页显示结果的效果。
给谷歌提供网站信息也是很重要的，这决定了谷歌搜索你网站时结果的质量。我首先在[Google Search Console](https://www.google.com/webmasters/tools)注册了一个账户，添加本站，然后在管理界面的“抓取”、“站点地图”添加了一个写着本站所有网页地址的sitemap.txt文件，这样谷歌就知道本站以及各个网页的位置了（只添加站点不提交地图也行，这关系到谷歌爬取网页的效率)。

![正确提交Sitemap之后](PageLog/GoogleSite.jpg)

这个解决方案可以得到在网站内显示谷歌搜索结果的效果。但是当我频繁修改网站的时候，谷歌的滞后是很明显的。

为了提高效果，还可以提交更多的信息给谷歌，包括一些关键词，这些我就没有深究了。

### 百度自定义搜索

百度自定义搜索其实跟谷歌差不多，都包括自定义一个引擎加入网页（[百度自定义搜索](http://zn.baidu.com/cse/searchbox/)），和为百度提供网站信息（[百度站长工具](http://zhanzhang.baidu.com)）两个步骤。这里就不赘述了。
但是百度跟Github Pages的兼容性似乎是有问题的，百度抓取github.io的效率明显比谷歌低很多（搜索同样的Github Pages网页就知道了，而且我把本站提交给百度没有效果）。所以本站的第三方搜索引擎还是选择了谷歌。

## 评论功能
评论功能是博客的重要互动功能，但是评论是动态的，Github Pages是静态的，需要借助一个第三方服务来实现。

### 新浪评论框的失败尝试
在网上搜了一下之后，我最先想到的解决方案是[微博开放平台](http://open.weibo.com)，因为微博几乎谁都有，从演示的效果来看，评论功能使用微博评论箱，再结合微博分享、微博赞按钮两个插件，就跟在微博内部评论是一样的了（甚至还有微博秀这种直接展示微博个人页的功能），等于把个人博客和微博社区结合起来了。

作为官方插件，需要先申请一个Appkey。这件事有两个步骤，一是在[微博开放平台](http://open.weibo.com)申请一个账户，然后提交身份验证。二是在[微博开放平台：网站接入](http://open.weibo.com/connect)接入你的网站，然后提交审核。
Github Pages显然是境外网站，不需要提供备案信息，只要选择境外网站的验证，把域名查询（[站长之家Whois查询](http://whois.chinaz.com)）和ip查询（[阿波罗ip查询](http://www.iapolo.com/ip/)）的结果放到一张图片上上传就行了。
在网页中加入评论箱本身并不复杂，只要点击[微博开放平台：评论箱](http://open.weibo.com/widget/comments.php)，然后把底部代码加入网页就行了。其他插件也是一样的，本地预览可以很快看到效果。

**然而，微博插件在Github Pages是不能正常使用的。**一开始我并不知道原因，后来用Chrome浏览器打开网页的时候，发现这些插件实际上是被浏览器安全政策阻止了。这对个人网站来说肯定是不可接受的，因为你不能要求访问者为了访问你的网站而修改浏览器设置。
进一步去查这些插件不安全的原因，就发现微博的插件是不支持https的，而现在Github Pages用的是https的域名。进入Github Pages库的设置，会发现这么一个无法更改的选项。

> Enforce HTTPS
> — Required for your site because you are using the default domain (hans2936.github.io)

也就是说除非我自己买一个域名，否则这个选项是不能修改的。再说好像没必要为了一个评论系统把网站从https改成http，这个方案也就搁浅了。
### 来必力
第二个方案是找一些专门提供评论服务的网站，我找到了现在比较流行的[来必力](http://www.laibili.com.cn/city-demo)。
加入这个服务就比新浪评论箱简单多了。先注册一个账户，然后点击“体验”、“管理中心”，就可以看到代码了。
这个服务的好处是可以用国内所有的社交网站登陆（QQ、微信、微博），而且界面也很好看。
但是来必力有一个硬伤，就是加载速度太慢了。感觉博客前两段看完了这个插件还没加载出来，一开始我觉得无伤大雅，直到我发现这个评论框不加载完，本站的导航栏就弹不出来。这就有点影响体验了，所以这个方案也没使用。

### Gitment
最后我终于找了一个比较合适的评论系统：[教程：Gitment](https://imsun.net/posts/gitment-introduction/)。
这个教程已经很详细了，唯一需要注意的是它跟search.php一样必须开本地Web服务器才能查看效果。
这个评论系统是直接把Github issues搬到你自己的网页。它有三个好处，第一个就是把Github pages和Github issues联系起来，这样连评论带网站都在一个库里面；第二个好处是Gitment是一个出色的评论框，支持markdown语法，还自带点赞；第三个好处是加载速度快。唯一的坏处就是只能使用Github账户评论，不过我觉得可以接受，因为本来就是技术性博客，提高评论门槛也未必是坏事。
所以这就是我网站的评论框解决方案了。

（2019年12月更新：忽然发现Gitment不能用了，通过查看Gitment的Github项目[Gitment issue 188](https://github.com/imsun/gitment/issues/188)解决了问题。又发现出现了类似的工具Gitalk。）
 
 (2020年4月更新：发现上次解决方案重新push的时候还是有bug，改用 [Gitalk](https://github.com/gitalk/gitalk/blob/master/readme-cn.md) 了，另外在外网，[Disqus](https://disqus.com)是可以用的。)

![Gitment的使用效果](PageLog/Gitment.jpg)

## 分享按钮
分享也是博客网站重要的互动功能。在国内比较重要的就是支持微信和微博的分享，尤其是微信。
本站的评论栏上面都有分享按钮，其中微信还可以扫码，这个功能来自于[Github:share.js](https://github.com/overtrue/share.js)。这个Github文件夹的Readme有一个教程，不过我是把整个包下载下来，仔细看了一下demo网页，然后照搬到本站的。
现在这个分享按钮并不是腾讯或新浪提供的原生插件，对微博、QQ来说是跳转到一个分享页，对微信来说是在微信中打开本网址，进而实现分享的。因为有微博评论箱的经验，我也就不强求原生插件了，这个效果还可以。

## 计数
在博客列表统计评论数、点击数、赞数可以让访问者快速找到比较火的文章。也可以为进一步排序提供依据。统计也是动态的，要借助一个第三方服务。
本站的博客都有一个点击量统计，这些数字是提供自[网站:不蒜子](http://busuanzi.ibruce.info)的。教程链接为[教程:不蒜子](http://ibruce.info/2015/04/04/busuanzi/)，非常简单，粘贴里面的代码进入网页即可。这个统计分本站点击，本站访问人次和本页点击。
但是现在点击统计仅限于本页，没有传递到博客列表。另外，评论和赞的统计比点击数更有意义，Gitment应该可以做到但还没有官方的解决方案。这些以后再去开发吧。

## 网页设计
### 使用Markdown编辑网页博客
在写博客的时候，如果直接在html里面敲字体验并不好。因为要不断地敲一些`<p>`, `<a>`,  `<li>` 这些格式的开头和结尾。虽然latex写习惯了之后这也没什么，但是个人博客还有更轻松的办法来写，就是用Markdown编辑器。
[Markdown语法](https://zh.wikipedia.org/wiki/Markdown)是一个非常适合写日志的语法。我直接从App Store里面搜了一个编辑器，叫[MWeb](http://zh.mweb.im)，还是很好用的。写完之后可以直接导出为html格式，然后取其中正文部分粘贴到博客网页即可。这个过程只需要注意检查图片、列表等特殊部分跟网站模板的兼容性就行了。

举例来说，Mweb支持数学公式的编辑和导出，这是用[MathJax](https://www.mathjax.org)来实现的。MWeb在导出html时会自动转换格式，但此时要注意检查网页的引用：

```
<script type="text/javascript" src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.1/MathJax.js?config=TeX-AMS-MML_HTMLorMML"></script>
 <script type="text/x-mathjax-config">MathJax.Hub.Config({TeX: {equationNumbers: { autoNumber: "AMS" } }});</script>
```
更简单的就是使用Hexo等博客发布工具，这样省去了自己导出html，直接编辑好Markdown，再加一条`hexo g`命令就变成网页了。

### 网页编辑后的预览
网页预览的方式有四种：

一，直接用浏览器打开html，这肯定是最快的。但是有些插件不能显示，比如不能测试上述的Gitment和search.php。但是大部分工作还是能胜任的。

二， 使用[Adobe Dreamweaver](https://zh.wikipedia.org/wiki/Adobe_Dreamweaver)这样的专业工具检查，我并没有开发太多功能，不过很重要的是检查网页的报错，以及批量查找/替换某个词句。

三， 打开本地Web服务器。这是能完美运行网站上所有插件，甚至可以测试动态模块的办法。具体步骤参见[简书博客:Mac OS X Web Server](http://www.jianshu.com/p/d006a34a343f)。比较重要的命令就是`apachectl`：

```
sudo apachectl start
sudo apachectl stop
```

启动`apachectl`后，把你的网站文件夹拷贝到`/Library/WebServer/Documents`，然后打开`http://127.0.0.1/网站文件夹`就可以了，博客里面还写了一些高级功能，如怎样更改路径和配置等。

四，如果用了Hexo博客工具，一条`hexo s`命令就可以了。

## 图片滚动
### 图片滚动插件
这个功能对博客来说可有可无，不过对于普通网站来说，宣传新闻，推销产品什么的还是很有用的。我觉得很有意思就留在首页了，循环播放本站的日志，这应该是本站唯一一个没那么朴素的效果。
本站的插件叫做[Nivo Slider](https://github.com/Codeinwp/Nivo-Slider-jQuery)，用起来比较舒服，因为图片和图例是分开的，设置也很多。我试了好几个素材网站的插件都没有这个好。但是Nivo Slider有一个毛病是图片的尺寸最好都一样，而且不能太方正（我都是1920x960的图），否则电脑端还好，但在移动端会出现显示问题。

点击链接[Github:Nivo Slider](https://github.com/Codeinwp/Nivo-Slider-jQuery)，下载并学习demo就可以把它加入你的网页了。

### 手机滑动的实现
Nivo Slider本身是不支持手机的，找那两个小按钮实在比较麻烦。所以我找到了一位高人的博客 [Will Rees](http://willrees.com/2013/02/make-your-nivo-slider-touch-capable/)。他在里面讲了怎样利用一段支持手机滑动指令的代码 [Hammer.js](http://hammerjs.github.io)实现Nivo Slider的左右滑动。
用手机打开本站主页就可以看到效果了。不过因为我只是在Nivo Slider上面稍作修改，在手机上是只能左右滑动，不能手指悬停的。完美的解决方案可能是为手机单独找个插件。

## 回到顶部按钮
在网页很长的时候，回到顶部按钮就很有用了。这个需要一段javascript code实现。我是在Google上随便找了一个，改了改图标和大小，也可以看文件[back.js](https://github.com/hans2936/Hans_Blog_Old/blob/master/assets/js/back.js)。确定按钮图片的位置对之后，引用一下就可以了。

## 关于、版权
考虑到现在转载不署名的情况到处都是，在某个位置声明一下网站内容的版权也是有必要的。
在网页中加入版权声明其实很简单，只要进入版权网页[Creative Commons](https://creativecommons.org/licenses/by-nc-sa/3.0/cn/)，在这一页最下方点击“[将这个授权协议使用在您自己的作品中](https://creativecommons.org/choose/results-one?license_code=by-nc-sa&amp;jurisdiction=cn&amp;version=3.0&amp;lang=zh)”就可以找到代码了，或者直接点这里的链接也行。

这个证书是一个署名-可修改-非商业用途证书，如果你需要其他政策，也可以去Google其他证书的名字，基本上第一个弹出来的就是这些Createive Commons协议了。

## 总结
至此我总结完了学习过程中遇到的各种功能插件。我属于业余，很多细节还在学习中，相信会有更好的解决方案。
