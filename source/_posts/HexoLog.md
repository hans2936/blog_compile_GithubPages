---
title: Github Pages(三)：使用Hexo博客生成工具
date: 2018-06-06 20:55:42
tags: 
- 个人网站
categories: 工作
---

最近用 Hexo 搭建了工作网站 [CERN:Shuo Han](https://cern.ch/shhan)，发现这个工具发文章就像是发朋友圈一样简单。虽然我以前用网页拼凑[我的老网站](https://github.com/hans2936/Hans_Blog_Old)花了很多时间，但我还是决定换了，写作和发布体验还是很重要的。
我的理解是，这是一个自动生成静态（html+css+js）网站的工具，主题里是很多模块化的网页，可以通过.yml配置文件运用这些模块，把MarkDown(.md)博客批量转化为网页并且跟主页形成总分结构。
下面是我用Hexo搭建个人网站的过程和网站配置：

首先，一切以[Hexo官网](https://hexo.io/zh-cn/index.html)为准。

### 准备工作
- 正常使用Github的电脑。
- 开了Github Pages的Repository。
如果不了解Github可以看[Github Pages(一)：一个最基础的个人网站](https://hans2936.github.io/2017/10/21/PageBasic/)。
<!--more-->

### 配置Hexo文件夹
在mac上安装 Node/Npm
```
brew update
brew install node
```
（windows/linux用户参见[此官方教程](https://hexo.io/zh-cn/docs/)）

安装 Hexo
```
npm install hexo -g
hexo init blog
cd blog
npm install
hexo s
```
这时博客就在本地生成了。访问`http://localhost:4000` 可以看效果。
可以说Hexo是很强大的，默认主题网站结构合理，适配手机，搜索栏（google）也有了。只需要优化（改一下失效的链接，添加评论，RSS等模块）就行了。

（2021年6月更新：node的版本不能太高，不然会出现localhost可以看，但生成网页为空的问题）

### 基本操作
- `hexo g` 生成/public 文件夹，里面是网站
- `hexo d` 把这个网站文件夹推送到服务器
- `hexo clean` 删除网站文件夹
- `hexo s` 本地查看效果

### 配置文件
配置文件是两个，第一个是根目录的 `_config.yml`。重要配置有
- language: zh-CN 是中文，不写是英文
- url: https://hans2936.github.io （网站地址）
- root: / (根目录在哪里，github就写斜杠，有些服务器会多一层路径)
- skip_render: README.md 这样可以在 /source 里面放一个 README.md，推送的时候自动传到 Github 上面
- theme: landscape 这里可以换主题

推送设置 (GitHub)
```
deploy:
  type: git
  repository: https://github.com/hans2936/hans2936.github.io.git
  branch: master
```
如果网站在服务器上，则可以用 rsync
```
deploy:
  type: rsync
  host: 服务器名
  user: 用户名
  root: 放网站的文件夹
  port: 22
```
第二个配置文件，是主题的配置文件 `themes/landscape/_config.yml`，主要有导航栏(`menu`)，侧边栏(`widgets`)，网站图标(`favicon`)等。

### 写新文章
```
hexo new "article name"
```
这条命令会在`source/_posts`产生新文件，然后改改文件名，在进入编辑MarkDown就行了。

每篇文章最上面是配置区，能用到的主要是
- title: 题目
- date: 日期 （会影响在主页的顺序）
- tags: 标签
- categories: 分类
- updated: 修改日期

配置区下面就随便写了。值得一提的是，文章正文是支持html语言的，这样特殊字体和元素就可以直接加进去。

### 创建页面
```
hexo new page about
```
会在 `source/about` 里面产生新文件，跟文章是一样编辑的。
然后再从主题配置的导航栏里面加上这一页。
```
menu:
  ...
  关于: /about
```
### 引用图片
执行 `npm install hexo-asset-image --save`
然后，主配置文件 `_config.yml` 设置为
```
post_asset_folder: true
```
这个时候创建新文章就会产生一个同名文件夹，把图片放入即可。
然后在文章正文这样引用放进去的图片。 
```
![图片描述](文章名/图片名)
```

### 引用视频文件
对某个链接中的视频文件，可以执行 `npm install hexo-tag-dplayer`
这是一个播放器插件，使用时在文章中加入：
```
{% dplayer "url="  "pic=" "loop=yes" "theme=#FADFA3" "autoplay=false" "token=tokendemo" %}
```
其中 `url` 是视频地址，`pic` 是缩略图地址。

这个方法是引入视频的最佳方法，但是需要一个地方来存放这些文件（url）。
Github只有1G太少了，可以考虑使用[阿里云oss存储](https://oss.console.aliyun.com/)，淘宝账户就能开，还很便宜。

### 引入第三方视频
对视频网站上的视频，可在Markdown文件中明码使用html语言 `iframe`，比如
```
<div class="selfadapting-video">
<iframe src="" scrolling="no" border="0" frameborder="no" framespacing="0" allowfullscreen="true"> </iframe>
</div>
```
其中`class="selfadapting-video"` 是一个自适应长宽比例的容器，具体看[这篇博客](https://angeltime.cc/archives/685.html)。

### 创建侧边栏
比如在 `themes/landscape/layout/_widget/` 创建一个 about.ejs
```
<% if (site.tags.length){ %>
  <div class="widget-wrap">
    <h3 class="widget-title">About</h3>
    <div class="widget">
      E-mail: xyz@123.com
    </div>
  </div>
<% } %>
```
然后在主题配置的侧边栏中加上：
```
widgets:
- ...
- about
```

### RSS 订阅
执行 `npm install hexo-generator-feed` 下载rss功能包，
然后在主配置文件 `_config.yml` 里面加上
```
plugin:
- hexo-generator-feed
feed:
type: atom
path: atom.xml
limit: 20
```
然后在主题配置文件里加上 `rss: /atom.xml`。

网上的教程都是这样的，但是当我实际加入的时候，发现rss软件里面的图片都是空的。
后来我找到了这个[Fork of hexo-generator-feed](https://github.com/atika/hexo-generator-feed)，用它来替换官方的包，然后去掉包里面 `atom.xml` 里面的 `cdata`就可以了。
这个包还剩下一个bug是当程序框里的某一行code过长(出现左右拖动的功能后)，rss显示的code字体过大，但是不涉及编程的文章就没有问题了。

### 站点地图
类似上一条，执行 `npm install hexo-generator-sitemap --save`
主配置文件添加：
```
# Sitemap 
sitemap:
    path: sitemap.xml
```
然后提交给 Google Search Console 就行了。

### 高级修改 (评论，分享)
Hexo的网页其实是被拆开成很多零件的，主要在`themes/landscape/layout/_partial/`，
当在网页中加入一个新的模块时，比如对评论系统Gitment来说（关于Gitment见[此教程](https://imsun.net/posts/gitment-introduction/)），首先要打开 `head.ejs` 引用js, css文件（需放入`themes/landscape/source`）

（2019年12月更新：忽然发现Gitment不能用了，通过查看Gitment的Github项目[Gitment issue 188](https://github.com/imsun/gitment/issues/188)解决了问题。又发现出现了类似的工具Gitalk。）

 (2020年4月更新：发现上次解决方案重新push的时候还是有bug，改用 [Gitalk](https://github.com/gitalk/gitalk/blob/master/readme-cn.md) 了，另外在外网，[Disqus](https://disqus.com)是可以用的。)

```
<link rel="stylesheet" href="/css/gitment.css">
<script src="/js/gitment.browser.js"></script>
```
然后在 `article.ejs` 里面加上 Gitment 的code
```
<% if (!index && post.comments){ %>
  <section id="comments">
  <div id="gitment"></div>
      <script>
      const gitment = new Gitment({
        id: '<%= post.date %>',
        owner: 'username',
        repo: 'username.github.io',
        oauth: {
        client_id: '---yourID---',
        client_secret: '---yourKey---',
        },
      })
      gitment.render('gitment')
      </script>
  </section>
<% } %>
```
就实现这个第三方功能了。注意 `id: '<%= post.date %>'` 这句话是为了修复网页路径过长不能显示评论的bug (在手机app内分享网页很容易出现长后缀)。

主题自带的功能也可以改，比如说分享功能可以在`themes/landscape/source/js/script.js`加一句，
```
'<a href="http://service.weibo.com/share/share.php?&title=' + encodedUrl + '" class="article-share-sina" target="_blank" title="微博"></a>'
```
然后找到`themes/landscape/source/css/_partial/article.styl` 比照着定义一个 `.article-share-sina` 就可以了。

像这种高级修改，对有一定网页知识的人来说有无限可能，自己做一个主题都是可以的。

## 本站的Hexo设置
- 默认主题 Landscape。
- 抬头图片位置`banner.jpg`：我家的猫。
- 评论系统为Gitment，需要引用`gitment.css`, `gitment.broser.js`， 并编辑 `article.ejs` 和 `head.ejs`。
- 分享按钮为自带按钮，包括Sina（改 `script.js` 和 `article.styl`）。
- jquery.min.js 路径从google改为 https://cdn.bootcss.com/jquery/3.3.1/jquery.min.js (改 `after-footer.ejs`)。
- 从 NexT 主题移植而来的本地搜索引擎，见[Github Pages(四)：Hexo进阶-NexT本地搜索引擎的移植](https://hans2936.github.io/2018/06/15/HexoSearch/)。
