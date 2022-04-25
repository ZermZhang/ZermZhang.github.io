---
title: tf2中subclassing自定义模型
tags: ['tensorflow','subclassing','save model']
date: 2022-04-25
categories: ['tensorflow']
---
# 1. tf2中的subclassing模型
tf2中常见的定义模型的方法分成三种：functional、sequenctial、subclassing

<!--more-->

其中，subclassing方法就是自己实现一个类来继承`tf.keras.Model`，来构建一个Model类的子类，在构建过程中，主要需要实现下面两个方法：
1. `__init__()`
这个方法是子类的构造器，主要是用来初始化参数（比如layers之类的）。
`super`主要用来初始化父构造器。
2. `call()`
该函数用于执行在`__init__`中定义的layers操作
同时，该函数可以使我们定义的模型直接作为函数使用，该功能主要依赖于python的`__call__`实现。

