---
title: hexo+github 常见问题
date: 2018-06-15 15:57:16
tags:
 - hexo
 - github

toc: ture
---

搭建和添加一些功能时所踩过的坑~

## 内容概要
* 常见语法问题
* 端口号占用
* 插入图片没有预览
* 利用GitHub实现相册url

<!--more-->

## 常见语法问题
[markdown文档](http://www.markdown.cn/#overview)
markdown对于语法格式要求很严格，缩进和空格要多注意

## 端口号占用
在命令行中输入 netstat -ano 即可查看端口占用情况，输入 taskkill /f /t /xxx(占用端口的进程) 可以结束进程。[详细内容](https://jingyan.baidu.com/article/3c48dd34491d47e10be358b8.html)
在git bash中更改server端口：

```bash
$ hexo s -p xxxx  #重设端口为xxxx
```

## 插入图片没有预览
在2.x版本中在文章中插入图片没有预览

```bash
[xxx](xxx/xxx.jpg)
```

在3.x版本可以添加预览

```bash
{% asset_img xxx.PNG xxx %}
```

## 利用GitHub实现相册url
相册在render函数中的保存图片的地址需写成：

```bash
$ var minSrc = 'https://raw.githubusercontent.com/自己的用户名/保存图片的仓库/master/xxxphotos/' + data.link[i];
```










