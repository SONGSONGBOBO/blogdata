---
title: '[deepin使用过程中一些问题和优化]'
date: 2018-11-22 12:10:30
tags:
- deepin
- linux
toc: ture
---

### 总览

* 
* 



deepin是基于debian的一款linux发行版，目前使用了半年左右，系统稳定，软件和桌面优化环境很好，问题很少，我会把现在和以前所碰到的一些问题整理出来

<!--more-->

### Picked up _JAVA_OPTIONS: -Dawt.useSystemAAFontSettings=gasp

设置环境变量时出现 Picked up _JAVA_OPTIONS: -Dawt.useSystemAAFontSettings=gasp

**主要原因：**系统原有的OpenJDK设置干扰了手动安装的JDK。干扰的文件是：/etc/profile.d/java-awt-font-gasp.sh

**解决方法：**在系统加载的时候就直接禁掉这个环境变量

```bash
unset _JAVA_OPTIONS #在jdk环境变量之前添加
```

