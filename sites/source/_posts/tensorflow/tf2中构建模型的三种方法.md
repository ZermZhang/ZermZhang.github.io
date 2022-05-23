---
title: tf2中构建模型的三种方法
tags: ['tensorflow','subclassing','sequenctial','functional']
date: 2022-04-29
categories: ['tensorflow']
---
# 0. 背景
tf2版本中，提供了三种不同的模型构建逻辑。

<!--more-->

1. sequential API
适用于层叠式的模型层结构，并且每层都只有一个明确的输入tensor和输出tensor
2. functional API
函数式ＡＰＩ是一种比Sequential API跟加简单和灵活的模型创建方式。
函数式API可以处理具有非线性拓扑的模型、具有共享层的模型，以及具有多个输入或输出的模型
3. model subclassing API
model subclassing API具有更大的自由度，可以让开发者控制模型、layer以及训练过程

![diff-api](https://raw.githubusercontent.com/ZermZhang/pictures/main/diff-api.jpeg)

# 1. Sequential API
## 1.1 构建方法
构建Sequential 模型主要通过Sequential constructor对layers的列表进行模型构建
```python
model = tf.keras.Sequential(
	[
        tf.keras.Input(shape=(512, 128)),
        tf.keras.layers.Dense(64, activation="relu"),
        tf.keras.layers.Dense(32, activation="relu")
    ]
)
```
![show](https://cdn.jsdelivr.net/gh/ZermZhang/pictures@main/PicX/show.5w4zduij7tw0.webp)

## 1.2 其他构建方法
* 除了直接通过sequential constructor对layers list进行模型构建外，也可以通过sequential constructor的add属性逐层进行构建
* 注意，通过这种方法进行模型构建的时候可以将一个 ​name​ 传入到sequential constructor中
```python
# 声明模型
model = tf.keras.Sequential(name='my_test_model')

# 声明模型的各层结构
model.add(tf.keras.Input(shape=(512, 128)))
model.add(tf.keras.layers.Dense(64, activation='relu'))
model.add(tf.keras.layers.Dense(32, activation='relu'))
```

# 2. Functional API
## 2.1 构建方法
* 注意：通过Functional API进行模型构建的时候，需要先创建一个输入节点：`inputs = tf.keras.Input(shape=(512, 128))`
* 然后可以在`inputs`对象上调用层，在层计算图中创建新的节点, `tf.keras.layers.Dense(64, activation='relu')(inputs)`
* 每调用一个层，就意味着将上一个层的输出作为新的调用层的输入，并通过该层处理后进行输出结果。
* 最后，可以通过`tf.keras.Model`指定输入和输出来创建模型。

```python
# 创建输入层
inputs = tf.keras.Input(shape=(512, 128))
# 中间层可以调用多个layers
x = tf.keras.layers.Dense(64, activation='relu')(inputs)
# 创建输出层
outputs = tf.keras.layers.Dense(32, activation='relu')(x)

# 声明模型
model = tf.keras.Model(inputs=inputs, outputs=outputs, name='my_test_model')

model.summary()
tf.keras.utils.plot_model(model, show_shapes=True)
```

## 2.2 使用模型进行训练、评估和推断
通过Functional API和Sequential API构建的模型，都可以通过同一种方法进行模型的训练、评估和推断工作。
```python
# 通过tf.data.datasets创建data_generator
batch_data = tf.data.datasets(...)

# 模型编译, 模型编译过程中需要声明模型训练过程中需要用到的loss_func, optimizer_func和评估需要用到的metric_func
# 用到的这三种func都可以是按照规范自定义的或者式已经存在的
loss_func = tf.keras.losses.SparseCategoricalCrossentropy(from_logits=True)
optimizer_func = tf.keras.optimizers.RMSprop()
# 直接通过tf.keras.metrics中API的name进行调用，类似'accuracy', 'AUC'等
metrics_name = ['accuracy']

# 模型编译
model.compile(loss=loss_func, optimizer=optimizer_func, metrics=metrics_name)

# 使用已经编译好的模型进行训练
# model.fit(x_train, y_train)
# model.fit 可以作用在分离的features和labels上，也可以直接作用在data_generator上
model.fit(batch_data)

# 模型评估
test_score = model.evaluate(batch_data)
```

# 3. Subclassing API
```python
class TestModel(tf.keras.Model):
    def __init__(self):
        super(TestModel, self).__init__()
        self.dense1 = tf.keras.layers.Dense(64, activation='relu')
        self.dense2 = tf.keras.layers.Dense(32, activation='relu')
    
    def call(self, inputs):
        emb = self.dense1(inputs)
        emb = self.dense2(emb)
        return emb
    
    def build_graph(self, input_shape):
        input_ = tf.keras.layers.Input(shape=input_shape)
        return tf.keras.models.Model(inputs=[input_], outputs=self.call(input_))

model = TestModel()

model.build_graph(input_shape=(512, 128)).summary()
tf.keras.utils.plot_model(model.build_graph(input_shape=(512, 128)), show_shapes=True)
```
subclassing种的build_graph不是必须的，但是不具有build_graph的subclass无法进行模型结构的展示。
可以通过如下[方式](https://zermzhang.github.io/2022/04/25/tf2%E4%B8%ADsubclassing%E8%87%AA%E5%AE%9A%E4%B9%89%E6%A8%A1%E5%9E%8B/)，进行模型结构的展示。