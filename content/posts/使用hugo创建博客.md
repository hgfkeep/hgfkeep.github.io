---
title: "如何使用typora编写Hugo draft"
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
hideHeaderAndFooter: true

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

# 使用hugo创建博客

1. 使用 hugo + asciidoc(tor)实现静态博客、代码高亮、图标等。
2. hugo site构建使用镜像完成。
3. github部署静态博客。
4. travis-ci 持续集成和部署。

## 博客配置

主要配置hugo和asciidoctor的触发协同构建。

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

我将可选的代码高亮css样式保存到一个文件夹，然后将自己选择的代码高亮css复制到`static/css/coder.css`。

我的仓库中有一些[css样式](https://github.com/hgfkeep/hgfkeep.github.io/tree/source/static/asscii-css)，可供大家参考。

### 警告 icon fonts样式

可以直接使用[asciidoc(tor)提供的样式](https://github.com/asciidoctor/asciidoctor-stylesheet-factory)

将一下内容存到`static/css/custom.css`。

```css
table{border-collapse:collapse;border-spacing:0}
.admonitionblock>table{border-collapse:separate;border:0;background:none;width:100%}
.admonitionblock>table td.icon{text-align:center;width:80px}
.admonitionblock>table td.icon img{max-width:none}
.admonitionblock>table td.icon .title{font-weight:bold;font-family:"Open Sans","DejaVu Sans",sans-serif;text-transform:uppercase}
.admonitionblock>table td.content{padding-left:1.125em;padding-right:1.25em;border-left:1px solid #ddddd8;color:rgba(0,0,0,.6)}
.admonitionblock>table td.content>:last-child>:last-child{margin-bottom:0}
.admonitionblock td.icon [class^="fa icon-"]{font-size:2.5em;text-shadow:1px 1px 2px rgba(0,0,0,.5);cursor:default}
.admonitionblock td.icon .icon-note::before{content:"\f05a";color:#19407c}
.admonitionblock td.icon .icon-tip::before{content:"\f0eb";text-shadow:1px 1px 2px rgba(155,155,0,.8);color:#111}
.admonitionblock td.icon .icon-warning::before{content:"\f071";color:#bf6900}
.admonitionblock td.icon .icon-caution::before{content:"\f06d";color:#bf3400}
.admonitionblock td.icon .icon-important::before{content:"\f06a";color:#bf0000}
.conum[data-value]{display:inline-block;color:#fff!important;background-color:rgba(100,100,0,.8);-webkit-border-radius:100px;border-radius:100px;text-align:center;font-size:.75em;width:1.67em;height:1.67em;line-height:1.67em;font-family:"Open Sans","DejaVu Sans",sans-serif;font-style:normal;font-weight:bold}

.conum[data-value] *{color:#fff!important}
.conum[data-value]+b{display:none}
.conum[data-value]::after{content:attr(data-value)}
pre .conum[data-value]{position:relative;top:-.125em}
b.conum *{color:inherit!important}
.conum:not([data-value]):empty{display:none}
```

## asciidoctor作图和预览

> 此功能目前受限。**hugo不支持asciidoctor的插件。**

asciidoctor支持plantUML，ditaa等作图工具做图，在本地visual studio 中编辑，需要安装插件：joaompinto.asciidoctor-vscode

然后 在adoc文件中添加文件属性`:plantuml-server-url: "http://plantuml.com/plantuml"`

由于plantuml是http协议的，默认visual studio认为不安全，需要配置 **asciidoc preview security**，
![15616072903063](/img/使用hugo创建博客/15616072903063-1627109.jpg)

允许insecure content。
![15616073401160](/img/使用hugo创建博客/15616073401160.jpg)


## docker打包hugo构建环境

### hugo+asciidoctor镜像

执行`docker pull hgfdodo/hugo-asciidoctor`获取此镜像。

```dockerfile
FROM alpine

ARG HUGO_VERSION=0.55.6

ENV DOC_DIR=/hugo \
    PORT=1313

ADD https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/hugo_${HUGO_VERSION}_Linux-64bit.tar.gz /tmp/hugo.tar.gz

RUN mkdir ${DOC_DIR} && \
    tar -xzf /tmp/hugo.tar.gz -C /usr/local/bin/ && \
    rm /tmp/hugo.tar.gz && \
    apk add asciidoctor ruby ruby-dev build-base bison  --no-cache && \
    gem install --no-document asciidoctor asciidoctor-revealjs \
         rouge asciidoctor-confluence asciidoctor-diagram coderay pygments.rb

WORKDIR ${DOC_DIR}

VOLUME ${DOC_DIR}
EXPOSE ${PORT}

CMD ["hugo", "server", "--bind" , "0.0.0.0"]
```

> 此镜像包括了所有hugo已有的功能，但[hugo exteranl helper 暂时不支持asciidoctor 插件](https://github.com/gohugoio/hugo/issues/5688)。所以无法使用asciidoctor-diagram作图。


### 支持asciidoctor插件镜像

```dockerfile
FROM alpine

ARG HUGO_VERSION=0.55.6

ENV DOC_DIR=/hugo \
    PORT=1313

ADD https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/hugo_${HUGO_VERSION}_Linux-64bit.tar.gz /tmp/hugo.tar.gz

RUN mkdir ${DOC_DIR} && \
    tar -xzf /tmp/hugo.tar.gz -C /usr/local/bin/ && \
    rm /tmp/hugo.tar.gz && \
    apk add openjdk8-jre asciidoctor ruby ruby-dev build-base bison  --no-cache && \
    gem install --no-document asciidoctor asciidoctor-revealjs \
         rouge asciidoctor-confluence asciidoctor-diagram coderay pygments.rb

WORKDIR ${DOC_DIR}

VOLUME ${DOC_DIR}
EXPOSE ${PORT}

CMD ["hugo", "server", "--bind" , "0.0.0.0"]
```

>此镜像字体和plantuml运行环境比较占存储，**仅能手动构建plantuml的asciidoc**。
>
>由于asciidoctor-diagram可能使用plantuml，plantuml需要依赖java运行环境，所以添加了openjdk8-jre。还可能[有字体的问题，需要添加`ttf-dejavu](https://blog.zjyl1994.com/post/alpine-fontconfig/)`， 还需要安装`graphviz`，用来做徒刑转换

构建本地hugo site：`docker run --rm -p 1313:1313 -v $PWD:/hugo hgfdodo/hugo-asciidoc hugo server --bind 0.0.0.0`

构建hugo草稿：`docker run --rm -p 1313:1313 -v $PWD:/hugo hgfdodo/hugo-asciidoc hugo server --buildDrafts --bind 0.0.0.0`

## 静态博客部署到github

主要流程：

1. github上创建项目，用来管理和部署静态博客，项目名为：`<username>.github.io`。
2. 分支管理
    1. hugo构建的静态页面为第一步创建项目的**master**分支；
    2. 静态博客的源代码为第一步创建项目的**source**分支，不包括`public`文件夹。

## 博客持续集成

### hugo根目录添加.travis.yml

使用docker持续集成， 核心思路：

1. 使用docker service；
2. 拉镜像
3. 构建博客
4. 将构建结果推送到仓库master分支

```yaml
dist: xenial
language: python
python:
  - "3.7"
services:
  - docker

before_install:
  - docker pull hgfdodo/hugo-asciidoc:alpine

# script - run the build script
script:
    - docker run --rm -v $PWD:/hugo hgfdodo/hugo-asciidoctor hugo
    # 如果有自己的域名，可以添加CNAME纪录
    # - echo "$CNAME_URL" > public/CNAME

deploy:
  provider: pages
  skip-cleanup: true
  github-token: $GITHUB_TOKEN  # Set in travis-ci.org dashboard, marked secure
  email: $GITHUB_EMAIL
  name: $GITHUB_NAME
  verbose: true
  keep-history: true
  local-dir: public
  target_branch: master  # branch contains blog content
  on:
    branch: source # branch contains Hugo generator code
```

### 配置travis

用github账号登陆[travis官网](https://travis-ci.org)，配置`.travis.yml`需要使用的环境变量（如下图），当代码提交后，便会自动触发构建过程。

![15615968253261](/img/使用hugo创建博客/15615968253261.jpg)


构建的历史如下图：
![15615969999516](/img/使用hugo创建博客/15615969999516.jpg)


## 参考和资源

### 代码高亮css

* [css样式参考](https://github.com/hgfkeep/hgfkeep.github.io/tree/source/static/asscii-css)

### asciidoc(tor) 书写手册

* [asciidoctor官方样式](http://themes.asciidoctor.org/preview/)