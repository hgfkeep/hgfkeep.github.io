+++
draft = false
date = 2019-06-26T19:54:00+08:00
title = "hugo静态博客"
description = "使用hugo创建美丽的静态博客，支持代码高亮、图标、icon fonts，并能持续集成，自动部署更新"
tags = ["blog", "Hugo", "asciidoctor"]
categories = ["Hugo"]
externalLink = ""
series = ["blog"]

+++



# 使用hugo创建博客

1. 使用 hugo + asciidoc(tor)实现静态博客、代码高亮、图标等。
2. hugo site构建使用镜像完成。
3. github部署静态博客。
4. travis-ci 持续集成和部署。

## 博客样式
### 思路

hugo [external helpers](https://gohugo.io/content-management/formats/#additional-formats-through-external-helpers) 提供外部格式化支持。通过官网给出的文档和[hugo的源代码](https://github.com/gohugoio/hugo/blob/77c60a3440806067109347d04eb5368b65ea0fe8/helpers/general.go#L65)，知道hugo是根据文件后缀使用external helpers取格式化文件。

文件起始的头部描述信息和markdnwon一样。

> 使用external helpers时，必须安装对应的工具，比如hugo+asciidoc(tor)就需要安装asciidoc(tor)。

所以需要使用hugo + asciidoc(tor)编译文档时，只需要将文件后缀改成`asciidoc`,`adoc`或者`ad`，hugo在构建site的时候，会自动的调用外部工具编译文件。

### external helpers支持情况

各类型文件后缀主持如下：

1. markdown的文件：
    1. md
    2. markdown
    3. markdown
2. asciidoc文件：
    1. asciidoc
    2. adoc
    3. ad
3. mmark文件：
    1. mmark
4. rst文件：
    1. rdt
5. html文件：
    1. htm
    2. html

### 配置样式

编辑hugo 根目录下面的`config.toml`，添加`Custom CSS`。

> 不同的hugo主题，`Custom CSS`配置的相对路径也不一样。有的是相对的`statis/`，有的是相对的`static/css`。

custom css包含两个部分：

1. 代码高亮。
2. 警告、icon fonts和其他样式。

我的`Custom CSS`配置：`custom_css = ["css/code.css","css/custom.css"]`。

### 代码高亮

我将可选的css样式保存到一个文件夹，然后将自己选择的css复制到`static/css/coder.css`。

我的仓库中有一些[css样式](https://github.com/hgfkeep/hgfkeep.github.io/tree/source/static/asscii-css)，可供大家参考。

### 警告 icon fonts样式

可以直接使用[asciidoc(tor)提供的样式](https://github.com/asciidoctor/asciidoctor-stylesheet-factory)

可以使用docker镜像编译




## docker打包hugo环境

镜像构建Dockerfile如下：

```dockerfile
FROM alpine

ARG HUGO_VERSION=0.55.6

ENV DOC_DIR=/hugo \
    PORT=1313

ADD https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/hugo_${HUGO_VERSION}_Linux-64bit.tar.gz /tmp/hugo.tar.gz

RUN mkdir ${DOC_DIR} && \
    tar -xzf /tmp/hugo.tar.gz -C /usr/local/bin/ && \
    rm /tmp/hugo.tar.gz && \
    apk add asciidoc --no-cache

WORKDIR ${DOC_DIR}

VOLUME ${DOC_DIR}
EXPOSE ${PORT}

CMD ["hugo", "server", "--bind" , "0.0.0.0"]
```

> 此docker镜像95MB。

构建本地hugo site：`docker run --rm -p 1313:1313 -v $PWD:/hugo-project rgielen/hugo-ubuntu:latest hugo server --bind 0.0.0.0`

构建hugo草稿：`docker run --rm -p 1313:1313 -v $PWD:/hugo-project rgielen/hugo-ubuntu:latest hugo server --buildDrafts --bind 0.0.0.0`

## 静态博客部署到github

主要流程：

1. github上创建项目，用来管理和部署静态博客，项目名为：`<username>.github.io`。
2. 分支管理
    1. hugo构建的静态页面为第一步创建项目的master分支；
    2. 静态博客的源代码为第一步创建项目的source分支，不包括public文件夹。


## 博客持续集成



## 参考和资源

### 代码高亮css

* [css样式参考](https://github.com/hgfkeep/hgfkeep.github.io/tree/source/static/asscii-css)

### asciidoc(tor) 书写手册

* [asciidoctor官方样式](http://themes.asciidoctor.org/preview/)