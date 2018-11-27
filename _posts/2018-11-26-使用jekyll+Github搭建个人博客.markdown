---
layout: post
title:  "2018-11-26-使用jekyll+Github搭建个人博客"
date:   2018-11-25 15:00:13 +0000
categories: 战神悟空
---



### 介绍
Jekyll 是一个简单的博客形态的静态站点生产机器。它有一个模版目录，其中包含原始文本格式的文档，通过一个转换器（如 Markdown）和我们的 Liquid 渲染器转化成一个完整的可发布的静态网站，你可以发布在任何你喜爱的服务器上。Jekyll 也可以运行在 GitHub Page 上，也就是说，你可以使用 GitHub 的服务来搭建你的项目页面、博客或者网站，而且是完全免费的。

### 安装Jekyll
安装Jekyll之前，要先确保在你的电脑上已经配置好Jekyll运行所需要的环境

Ruby
RubyGems
Python2.7(或2.7以上版本)
NodeJS

- 安装ruby

```bash
sudo apt-get install ruby2.0
```

ruby 1.9.2版本以后默认已安装RubyGems。所以在安装Ruby之后，查看gem版本

```bash
gem -v
```



- 安装python
  - 首先确保已安装gcc、g++，如没有则安装：

```bash
sudo apt-get install python 

sudo apt-get install build-essential 

sudo apt-get install gcc 

sudo apt-get install g++ 
```



- 安装NodeJs

```bash
sudo apt-get install nodejs
```

验证是否安装成功

```
nodejs -v
```

以上在所有系统下都是必须的，如安装出现问题请自行谷歌。windows下安装方法有问题可以查看[官方文档](https://link.jianshu.com/?t=http%3A%2F%2Fjekyllcn.com%2Fdocs%2Fwindows%2F%23installation)



- 安装Jekyll

```bash
gem install jekyll
```

