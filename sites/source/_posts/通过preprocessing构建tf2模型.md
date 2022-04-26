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


```python
class FeatureBaseBuilder:
    """
    feature_params: the params for preprocessingLayer for input features
    emb_layer_params: the params for embedding layer
    use_emb_layer: use the layers.Embedding or not
        when yes: return the embedding features
        when no: return the feature after preprocessing layer
    """
    def __init__(self, feature_params: dict, emb_params: dict = None,
                 use_emb_layer: bool = True):
        self.feature_params = feature_params
        self.emb_params = emb_params
        self.use_emb_layer = use_emb_layer

    def input_layer(self):
        return preprocessing.PreprocessingLayer(**self.feature_params)

    def __call__(self, inputs):
        if self.use_emb_layer:
            if self.emb_params:
                return layers.Embedding(**self.emb_params)(self.input_layer()(inputs))
            else:
                input_dim = (
                    self.feature_params['num_bins']
                    if 'num_bins' in self.feature_params else
                    len(self.feature_params['bin_boundaries'])
                )

                emb_params = {
                    'input_dim': input_dim,
                    'output_dim': 8
                }
                return layers.Embedding(**emb_params)(self.input_layer()(inputs))

        else:
            return self.input_layer()(inputs)


class HashEmbeddingBuilder(FeatureBaseBuilder):
    def input_layer(self):
        return preprocessing.Hashing(**self.feature_params)


class VocabEmbeddingBuilder(FeatureBaseBuilder):
    def input_layer(self):
        return preprocessing.StringLookup(**self.feature_params)


class NumericalBuilder(FeatureBaseBuilder):
    def input_layer(self):
        return preprocessing.Discretization(**self.feature_params)


class CrossedBuilder(FeatureBaseBuilder):
    def input_layer(self):
        return tf.keras.layers.experimental.preprocessing.HashedCrossing(**self.feature_params)


class EncodedFeatureBuilder:
    def __init__(self, config: dict):
        self.config = config
        self.all_inputs = []
        self.encoded_features = []
        self.feature_builder()

    def feature_builder(self) -> None:
        for feature_name, feature_config in self.config.items():
            if feature_config['type'] == 'hashing':
                input_col = tf.keras.Input(shape=(1,), name=feature_name, dtype=tf.string)
                encoding_layer = HashEmbeddingBuilder(feature_params=feature_config['config'])
                self.all_inputs.append(input_col)
                self.encoded_features.append(encoding_layer(input_col))
            elif feature_config['type'] == 'vocabulary':
                input_col = tf.keras.Input(shape=(1,), name=feature_name, dtype=tf.string)
                encoding_layer = VocabEmbeddingBuilder(feature_params=feature_config['config'])
                self.all_inputs.append(input_col)
                self.encoded_features.append(encoding_layer(input_col))
            elif feature_config['type'] == 'numerical':
                input_col = tf.keras.Input(shape=(1,), name=feature_name, dtype=tf.float32)
                encoding_layer = NumericalBuilder(feature_params=feature_config['config'])
                self.all_inputs.append(input_col)
                self.encoded_features.append(encoding_layer(input_col))
            elif feature_config['type'] == 'pre-trained':
                raise Exception("There is no preprocessing layer for type: {}".format(feature_config['feature_type']))
                pass
            elif feature_config['type'] == 'crossed':
                input_col = tf.keras.Input(shape=(1,), name=feature_name, dtype=tf.string)
                encoding_layer = CrossedBuilder(feature_params=feature_config['config'])
                self.all_inputs.append(input_col)
                self.encoded_features.append(encoding_layer(input_col))
            else:
                raise Exception("There is no preprocessing layer for type: {}".format(feature_config['feature_type']))
                pass
```

# 3. 模型结构
```python
class MLP(tf.keras.Model):
    def __init__(self, feature_encoder):
        super(MLP, self).__init__()
        self.dense_layer = tf.keras.layers.Dense(32, activation='relu')
        self.dropout_layer = tf.keras.layers.Dropout(0.5)
        self.output_layer = tf.keras.layers.Dense(1)
        self.feature_encoder = feature_encoder

    def call(self, inputs):
        x = tf.keras.layers.concatenate(inputs)
        x = self.dense_layer(x)
        x = self.dropout_layer(x)
        x = self.output_layer(x)
        return x
    
    def build_graph(self, shapes=None):
        return tf.keras.models.Model(
            inputs=self.feature_encoder.all_inputs,
            outputs=self.call(self.feature_encoder.encoded_features))
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