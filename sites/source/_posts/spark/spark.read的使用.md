---
title: spark.read的使用
tags: ['spark','read']
date: 2022-08-31 19:53
categories: ['spark']
---
> spark在读取csv文件的时候，会涉及到许多参数，这些参数的作用各有不同，这里进行一个简单的介绍。

## 1. spark读取csv文件
```python
df = spark.read.format('csv').option('sep', '\t').schema(schemas).load(path)
```

## 2. option参数介绍
|参数|解释|默认值|
|---|---|---|
|sep|读取文件内容后，对同一行内容实用该参数指定的分割符进行分割|`,`|
|encoding|解码方式|`utf-8`|
|quote|通过当前配置值包裹的内容，不通过sep进行分割|`"`|
|escape|当quote字符内部还有quote字符时，可以通过当前字符进行转义|`\`|
|header|是否将文件的第一行作为列名|`false`|
|inferSchema|从数据进行schema推断|`false`|