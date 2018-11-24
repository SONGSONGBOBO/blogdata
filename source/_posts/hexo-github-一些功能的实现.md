---
title: hexo+github 一些功能的实现
date: 2018-06-16 9:58:40
tags:
 - hexo
 - github
 
toc: ture
---

### 内容概要

* 更换主题
* blog主页文章预览大小
* 评论
* 文章字数和阅读时长
* 评论
* 相册

<!--more-->

### 更换主题
比较流行的、功能也比较丰富的主题;
1.[yilia主题](https://github.com/litten/hexo-theme-yilia.git) [实现效果](http://litten.me/)
2.[next主题](https://github.com/iissnan/hexo-theme-next) [实现效果](https://notes.iissnan.com/) [next文档](http://theme-next.iissnan.com/)
3.[yelee主题](https://github.com/MOxFIVE/hexo-theme-yelee.git) [实现效果](http://moxfive.xyz/) [yelee文档](http://moxfive.coding.me/yelee/)

在创建的blog根目录中打开git bash输入：

```bash
$ git clone https://github.com/xxxx/hexo-theme-xxx.git themes/xxx
```

在根目录中的config.xml中将theme属性改为所要使用的主题名。

## blog主页文章预览大小
在需要隔断的位置插入：

```bash
<!--more-->
```

## 评论
网上查了很多种，有一些不让用了，有一些广告比较多，还有的需要翻墙不方便，这里推荐两种

### 来必力
来必力简单，没有广告，支持多种账号登录

1.登陆注册来必力
2.在右上角可以进入管理界面，可以进行各种配置

{% asset_img laibili.jpg test %}

3.在 themes/主题名/layout/_partial/xxx文件夹创建livere.ejs文件，将管理界面中的代码管理所提供的代码拷贝于此。
4.在theme/主题名/_config.yml中添加配置信息

```bash
livere: 
   on: ture
   livere_uid: your uid

```

5.在themes/xxxx/layout/_partial/article.ejs中，配置评论

```bash
<% if (theme.livere.on){ %>
  <%- partial('post/livere') %>
<% } %>
```

### gitment
[gitment文档](https://github.com/imsun/gitment#customize)
[gitment demo](https://imsun.github.io/gitment/)

1.在github创建一个准备存放评论的仓库
2.注册一个新的 [OAuth Application](https://github.com/settings/applications/new)。callback URL填入评论页面对应的域名，其余随便写一些。你会得到一个 client ID 和一个 client secret，这个将被用于之后的用户登录。
3.在themes中的config进行配置

```bash
#Gitment
gitment_owner: ''      #你的 GitHub ID
gitment_repo: ''          #存储评论的 repo
gitment_oauth:
  client_id: ''           #client ID
  client_secret: ''     #client secret
```

4.在themes/xxx/_partial/post/下创建gitment.ejs

```bash
<div id="gitment-ctn"></div> 
<link rel="stylesheet" href="//imsun.github.io/gitment/style/default.css">
<script src="//imsun.github.io/gitment/dist/gitment.browser.js"></script>
<script>
var gitment = new Gitment({
  id: '<%=page.date%>',
  owner: '<%=theme.gitment_owner%>',
  repo: '<%=theme.gitment_repo%>',
  oauth: {
    client_id: '<%=theme.gitment_oauth.client_id%>',
    client_secret: '<%=theme.gitment_oauth.client_secret%>',
  },
})
gitment.render('gitment-ctn')
</script>
```

5.在article.ejs中添加

```bash
<% if (theme.gitment_owner && theme.gitment_repo &&theme.gitment_oauth && theme.gitment_oauth.client_id && theme.gitment_oauth.client_secret){ %>
<%- partial('post/gitment', {
    key: post.slug,
    title: post.title,
    url: config.url+url_for(post.path)
  }) %>
<% } %>
```

## 相册
### 相关工具
PIL（Python Imaging Library）是Python常用的图像处理库，而Pillow是PIL的一个友好Fork，提供了了广泛的文件格式支持，强大的图像处理能力，主要包括图像储存、图像显示、格式转换以及基本的图像处理操作等。
1.安装python环境
2.新版的python都有pip，直接安装pil

{% asset_img pillow.png test02 %}

### 创建点击链接

```bash
$ hexo new page photos
```

立即在source下生成photos/index.md文件，然后在主题站点的配置文件theme/yelee/_config.yml中添加点击的文案和跳转到位置。

{% asset_img menu.PNG test03 %}

### 处理图片
对于相册浏览，要有预览时的缩略图和点击后的原图，所以在blog根目录中创建两个文件夹，一个放小图，一个放原图
1.首先可以参考他人[blog备份](https://github.com/SONGSONGBOBO/blogdata/tree/master/source/photos)
2.修改ins.js文件render()函数，修改两个src即可

{% asset_img render.PNG test04 %}

3.对于图片的压缩、生成json数据和图片的上传使用python脚本解决，[脚本地址](https://github.com/SONGSONGBOBO/blogdata),代码在tool.py中查看
4.blog部署后，效果可以看[我的相册](http://imsongbo.com/photos/)。

## 文章字数和阅读时长
[hexo-wordcount使用文档](https://www.npmjs.com/package/hexo-wordcount)
1.安装hexo-wordcount

```bash
$ npm i --save hexo-wordcount
```

2.themes下的config.xml中配置是否需要功能的flag

```bash
#字数统计和阅读时长
word:
  word_count: ture
```

3.在themes/xxxx/layout/_partial/post/word.ejs下创建word.ejs文件

```bash
<div style="margin-top:10px;">
    <span class="post-time">
      <span class="post-meta-item-icon">
        <i class="fa fa-keyboard-o"></i>
        <span class="post-meta-item-text">  字数统计: </span>
        <span class="post-count"><%= wordcount(post.content) %>字</span>
      </span>
    </span>

    <span class="post-time">
      &nbsp; | &nbsp;
      <span class="post-meta-item-icon">
        <i class="fa fa-hourglass-half"></i>
        <span class="post-meta-item-text">  阅读时长: </span>
        <span class="post-count"><%= min2read(post.content) %>分</span>
      </span>
    </span>
</div>
```

4.在 themes/xxx/layout/_partial/article.ejs的header中

```bash
<% if (theme.word.word_count && !post.no_word_count){ %>
  <%- partial('post/word') %>
<% } %>
```

5.发布后成功如图所示。

{% asset_img wordcount.PNG test05 %}



