---
title: airflow使用技巧
tags: ['airflow','trick']
date: 2022-05-31 13:41
categories: ['airflow']
---

> 本文主要针对airflow的一些简单的使用技巧进行归纳总结，方便记录。

# 1. 复杂Variable的使用
Variables是airflow中一个用来存储和重载自定义变量的功能，可以极大的简化针对dag中可能用到的变量信息。
最常用的方式就是在Variables中存储某一个值，需要的时候可以通过`var = Variable.get("my_var")`进行获取。
但是当你需要使用到复杂个是的variable的时候，上述逻辑就会遇到问题。比如：
![airflow-variables](https://s2.loli.net/2022/05/31/mNxLnk6pqbrOdQu.png)
这个时候，针对变量`example_variables_config`的使用就需要使用json进行导入，导入方法如下：
```python
dag_config = Variable.get("example_variables_config", deserialize_json=True)
var1 = dag_config["var1"]
var2 = dag_config["var2"]
var3 = dag_config["var3"]
```