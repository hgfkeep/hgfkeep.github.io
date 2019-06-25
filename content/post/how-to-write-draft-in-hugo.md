---
title: "How to Write Draft in Hugo"
date: 2019-06-25T16:33:26+08:00
lastmod: 2019-06-25T16:33:26+08:00
draft: false
keywords: [markdown]
description: "合理有效的使用typora编辑器编写hugo文章"
tags: [markdown]
categories: [markdown, 工具]
author: "heguangfu"

# You can also close(false) or open(true) something for this content.
# P.S. comment can only be closed
comment: true
toc: true
autoCollapseToc: true
postMetaInFooter: true
hiddenFromHomePage: false
# You can also define another contentCopyright. e.g. contentCopyright: "This is another copyright."
contentCopyright: <img src="https://licensebuttons.net/l/by-nc-sa/3.0/88x31.png"><br/>感谢阅读，如果有问题请您留言，我会及时改正<br/> 本博客所有原创文章版权归hgf所有，转载请注明出处hgfdodo.win/blog
reward: false
mathjax: true
mathjaxEnableSingleDollar: true
mathjaxEnableAutoNumber: true

# You unlisted posts you might want not want the header or footer to show
hideHeaderAndFooter: false

# You can enable or disable out-of-date content warning for individual post.
# Comment this out to use the global config.
#enableOutdatedInfoWarning: false

flowchartDiagrams:
  enable: false
  options: ""

sequenceDiagrams: 
  enable: false
  options: ""

typora-root-url: ../../static
---





# 如何使用typora编写Hugo draft

本文主要讲解如何使用typora编写hugo 文章，达到本地编辑实时预览，和真正部署的结果一致。

<!--more-->

## Hugo 目录结构


```
├── archetypes
├── config.toml
├── content
├── data
├── i18n
├── images
├── layouts
├── public
├── resources
├── static
└── themes
```

默认的静态文件全部存储在根目录的`static`文件夹中， 图片文件也存在static文件夹中，例如存储路径为 `static/${filename}/setting.png`。那么访问路径为 `BASE_URL/img/${filename}/setting.png`。所以图片引用写法为`![设置](img/${filename}setting.png)`



但是这样会存在一个问题，**编辑时无法完整预览文件图片**。



## typora编辑时预览



1. 设置全局图片规则:**图片插入时自动复制**到自定目录`[hugo-site]/static/img/${filename}`

![setting](/img/how-to-write-draft-in-hugo/setting.png)





2. 配置图片根目录

  

  通过设置图片的根目录，这样图片在markdown文件中的引用地址就成相对于图片根路径的路径。
  ![image-20190625172634870](/img/how-to-write-draft-in-hugo/image-20190625172634870.png)