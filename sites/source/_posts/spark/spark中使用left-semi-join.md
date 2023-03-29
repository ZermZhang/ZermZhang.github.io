---
title: spark中使用left-semi-join
tags: ['spark', 'join', 'left semi join']
date: 2023-03-29 11:01
categories: ['spark']
math: true
---
> left semi join是在Hive中常用的一种join方式，但是他和常用的join也会有一些不同。spark中也有一样的处理逻辑。那么使用过程中有哪些需要外注意到的东西呢？

## 1. 使用方法
left semi join是IN/EXISTS的一种高效实现方法，可以用来处理两个DataFrame取交集的情况。
```python
a = spark.createDataFrame(
    [
      ("tom", "male", 6),
      ("jerry", "female", 18),
      ("kevin", "male", 32)
    ],["name", "gender", "age"]
)

b = spark.createDataFrame(
    [
        ('tom', 'male', 100),
        ('jerry', 'male', 50)
    ], ['name', 'gender', 'salary']
)
```
根据上面两个示例DataFrame，我们需要取b中name存在在a中的那些值，最简单的方法当然是通过join实现。
```python
same_name_df_join = b.join(a, a.name == b.name, how='inner').select([b.name, b.gender, b.salary])
same_name_df_join.show()
```
结果+耗时：
![join结果耗时情况](https://raw.githubusercontent.com/ZermZhang/pictures/main/20230329110540.png)
那么，这里我们可以通过left semi join实现想要的逻辑，可以用更短的时间实现更好的效果。
```python
# 根据name过滤b在a中
same_name_df = b.join(a, a.name == b.name, how='leftsemi')
same_name_df.show()
```
![left semi join结果耗时情况](https://raw.githubusercontent.com/ZermZhang/pictures/main/20230329110845.png)
注意结果里的耗时情况，当然，测试的DataFrame数据很少，耗时情况不太置信，但是也能从一定程度上说明left semi join的高效了。
当然，在使用left semi join的时候，也想join一样，可以通过多个key进行处理。
```python
# 根据name+gender过滤b在a中
same_name_gender_df = b.join(a, (a.name == b.name) & (a.gender == b.gender), how='leftsemi')
same_name_gender_df.show()
```
![多个key进行left semi join](https://raw.githubusercontent.com/ZermZhang/pictures/main/20230329110746.png)

## 2. 注意事项
1. 使用Left semi join的时候，最终的输出结果里的col只能是存在在左边DataFrame的col，如果只在右边的DataFrame里有的话，是不能关联到结果里的
2. 右边DataFrame如果存在重复数据的话，left semi join会直接跳过重复数据，而不会像join一样生成过个记录，这一点要额外注意。