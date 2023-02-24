---
title: hexo-blog中配置个人简历
tags: ['Hexo','blog','resume']
date: 2023-02-24
categories: ['blog']
---
近期在整理个人简历的时候，突发奇想，准备将我自己的简历放到github-pages上部署的blog里。所以就研究了一下如何在hexo中配置个人简历，一番折腾下来，确实配好了，但是却达不到完全通过markdown就可以实现我想要的效果。
最终的结果就是在markdown里面增加了许多的html相关的配置，才达到了一个简单的效果。
折腾过程就记录在这里，方便后续查阅吧。

## 最终效果
<img src='https://raw.githubusercontent.com/ZermZhang/pictures/main/20230224085327.png' width=40% alt="简历效果">

# 1. 前期准备
为了实现我想要的简历效果，最主要的难点在于如何通过markdown控制简历中各类型信息的排版问题，最常见的方法就是通过表格。但是markdown中的表格无法自由的控制单元格宽度和表格宽度，最终只能通过空单元格解决。
### 1.1 表格排版
> 通过空单元格对信息排版进行控制

* 正常的单元格配置
```markdown
|name1|name2|
|---|---|
|age1|age2|
```
|name1|name2|
|---|---|
|age1|age2|

* 具有空单元格的表格
```markdown
|name1||name2|
|---|---|--|
|age1||age2|
```
|name1||name2|
|---|---|---|
|age1||age2|

* 通过增加一个空单元格的方法，可以让简历中的信息不用都聚集在屏幕的左侧，但是当前表格的边框影响了么美观，为了将单元格的边框去掉，只好通过当前页面的自定义style信息配置实现。
```html
<style>
td, th, tr {
   border: none!important;
}
</style>
```
在文档的开始部分增加如上配置，可以使当前页面的所有表格隐藏边框，提升信息排版的美观度。
![无边框表格](https://raw.githubusercontent.com/ZermZhang/pictures/main/20230224085040.png)

### 1.2 图标配置
在上面的简历图片中可以看到使用了很多的矢量图图标，提升了简体整体的丰富度。因为我觉着再web端不需要像纸质简历那么严肃，可以加一些有趣的东西在里面，所以我在简历中将本来用文字描述的部分增加了一些图标。
在图标的选择上，可以根据个人喜好来定，常见的iconfont、makrdown-emoji都可以，我这里选用了[font-awesome](https://fontawesome.com/search)来用，因为我觉这款图标比较符合我的审美，而且使用简单。
* 使用方法：
```html
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.3.0/css/all.css">
```
* 在文档开始部分增加如上配置，就可以在文档里随时使用font-awesome的矢量图标
![font-awesome图标](https://raw.githubusercontent.com/ZermZhang/pictures/main/20230224091017.png)
将图中的html标签复制下来，就可以在文档里显示了，就像这样<i class="fa-solid fa-house"></i>