---
title: airflow自定义operator
tags: ['airflow','调度器','custom','operator']
date: 2022-04-24
categories: ['airflow']
---
任意的自定义我们需要的operator是airlfow的一大优势，这极大的方便了我们在日常开发调度流程中的灵活性和可拓展性。
<!--more-->
# 1. 基本介绍
自定义airflow-operator的时候需要继承`airflow.models.baseoperator.BaseOperator`.
同时，至少需要对基类里的两个方法进行重写：
1. Constructor
    * 定义当前operator需要使用到的参数。
2. Execute：
    * 当operator进行执行的时候需要执行的代码。

**一个基本的自定义Operator的例子**：
```python
from airflow.models.baseoperator import BaseOperator

class HelloOperator(BaseOperator):
    def __init__(slef, name: str, **kwargs) -> None:
        super().__init__(**kwargs)
        self.name = name
    
    def execute(self, context):
        message = f"Hello {self.name}"
        print(message)
        return message
```