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

# 2. airflow的时间说明
> airflow中存在多种时间信息，包括dag开始时间、结束时间、调度时间等
> 比较容易产生混乱

![image.png](https://raw.githubusercontent.com/ZermZhang/pictures/main/20230417133930.png)
如上图：
* Run：dag的调度时间，即execution_date，注意该时间相较当前的执行时间会早一个调度周期
* Started：dag的启动时间，即当前dag启动时的系统时间
* Ended：dag的结束时间，即当前dag运行结束时的系统时间

**时间获取方法：**
```python
execution_date = "{{ execution_date }}" # yyyymmdd
execution_date = "{{ ds }}" # yyyy-mm-dd
execution_date = "{{ ds_nodash }}" # yyyymmdd
beijing_today = "{{ (macros.datetime.utcnow() + macros.timedelta(hours=8)).strftime('%Y-%m-%d %H') }}"
```