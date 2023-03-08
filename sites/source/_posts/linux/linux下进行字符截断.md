---
title: linux下进行字符截断
tags: ['Linux', 'string', 'cut']
date: 2023-03-08 11:36
categories: ['Linux']
math: true
---
在日常的操作中，我们经常会遇到需要对字符串进行截断的操作，如果是在python中可以直接通过对string的index进行筛序获得想要的子字符串。
那么，在shell命令行中，有没有类似的操作方法呢？答案是有的，而且有多种实现方法~

## 1. 变量截取
> 这种方法，可以在使用变量的时候进行快速截取，非常类似python中的a[:1]的使用方法。

**使用方法如下**：
```shell
str='1234567890'

# sub_str=${str: start_index: length}
sub_str=${str: 3: 4}
```
* str: 需要被截断的字符串
* start_index：起始index，注意在这里使用的时候index是从0开始的
* length：需要截取的长度

#### 1.1 省略写法
1. 省略起始位置
```shell
sub_str=${str: :4}
echo $sub_str
# 1234
```
省略起始位置的话，相当于默认从0开始阶段

2. 省略长度
```shell
sub_str=${str: 7}
echo $sub_str
# 890
```
省略截断长度的额话，相当于从指定位置截取到字符串末尾

#### 1.2 从右边开始计数
如果想从字符串的右边开始计数，对字符串进行阶段的话，可以使用如下操作方式：
```shell
# sub_str=${str: 0-start_index: length}
sub_str=${str: 0-3: 2}
echo $sub_str
# 89
```
* 注意, 相较从左边开始计数，start_index, length含义不变。
* 但是start_index前面多了`0-`字符，这是固定写法，专门用来标明从字符串右边开始计数
* PS：从右边开始计数的时候字符的index是从1开始的，和默认从左边开始不一样，需要额外注意

#### 1.3 从指字符开始
1. 根据指定字符截取右边子串
```shell
sub_str=${str#*7}
echo $sub_str
# 890
```
如果你的字符串中包含多个需要匹配的字符，上面的匹配方法会输出从第一个字符开始到结尾的子串。
```shell
str="12732723"
sub_str=${str#*7}
echo $sub_str
# 32723
```
虽然当前方法无法指定具体从哪个字符开始，但是提供了一种从最后一个匹配到的字符开始截取的方法，如下：
```shell
str="12732723"
sub_str=${str##*7}
echo $sub_str
# 23
```
2. 根据指定字符截取左边子串
同样，通过指定字符匹配的方法，也可以从左侧开始，不过指定的格式变成了`%chars*`
```shell
str="12732723"
sub_str=${str%7*}
echo $sub_str
# 12732
```

#### 1.4 总结
|格式|说明|
|--|--|
|${str: start_index: length}|从str左边第start_index个字符开始，向右边截取length个字符|
|${str: start_index}|从str左边第start_index个字符开始，截取到str末尾|
|${str: 0-start_index: length}|从str右边第start_index个字符开始，向右边截取length个字符|
|${str: 0-start_index}|从str右边第start_index个字符截取到str末尾|
|${str#*chars}|从str中第一次出现chars的位置开始，截取到str末尾|
|${str##*chars}|从str中最后一次出现chars的位置开始，截取到str末尾|
|${str%chars*}|从str右边开始算起，第一次出现chars开始（即str左边起最后一次出现chars），截取chars左边的所有字符|
|${str%%chars&}|从str右边开始算起，最后一次出现chars开始（即str左边起第一次出现chars），截取chars左边所有字符|

## 2. cut函数
上面通过变量进行截取不太方便进行进一步的操作，只能赋给一个新的变量。为了方便处理，也可以使用cut函数进行处理。
cut命令可以从文件的每一行对字节、字符、字段进行处理，并将处理后的结果写到标准输出。
如果不指定file参数的话，cut可以读取标准输入。
**常用使用方法**:
```shell
cut [-bn] [file]
cut [-c] [file]
cut [-df] [file]
```
* -b: 以字节为单位进行分割。注意，这些字节位置会忽略多字节的字符边界
* -c：以字符为单位进行分割
* -d：自定义分隔符，模人物制表符
* -f：与-d一起使用，指定显示哪个区域
* -n：取消分割多个字节字符。仅和-b一起使用

#### 例子：
```shell
str='1234567890'

echo $str | cut -c 7
# 7，输出第7个字符
echo $str | cut -b 1-7
# 1234567，输出第1到第7个字符
# 注意：cut的序号从1开始，而且不会忽略右侧的index

str='12t34t56t78'
echo $str | cut -d 't' -f2
# 34
```