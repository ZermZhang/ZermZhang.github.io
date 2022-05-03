---
title: tf中ProfilerHoo的使用
tags: ['tensorflow','profilerhook']
date: 2022-05-03
categories: ['tensorflow']
---
# 1. ProfilerHook
在深度学习训练过程中，经常会遇到性能的问题，为了提高训练过程的效率，各个训练环节的耗时分析和性能优化是必须的。
而如果通过手动打印日志和耗时情况，不仅繁琐，而且准确度不高。那么我们可以通过tf提供的Profiler工具进行相关分析。

<!--more-->

tf里的`tf.estimator.ProfilerHook`就是一个可以定期收集分析信息的接口，通过该接口收集到的信息会被保存到"timeline-${step}.json"文件中，这些文件可以直接通过`chrome://tracing/`导入可视化，方便具体分析每个流程的耗时情况。

# 2. 使用方法
```python
def train_and_eval(model):
    """
    :param model: 声明的estimator实例
    :return: None
    :usage: 进行模型训练，并在指定步长的时候进行结果评估
    """
    timeline_hook = tf.train.ProfilerHook(save_steps=100, output_dir=os.path.join(
            os.getcwd(), './timeline_track'
        ))

    hook = tf.contrib.estimator.stop_if_no_increase_hook(
        model,
        metric_name='ctcvr_cvr_auc_esmm',
        max_steps_without_increase=configuration_params['max_steps_without_increase'],
        # maximum number of training steps with no decrease in the given metric.
        min_steps=configuration_params['min_steps'],  # stop is never requested if global step is less than this value
        run_every_steps=configuration_params['run_every_steps'],
        run_every_secs=None
    )

    train_spec = tf.estimator.TrainSpec(
        input_fn=lambda: input_fn(os.path.join(os.getcwd(),
                                               CONFIG_TRAIN['train_data']),
                                  'train', CONFIG_TRAIN['batch_size']),
        hooks=[hook, timeline_hook]
    )

    eval_spec = tf.estimator.EvalSpec(
        input_fn=lambda: input_fn(os.path.join(os.getcwd(),
                                               CONFIG_TRAIN['test_data']),
                                  'eval', 128),
        steps=CONFIG.evalconfig['steps'],
        throttle_secs=30
        )

    tf.estimator.train_and_evaluate(model, train_spec, eval_spec)
```
## 2.1 参数说明
|参数|说明|
|--|--|
|save_steps|间隔多少步保存一次profiler trace|
|save_secs|间隔多少秒保存一次profiler trace，save_steps和save_secs只设置一个|
|output_dir|profiler trace文件的存储路径|

## 2.2 输出信息
![profiler-output](https://s2.loli.net/2022/05/03/7YpS2AOl5qHuECo.png)

* timeline-${step}.json：每个报错步长输出的profiler trace文件

# 3. web分析
1. 在浏览器中打开`chrome://tracing/`页面
![tracing-page](https://s2.loli.net/2022/05/03/t1NwGXKhT7FsgZl.png)
2. 通过`load`将profiler trace文件导入浏览器
3. 分析具体耗时情况
![time-cost](https://s2.loli.net/2022/05/03/w9rgJIdiGEABQX3.png)
