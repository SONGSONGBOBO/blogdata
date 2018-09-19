---
title: '[hexo+github building process]'
date: 2018-06-14 18:44:31
tags: 
- hexo 
- github
toc: ture
---
闲的没事自己试着搭了下博客，记录一下搭建过程，给像我这样的小白一点经验。
[Hexo官网](https://hexo.io/) 

### 步骤

·搭建之前
·个人域名
·github与git
·hexo
·hexo与GitHub关联
·绑定域名

<!--more-->

### 搭建之前

1.安装node.js和npm和git
2.GitHub账号
我的环境是:win10 node-v8.11.2-x64
命令行命令建议使用git bash here

### 个人域名

我是在阿里云注册的域名，按照官网步骤注册即可，一般.top比较便宜。

#### 解析域名

进入控制台可以在自己注册的域名右侧有解析选项，添加两个记录，记录名分别为A和CNAME。
{% asset_img yumingjiexi.PNG test %}
A的记录值为你的gihub的ip，可以ping 用户名.github.io获取一下。
CNAME的记录值为 用户名.github.io

### github与git

#### GitHub建立仓库

在GitHub中建立个人仓库，仓库名字为 用户名.github.io 这个用户名必须是你的GitHub的账号名。

#### git配置

廖雪峰老师的[git教程](https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000),安装成功后，右键git bash here 配置。

```bash
$ git config --global username "自己的github用户名"
$ git config --global mail "自己的github注册邮箱"
```

#### 配置ssh key

```bash
$ ssh-keygen -t rsa -C "你的GitHub注册邮箱" #生成ssh密钥文件
```

在user目录下找到.ssh文件,复制id_rsa.pub文件的内容，粘贴到GitHub中新建的ssh key中，title随便写一下。

{% asset_img ssh.PNG test %}

在命令行中验证是否安装成功。

```bash
$ ssh -T git@github.com #不需要修改
```

如果有提示yes/no，输入yes，出现Hi xxxxxx! You've successfully authenticated, but GitHub does not provide shell access.则配置成功。

### hexo

#### 安装hexo

建立一个文件夹存储代码

```bash
$ npm install -g hexo-cli  #npm安装hexo
```

进行测试：

```bash
$ hexo init blog #初始化
$ hexo n "test"  #创建测试文章
$ hexo g         #生成静态文件
$ hexo s         #开启本地服务器
```

在本地浏览器中访问localhost:4000即可访问。
加载不出来有可能端口被占用，可以通过

```bash
$ hexo s -p xxxx  #修改默认端口为xxxx
```

### hexo与GitHub关联

#### _config.xml 配置

打开根目录的配置文件_config.xml，在文件后添加

deploy: 
type: git
repo: GitHub仓库的完整路径.git
branch: master

仓库路径可以使用https或者ssh，哪个好使用哪个。

#### 安装git部署插件

```bash
$ npm install hexo-deployer-git --save  #安装git部署插件
```
随后可以进行部署测试：

```bash
$ hexo clean   #清理缓存文件（db.json）和生成的文件（public）
$ hexo g       #生成静态文件
$ hexo d       #部署网站
```

可以通过访问个人仓库路径（xxx.github.io）来进入你的网站。

### 绑定域名

在自己的GitHub→setting→GitHub Pages→Custom domain中，填入自己所解析的域名，save后即可通过自己的域名访问网站。
在网站的根目录中的source文件夹中创建一个文件(格式为所有文件，可以创建其他格式文件后删除文件后缀)，在文件中输入自己所创建的域名，可以选择是否带www，
·带www，必须使用www.xxx.com访问。
`不带www,可以使用www.xxx.com，也可以使用xxx.com访问。

```bash
$ hexo clean   #清理缓存文件（db.json）和生成的文件（public）
$ hexo g       #生成静态文件
$ hexo d       #部署网站
```

更新部署后，可以发现文件内容覆盖Custom domain。

