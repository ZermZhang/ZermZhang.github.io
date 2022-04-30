---
title: github-pic-service
tags: ['github','图床','blog']
date: 2022-04-30
categories: ['git']
---
在日常写blog的过程中，我习惯使用markdown编写文章内容，这时候需要提供图片的地址以方便文章中可以显示图片，将图片放到本地当然方便，但是这样随着blog文章的增加，需要处理的源文件也越来越大。
而且这种情况下，文章内容不方便进行不同平台的迁移，迁移到其他平台的时候，为了防止文章中的图片无法显示，只能同时调整图片位置。

<!--more-->

为了解决上面的两个痛点，决定增加一个图床服务，方便在不同位置上都可以正常的访问文章中的图片情况而不受影响。

# 1. 常用的图床服务
1. github图床
2. SM.MS图床
3. 七牛云图床
4. 腾讯云
5. 阿里云

上面是一些常用的图床服务，各有千秋，我现在使用的图床服务是github的图床，github最大的优点就是所有数据都是存储在你的github仓库上，完全可控，但是国内github的访问情况就会导致在没有工具或者是网络不好的情况下，图片打开很慢或者完全打不开。
如果大家希望能有一个比较稳定的国内访问环境的话，可以考虑其他的图床，或者通过cdn进行加速访问。

# 2. 准备工作
## 2.1 准备一个github仓库
这一步比较简单，生成一个新的仓库，用来存储需要用到的图片，除了仓库权限需要是public的之外，其他的并没有什么要求，全部默认即可。
![register-repository](https://raw.githubusercontent.com/ZermZhang/pictures/main/20220430082824.png)

## 2.2 上传图片
> 图片上传方式可以使用传统的git同步逻辑，但是相对比较繁琐，所有推荐使用工具进行图片上传，常用的工具是PicGO

**通过PicGO上传图片**
1. 下载安装PicGo
先从[官网](https://molunerfinn.com/PicGo/)下载，然后安装。

2. 配置PicGo
配置PicGo的过程主要是指定github中需要使用到的仓库名，分支名和同步过程中需要使用到的token。
存储路径和自定义域名可以直接使用默认的值。
![config-for-PicGo](https://raw.githubusercontent.com/ZermZhang/pictures/main/20220430083319.png)

# 3. 图床使用
## 3.1 上传图片
PicGo支持方便的图片上传功能，可以直接通过拖拽将指定图片拖到上传区域中，即可上传。
截图的图片可以通过打开软件的方式，直接上传，很是便捷。
![upload-pic](https://raw.githubusercontent.com/ZermZhang/pictures/main/20220430083616.png)

## 3.2 获取图片链接
图片上传结束之后，就可以直接在相册中找到这张图片，通过点击的方式就可以将图片的链接复制到剪切板中。
之后，直接将图片的url粘贴到md格式的文章中就可以直接在文章里显示了。
![copy-pic-url](https://raw.githubusercontent.com/ZermZhang/pictures/main/20220430083651.png)

# 4. 存在的问题
1. 网络问题
因为这里的图床是通过github实现的，网络需要经过github，所有在网络不好的情况下，经常会出现网页图片无法显示的问题，需要的话可以切换到国内的图床工具上。