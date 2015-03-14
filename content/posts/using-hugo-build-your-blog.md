+++
author = "Ryan Zhao"
categories = []
date = "2015-03-14T17:27:08+08:00"
description = ""
keywords = []
mathjax = false
tags = []
title = "使用Hugo构建个人博客"
draft = true

+++

## 简介

### 什么是Hugo

[Hugo](http://http://gohugo.io/)是一款静态站生成器，跟Wordpress不同，Hugo被用来从内容中生成静态的HTML，而不是页面被访问时再生成HTML。因此，Hugo并没有对运行时环境的依赖，只需把Hugo生成的静态资源上传到站点上即可。

虽然是静态站生成器，但是在书写博客上，Hugo与Wordpress的功能不相上下，甚至体验更好。

### 使用Hugo

Hugo是使用Go语言开发，支持Mac，Linux，Windows平台。相较于其他静态站生成器，仅仅需要下载一个预编译的可执行包就可以使用了，不需要其他依赖(在windows上使用jekyll有点麻烦的)。

在此页面下载[Hugo](https://github.com/spf13/hugo/releases)最新版本，然后将其添加到PATH中即可。

然后使用hugo命令，创建一个站点：

```bash
hugo new site /path/to/site
```

进入到站点目录后，可以看到如下的目录结构：

```bash
  ▸ archetypes/
  ▸ content/
  ▸ layouts/
  ▸ static/
    config.toml
```

上述目录中

archetypes
: 存放不同类型内容的模板，可以使用模板创建内容。

content
: 存放你的站点内容

layouts
: 存放HTML模板，用于从内容中生成HTML。

static
: 用于存放站点的静态资源，如css,js文件等。

config.toml
: 全局配置文件，使用TOML格式，不熟悉的可以参考[这篇文章](http://mlworks.cn/posts/introduction-to-toml/)。

下面，我们创建一些内容

```bash
hugo new about.md
```

```new```创建的内容在```content```目录中，可以查看到下面的内容

```bash
+++
date = "2015-01-08T08:36:54-07:00"
draft = true
title = "about"

+++
```

文章的顶部用+++围起来的内容是TOML格式配置的内容属性(一些是Hugo需要用到的，你可以随意添加自己的属性)。文章的内容是使用Markdown格式书写，不熟悉的可以参考[这篇文章](##TODO)。

安装主题

Hugo



