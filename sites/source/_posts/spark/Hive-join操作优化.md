---
title: Hive-join操作优化
tags: ['spark', 'hive', 'join']
date: 2023-03-20 15:31
categories: ['spark']
math: true
---
Join是Hive SQL操作中极为常见的一种，常出现在两个表处理数据的过程中互相关联的操作过程。
Join操作也是Hive SQL中经常出现问题的一种操作，最常见的问题就是出现较为严重的数据倾斜，导致运行时间远超预期；或者是出现了笛卡尔积，导致输出的数据量和预期的数据量不一致。

### Join的tedia能
1. 只支持等值连接
2. 底层处理的时候会将Hive SQL的代码转换成MapReduce过程，而且Reduce过程会按照join的过程从左到右（除最后一个表外）对涉及到的表进行缓存
3. 当三个或者是更多个表进行join时，如果每个on使用相同的字段连接，只会产生一个MapReduce


### Join优化的几条方法
1. 合理设置map和reduce的数量
   * 设置map数目
   `set mapred.reduce.tasks=10;`
   * 设置reduce数目
   `set mapred.reduce.tasks = 15;`
   `set hive.exec.reducers.bytes.per.reducer=500000000; （500M）`
2. 对于没有相互依赖关系的阶段可以通过并行执行任务进行加速
   `set hive.exec.parallel=true`
3. 使用left semi join替代IN，EXIST等操作
   left semi join 是 IN、EXIST的一种高效实现
   * 示例：
    ```
    select a.key, a.value from a
    where a.key in (select b.key in b)

    select a.key, a.value from a
    left semi join
    b
    on a.key = b.key
    ```
    * left semi join右边的表只能在on中设置过滤条件，不能在select子句中设置
    * left semi join的select结果只能出现左边表的值
    * left semi join的on条件遇到右边表存在重复值的情况，遇到第一条就会结束
4. 当使用多个表进行join的时候，注意join的顺序
   join的表从左到右应该是从小表到大表。
   因为Hive在进行操作的时候会把其他表都缓存起来，直到join的最后一个表
5. 小表进行join的时候使用mapjoin
   为了加速计算，可以将小表先缓存起来。
   1. 自动缓存
    ```hive
    set hive.auto.convert.join=true; 是否开启自动将小于设置值的表进行缓存
    set hive.mapjoin.smalltable.filesize=300000000;     可以进行缓存的表的最大值，300000000，即30M
    set hive.auto.convert.join.noconditionaltask=true;  是否将多个mapjoin合成一个
    set hive.auto.convert.join.noconditionaltask.size=300000000;    多个mapjoin转换为1个时，所有小表的文件大小总和的最大值。
    ```
    2. 手动缓存
    ```hive
    select  /*+ mapjoin(x)*/  x.a,  y.b from t_x x join t_y y on x.id=y.id;
    ```
6. jion中的数据亲写处理
   ```hive
    set hive.optimize.skewjoin=true;
    set hive.skewjoin.key=100000;
   ```
