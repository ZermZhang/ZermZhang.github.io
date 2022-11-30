---
title: markdown中使用流程图
tags: ['markdown','流程图','blog','mermaid']
date: 2022-11-30
categories: ['blog']
---

> markdown使我们在工作中常用的一种工具，可以极大的优化我们在文档编写方面的效率。
>
> 同样，流程图在日常工作中也是一件必不可少的利器。
>
> 那么有没有什么方法可以将这两种格式融合到一起进行处理呢？

## 1. mermaid介绍

> Meimaid是一个基于Javascript的图标绘制工具，通过解析类Markdown的文本语法来实现图表创还能和动态修改。
>
> Mermaid诞生的主要目的是让文档的更新能够及时跟上开发的进度。

上面是mermaid官方仓库对mermaid的介绍。

在我的日常使用中，可以很好的体会到mermaid在这一方面的努力，通过在markdown中集成mermaid，将之前繁琐的流程图绘制工作简化成了大家熟悉的类markdown语言编写过程，极大的方便了文档编写过程中的快速变更和方便阅读。

[mermaid官方仓库](https://github.com/mermaid-js/mermaid)

## 2. mermaid支持的图表类型

| 图表类型       | 关键字          |
| -------------- | --------------- |
| 流程图         | flowchart       |
| 时序图         | sequenceDiagram |
| 甘特图         | gantt           |
| 类图           | classDiagram    |
| 状态图         | stateDiagram    |
| 饼图           | pie             |
| Git图          | gitGraph        |
| 用户体验旅程图 | journey         |
| C4图           | C4Context       |

## 3. mermaid使用示例

* 通过如下代码可以直接在markdown中生成指定的流程图

```shell
flowchart LR
	a[hard] -->|relation| b(Round)
	b --> c{Decision}
	c --> D((result 1)) & E[result 2]
```

![image-20221130181955501](https://raw.githubusercontent.com/ZermZhang/pictures/main/image-20221130181955501.png)



* 说明：
  * `flowchart LR`: 通过flowchart声明需要生成流程图，LR代码图的走向为从左到右
    * LR：从左（left）到右（right）
    * RL：从右（right）到左（left）
    * TB：从上（top）到下（bottom）
    * BT：从下（bottom）到上（top）
  * `a[hard]`：声明节点，节点名为a，``[hard]``表示当前节点为文本为hard的直角矩形框
    * `()`: 圆角矩形框
    * `[]`: 直角矩形框
    * `{}`：判断框
    * `(())`：圆形框
    * 当然，mermaid还支持很多其他类型的框图，具体可以参考[文档](https://mermaid-js.github.io/mermaid/#/flowchart)
  * `-->`：节点之间的连接线