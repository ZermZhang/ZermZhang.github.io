---
title: hexo自定义主页
tags: ['Hexo','blog','index']
date: 2022-05-16
categories: ['blog']
---

经过了[上文](https://zermzhang.github.io/2022/04/19/blog/hexo%E8%87%AA%E5%8A%A8%E5%8C%96%E9%83%A8%E7%BD%B2/)的介绍之后，我们可以简单的将自己的blog搭建起来。
这个时候，blog的主页根据你选择的主题会有一定的不同，但是大多是以文章列表作为主页，如下图：

<!--more-->

![原始主页](https://raw.githubusercontent.com/ZermZhang/pictures/main/20220517084558.png)
但是，像我们这种喜欢折腾的人，怎么能接受一直使用默认的页面作为主页呢？那么这时候，自定义主页的需求就开始提上日程了。
这里主要介绍一种简单的自定义hexo主页的方法，这种方法其实是通过将一个文章内容作为首页展示的这种方案。


# 1. 主页内容
想要是用自定义主页，当然需要先创建一个自定的主页内容文件，该文件需要位于`/source/`文件夹下，文件名需要是`index.md`，这样才能保证hexo能够读取到这个文件。
文件内容可以自定义，如果希望主页内容能够有足够的个性化的话，可以在md文件中通过html、css实现更复杂的功能展示。示例如下：
```markdown
<!--<span style="font-size: 40px;">欢迎</span>-->

> 这里是我在日常生活中进行记录的地方，包括又不限于工作、生活、知识、娱乐等方方面面。

<img src="https://s2.loli.net/2022/05/14/kJga8xRWGnB4Ueo.png" style="border-radius: 50%; box-shadow: rgb(221, 222, 222) 0px 0px 0px 5px, white 0px 0px 30px; width: 20%; display: inline-block; float: right;">
这里是<span style="font-size: 35px; color: #E76D34;">zermzhang</span>的个人博客。 一个毕业于普通学校的普通人。

## 我是：
* <span style="font-size: 35px; color: #3370AC;">小说迷</span>，各种小说都喜欢。
* <span style="font-size: 35px; color: #F9D171;">码农</span>，日常写代码。
* <span style="font-size: 35px; color: #AD2625;">普通人</span>，自我感觉没有出众的地方。

请开始[<span style="font-size: 35px; color: rgb(44, 218, 376);">阅读</span>](/articles/)我的blog吧。
```

# 2. _config.yml调整
为了保证能够正常展示自定义主页，并且能够将主页和需要展示的文章列表页进行恰当的链接，需要对_config.yml文件进行调整。
主要调整内容如下：
```yml
index_generator:
  path: '/articles/'
  per_page: 10
  order_by: -date
```
`index_generator`下的`path`需要调整为一个新的空文件夹的名字，该文件夹下没有内容，这样的话，hexo才能够将原来的主页渲染到这个路径下。
自定义主页需要调整的超链接也需要填入当前路径，当然可以是相对路径，从`/source/`下算起即可。

# 3. 调整结果
经过了上述调整后，你的自定义主页就可以生效了，尽情的享受与众不同的快乐吧！
![自定义主页](https://raw.githubusercontent.com/ZermZhang/pictures/main/20220517085426.png)