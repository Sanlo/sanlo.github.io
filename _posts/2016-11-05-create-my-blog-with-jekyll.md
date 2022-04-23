---
layout: post
title:  "Jekyll 搭建静态博客"
categories: jekyll
tags: jekyll RubyGems
---

* content
{:toc}

在github上用jekyll搭建博客。



## 搭建过程

在jekyll的官网上 [http://jekyllrb.com/](http://jekyllrb.com/) 其实已经说得比较明白了，我在这里还是简单的说一下吧。我用的是Windows系统。
主要环节有：安装Ruby，安装RubyGems，安装jekyll，安装代码高亮插件，安装node.js

### 安装Ruby

ruby官网下载安装：[https://www.ruby-lang.org/en/downloads/](https://www.ruby-lang.org/en/downloads/)

安装完成后配置环境变量

在命令提示符中，得到ruby版本号，如下图，即安装成功

[![4HHwIx.png](https://z3.ax1x.com/2021/10/02/4HHwIx.png)](https://imgtu.com/i/4HHwIx)

### 安装RubyGems

官网下载 [http://rubygems.org/pages/download](http://rubygems.org/pages/download) rubygems

cd到RubyGems 安装目录，执行安装

```
cd D:\下载\rubygems-3.3.12\rubygems-3.3.12	#切换文件目录
ruby setup.rb       #安装rubygems
ruby -v     #查看rubygems版本号
```

### 用RubyGems安装Jekyll

以上两个步骤操作完成后，在 CMD 窗口执行如下命令安装Jekyll：

```
 gem install jekyll   #安装jekyll
 jekyll -v    #查看jekyll版本号

```

至此jekyll就已经安装完毕了，后续就是个性化的自己设定了。

### 本地创建博客

#### 创建博客项目
```
jekyll new restlessManBlog   #新建博客
cd restlessManBlog     #切换目录
jekyll server      #启动项目
```
在浏览器访问当前项目：`http://localhost:4000/`

#### 添加MarkDown文档
在项目根目录下的`_posts`目录创建 markdown 文档。这里注意 md 文档命名要添加 “yyyy-mm-dd”的前缀。

例如：`2019-10-11-my-new-blog.md`

查看MarkDown的 [基础语法](https://markdown.com.cn/basic-syntax/)，可以获得MarkDown使用简介。

#### 部署代码到GitHub

```
git add .   #git 命令添加所有文件
git commit  #git 提交文件
git push    #git 推送代码到远程

```