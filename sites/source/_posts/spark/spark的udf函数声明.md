---
title: spark的udf函数声明
tags: ['spark', 'udf']
date: 2023-03-27 18:56
categories: ['spark']
math: true
---
> udf（user defined function）函数，是spark中针对内建函数无法满足用户需求是，提供给用户的一种自定义处理函数的方式。
> 很好的扩展了用户在处理dataframe数据时候的自由度和便捷性。

## 0. 产生例子
```python
import pyspark.sql.functions as f
from pyspark.sql.types import StringType

df = spark.createDataFrame(
    [
      ("tom", "male", 6),
      ("jerry", "female", 18),
      ("kevin", "male", 32)
    ],["name","age"]
)

df.createOrReplaceTempView("namge_age_df")
```
显示结果：
![df展示](https://raw.githubusercontent.com/ZermZhang/pictures/main/20230327192912.png)
后续会在该示例df上展示pdf的功能

## 1. udf创建方法
udf函数有多种创建方法，但是，每种创建方法都离不开最基本的python函数。
这里的python函数和日常本地程序中使用到的函数没有任何的不通，大家可以通过最常用的方法进行处理。
这里，我们通过一个对年龄进行分类的例子进行说明。

#### 1.1 python函数的例子
```python
def bucket_age(age):
    if age < 16:
        return 'child'
    elif age < 30:
        return 'young'
    else:
        return 'adult'
```
* 这是一个很简单的函数，我们通过16、30两个年龄，将所有人划分成了三个年龄段，如果是在本地的pyhton数据结构中，我们可以直接使用当前的pyhton函数进行处理，但是当我们面临的是一个数据量极多的pyspark-DataFrame的时候，我们就需要将当前函数转换成可以对DataFrame进行操作的udf函数。

#### 1.2 使用@装饰器注册udf函数
```python
@f.udf(returnType=StringType())
def bucket_age(age):
    if age < 16:
        return 'child'
    elif age < 30:
        return 'young'
    else:
        return 'adult'
```
* 使用这种方式注册udf，比较简单，编程逻辑和正常的python函数基本一致
* 但是将当前函数注册为udf函数后，不能独立命名，只能使用做udf函数，而且无法不方便处理多参数输入

#### 1.3 直接注册为udf函数
```python
def bucket_age(age):
    if age < 16:
        return 'child'
    elif age < 30:
        return 'young'
    else:
        return 'adult'

udf_bucket_age = f.udf(bucket_age, StringType())
```
* 最常见的注册方式，python函数和注册过程相互独立，方便测试和开发

#### 1.4 使用闭包注册udf函数
```python
def udf_bucket_age():
    def bucket_age(age):
        if age < 16:
            return 'child'
        elif age < 30:
            return 'young'
        else:
            return 'adult'
    return f.udf(bucket_age, StringType())
```
* 方便处理多参数输入的方式，而且能够在闭包函数中独立输入不变变量，方便区分不通的变量逻辑

## 2. udf的使用
#### 2.1 基于列的udf使用
最常见的函数使用方式，即将当前的udf函数应用到dataframe的一列数据上，针对当前列进行处理，生成新的列。
```python
new_df = df.withColumn('bucket_age', udf_bucket_age(f.col('age')))
new_df.show()
```
![udf处理](https://raw.githubusercontent.com/ZermZhang/pictures/main/20230327192952.png)
* 如上，经过了注册后的udf函数处理后，dataframe中生成了新的一列，这一列正是对年龄的结果分桶后的结果。而且结果也正是我们在python函数中声明的逻辑。

#### 2.2 基于agg的udf使用
在日常使用中，还有一种方式出现频率较高，那就是针对group聚合之后的结果进行处理，在这种处理中，udf函数也是一种常见的处理方式。
比如，我们将上述Dataframe根据gender聚合后，计算每种年龄人群的平均年龄。我们可以通过内建函数和udf函数分别计算，来说明udf函数在这种情况下的使用方法：
```python
# 1. 注册udf函数
def avg_age(ages):
    return sum(ages) / len(ages)

udf_avg_age = f.udf(avg_age, DoubleType())

# 2. 对group数据进行处理
df.groupby('gender').agg(
    f.avg('age').alias('in_avg_func'),
    udf_avg_age(f.collect_list(f.col('age'))).alias('udf_avg_func')
).show()
```
* 输出结果：
![输出结果](https://raw.githubusercontent.com/ZermZhang/pictures/main/20230327200604.png)