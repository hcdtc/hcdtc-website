+++
title = "Hugo框架指引"
description = ""
weight = 30
icon = ""
home = true
+++

# Hugo 框架指引
---

Hugo是由Go语言实现的静态网站生成器。简单、易用、高效、易扩展、快速部署。

## Quick start 

Hugo的安装使用便捷简单，参考官网[Hugo quick start](https://www.gohugo.org/)。以下主要介绍其基本概念和本项目开发流程。

## 目录结构

``` hugo
    ├── archetypes
    ├── config.toml
    ├── content
    │   └── docs
    │       └── xx-doc
    │           └── x-doc
    ├── data
    ├── layouts
    ├── static
    └── themes
```

**archetypes**

在通过hugo new xxx 创建内容页面的时候，默认情况下hugo会创建date、title等front matter，可以通过在archetypes目录下创建文件，设置自定义的front matter。

**config.toml**

所有的hugo站点都有一个全局配置文件，用来配置整个站点的信息，hugo默认提供了跟多配置指令。

**content**

站点下所有的内容页面，也就是我们创建的md文件都在这个content目录下面。

**data**

data目录用来存储网站用到一些配置、数据文件。文件类型可以是yaml|toml|json等格式。

**layouts**

存放用来渲染content目录下面内容的模版文件，模版.html格式结尾，layouts可以同时存储在项目目录和themes/<THEME>/layouts目录下。

**static**

用来存储图片、css、js等静态资源文件。

**themes**

用来存储主题，主题可以方便的帮助我们快速建立站点，也可以方便的切换网站的风格样式。

**public**

hugo编译后生成网站的所有文件都存储在这里面，把这个目录放到任意web服务器就可以发布网站成功。

## Hugo 常用命令

- hugo init path：在path路径下初始化一个hugo博客目录
- hugo server --buildDrafts：hugo server如果没有带参数，默认在当前目录寻找config.toml，如果没有config.toml，命令中就需要带很多参数。--buildDrafts意为“编译草稿（draft: true的Markdown文件）”
- hugo --buildDrafts：生成可发布的静态html。如果没有指定路径，默认在当前路径下新建public。如果在config.toml中指定了publicdir参数，则默认到该路径下，注意路径的斜杠问题。

## 页面配置

**front matter** 

用来配置文章的标题、时间、链接、分类等元信息，提供给模板调用

```
+++
title = "post title"
description = "description."
date = "2018-08-20"
tags = [ "tag1", "tag2", "tag3"]
categories = ["cat1","cat2"]
weight = 20
+++
```

## 本文档项目开发流程及注意项

* 本地启动命令： hugo server | 浏览器里打开： http://localhost:1313
* 目录结构及目录名规则： docs 目录下创建 xx(number)-xxx， _index.md 中 weight 为目录权重， 从 1 开始，越小权重越大。
* _index.md 内关联下级菜单
    * `{{< `docdir` >}}` 功能模板 | 此代码为短代码（shortcodes）相当于一些自定义模版，通过在md文档中引用，生成代码片段，类似于一些js框架中的组件。
    *  `- [**xxx**](../xxx/xxx) 描述性文字。` 非功能模板关联菜单
* 图片引用
    * 在config.toml中配置baseurl | 作用是无论在本地还是发布出去，图片的路径都是相对于baseurl
    * 在Markdown图片路径中，以static为根目录写全路径。
     
        `示例：假设测试baseurl为http://localhost:1313/，图片位置：/static/img/hugo/123.png，那么在Markdown中写作![图片说明](/img/hugo/123.png)`
