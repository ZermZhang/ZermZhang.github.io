---
title: 通过preprocessing构建tf2模型
tags: ['tensorflow','subclassing','custom model','preprocessin']
date: 2022-04-26
categories: ['tensorflow']
---
# 1. 背景
> tf2.x里增加了一类特殊的keras layers, 可以替代`tf.feature_column`进行特征处理，这一类的layers就是`preprocessing layers`

<!--more-->

通过preprocessing对特征进行处理，在实现逻辑上，相对feature_column要更统一一些。
本文主要是记录一些通过preprocessing进行特征处理的过程，以及从feature_column迁移到preprocessing的过程中需要注意的事项。

# 2. preprocessing过程构建
> 通过调用preprocessing对特征处理的逻辑和feture_column基本类似
> 这里主要涉及到两个过程，一个是直接声明preprocessing-layers，方便对原始数据进行处理
> 另外一个是inputs格式的声明，方便通过summary可视化分析模型和export模型

1. build_encoded_features
该函数会根据原始特征的格式不同进行不同的preprocessing layer的声明，方便后续的特征处理。
2. build_inputs
该函数负责生成不同特征的指定格式，方便输出可视化模型结构

```python
class EncodedFeatureBuilder:
    @staticmethod
    def build_encoded_features(feature_config):
        feature_encoder_type = feature_config['type']
        feature_encoder_params = feature_config['config']
        feature_embedding_params = feature_config.get('embed_config', None)
        if feature_encoder_type == 'hashing':
            encoding_layer = HashEmbeddingBuilder(
                feature_params=feature_encoder_params,
                emb_params=feature_embedding_params
            )
            return encoding_layer
        elif feature_encoder_type == 'vocabulary':
            encoding_layer = VocabEmbeddingBuilder(
                feature_params=feature_encoder_params,
                emb_params=feature_embedding_params
            )
            return encoding_layer
        elif feature_encoder_type == 'numerical':
            encoding_layer = NumericalBuilder(
                feature_params=feature_encoder_params,
                emb_params=feature_embedding_params
            )
            return encoding_layer
        elif feature_encoder_type == 'pre-trained':
            raise Exception("There is no preprocessing layer for type: {}".format(feature_encoder_type))
            pass
        elif feature_encoder_type == 'crossed':
            encoding_layer = CrossedBuilder(
                feature_params=feature_encoder_params,
                emb_params=feature_embedding_params
            )
            return encoding_layer
        else:
            raise Exception("There is no preprocessing layer for type: {}".format(feature_encoder_type))
            pass

    @staticmethod
    def build_inputs(feature_name, feature_config):
        feature_encoder_type = feature_config['type']
        if feature_encoder_type in ('hashing', 'vocabulary'):
            input_col = tf.keras.Input(shape=(1,), name=feature_name, dtype=tf.string)
            return input_col
        elif feature_encoder_type in {'numerical'}:
            input_col = tf.keras.Input(shape=(1,), name=feature_name, dtype=tf.float32)
            return input_col
        else:
            raise Exception("There is no preprocessing layer for type: {}".format(feature_encoder_type))
            pass
```

# 3. 模型结构
```python
class MLP(tf.keras.Model):
    def __init__(self, config):
        super(MLP, self).__init__()
        # init the feature config info
        self.config = config
        # definite the layers
        self.feature_encoder_layers = EncodedFeatureBuilder()
        self.dense_layer = tf.keras.layers.Dense(32, activation='relu')
        self.dropout_layer = tf.keras.layers.Dropout(0.5)
        self.output_layer = tf.keras.layers.Dense(1)

    def call(self, inputs):
        encoded_features = []
        for feature_name, feature_config in self.config.items():
            encoded_features.append(
                self.feature_encoder_layers.build_encoded_features(feature_config)(inputs[feature_name])
            )
        x = tf.keras.layers.concatenate(encoded_features)
        x = self.dense_layer(x)
        x = self.dropout_layer(x)
        x = self.output_layer(x)
        return x

    def build_graph(self):
        all_inputs = {}
        for feature_name, feature_config in self.config.items():
            all_inputs[feature_name] = self.feature_encoder_layers.build_inputs(feature_name, feature_config)
        return tf.keras.models.Model(
            inputs=all_inputs,
            outputs=self.call(all_inputs)
        )
```

# 4. 调用流程

```python
config = {
    'PhotoAmt': {
        'type': 'numerical',
        'config': {'bin_boundaries': [0., 1., 2. ,3., 4., 5., 6., 10, 30.]}
    },
    'Fee': {
        'type': 'numerical',
        'config': {'num_bins': 20}
    },
    'Age': {
        'type': 'numerical',
        'config': {'num_bins': 20}
    },
    'Type': {
        'type': 'hashing',
        'config': {'num_bins': 4}
    },
    'Color1': {
        'type': 'hashing',
        'config': {'num_bins': 8}
    }
}

feature_encoder = EncodedFeatureBuilder(config)
model = MLP(feature_encoder)
model.build_graph().summary()
tf.keras.utils.plot_model(model.build_graph(), to_file='./test_model.png', show_shapes=True)
```