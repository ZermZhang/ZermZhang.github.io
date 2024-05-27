---
title: python常用的udf的简要说明
tags: ['python', 'udf']
date: 2024-05-24 11:44:36
categories: ['python']
math: true
---

# 说明
udf是在pandas数据处理和pyspark数据处理开发过程一个常用的逻辑，即自定义函数。
虽然pandas和pyspark都提供了强大和灵活的内置处理逻辑，但是在遇到一些复杂的处理过程的时候，udf还是无法绕过的一步。

# 常用的udf类型
|作用范围|方法名|备注|
|---|---|---|
|pd.Series|map|把自定义函数映射到每一个元素上，同时可以把dict映射到元素上，通过key自动映射value|
|pd.Series|apply|只能把自定义函数映射到每一个元素上|
|pd.DataFrame|map|其实就是pd.Series的map，只能作用在pd.DataFrame的指定列，也就是一个Series上|
|pd.DataFrame|apply|把自定义函数映射到指定的坐标轴上，axis=0，纵轴，axis=1, 横轴|
|pd.DataFrame|applymap|把自定义函数映射到每一个元素上|
|pd.DataFrame|assign|为pd.DataFrame拓展列|
|pyspark.sql.DataFrame|mapInPandas|把df的每个分区当成pd.DataFrame来处理，需要返回一个df的可迭代器|
|pyspark.sql.DataFrame|applyInPandas|把group之后的每个分区内容当成pd.DataFrame来处理，返回df|