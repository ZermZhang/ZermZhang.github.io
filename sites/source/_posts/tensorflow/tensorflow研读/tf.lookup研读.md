---
title: tf.lookup研读
tags: ['tensorflow','lookup','Module']
date: 2022-06-16
categories: ['tensorflow']
---
> tf.lookup模块就是tf的字典功能，方便在大量备选的元素中根据key进行快速筛选。

# 1. 主要构成
tf.lookup模块包含了5个class和1个Module. Module中包含两个实验性质的class.
同样，这几个class可以按照我们的使用习惯，简单的分成3类.
|分类|Class|说明|
|---|----|----|
|从tensor生成table|KeyValueTensorInitializer|根据k,v对生成table initializer|
|从tensor生成table|StaticHashTable|从table initializer生成可以检索的table|
|从tensor生成table|StaticVocabularyTable||
|从文件内容生成table|TextFileInitializer||
|从文件内容生成table|TextFileIndex||
|先生成可变table|MutableHashTable||
|先生成可变table|DenseHashTable||

# 2. 从tensor生成table
通过现有的tensor生成可以检索的table，需要提供现成的key，val数据，经过KeyValueTensorInitializer生成init后，便可以通过StaticHashTable或StaticVocabularyTable生成table。
```python
keys_tensor = tf.constant(['a', 'b', 'c'])
vals_tensor = tf.constant([7, 8, 9])
input_tensor = tf.constant(['a', 'f'])
init = tf.lookup.KeyValueTensorInitializer(keys_tensor, vals_tensor)
table = tf.lookup.StaticHashTable(
    init,
    default_value=-1)
table.lookup(input_tensor).numpy()
# array([ 7, -1], dtype=int32)
```
备注：
1. keys, vals可以式tensor，也可以是普通的list，在init处理过程中会进行convert_to_tensor
2. 检索过程中（table.lookup），输入的input_tensor必须是tensor，因为在检索过程中会进行type的检验
    - 如果声明init的时候使用python array，同时没有显式的声明dtype的时候，模型的dtype是tf.string


# 3. 从文件内容生成table
当你有一个有<key, val>格式组成文件的时候，也可以通过tf自带的api将文件里的内容直接生成可以检索的table数据。
```python
f = tempfile.NamedTemporaryFile(delete=False)
content='\n'.join(["emerson 10", "lake 20|", "palmer 30",])
f.file.write(content.encode('utf-8'))
f.file.close()

init1= tf.lookup.TextFileInitializer(
   filename=f.name,
   key_dtype=tf.string,
   key_index=0,
   value_dtype=tf.string,
   value_index=1,
   delimiter=" ")

table1 = tf.lookup.StaticHashTable(init1, default_value='')
table1.lookup(tf.constant(['palmer','lake','tarkus'])).numpy()
# array([b'30', b'20|', b''], dtype=object)
```

# 4. 生成可变table
上述api都需要通过预先提供的数据生成一个固定的table，没有遇到过的key，只能通过default_value进行取值。
这种table相对比较死板，mutable_table解决的就是这种问题，可以生成一个能够随时增加元素的table，使用更便捷，更接近原始的python-array。
```python
table = tf.lookup.experimental.MutableHashTable(key_dtype=tf.string,
                                                value_dtype=tf.int64,
                                                default_value=-1)

keys_tensor = tf.constant(['a', 'b', 'c'])
vals_tensor = tf.constant([7, 8, 9], dtype=tf.int64)
input_tensor = tf.constant(['a', 'f'])
table.insert(keys_tensor, vals_tensor)
table.lookup(input_tensor).numpy()
# array([ 7, -1])   注意这里的输出，'f'因为检索不到对应的值，只能返回-1的默认值

new_keys = tf.constant(['d', 'e', 'f'])
new_vals = tf.constant([1,2,3], dtype=tf.int64)
table.insert(new_keys, new_vals)
table.lookup(input_tensor).numpy()
# array([7, 3])     增加了元素'f'之后，可以正常的检索到对应的val了

table.remove(tf.constant(['a']))
table.lookup(input_tensor).numpy()
# array([-1,  3])   删除了元素'a'之后，因为检索不到'a'的值，只能返回-1的默认值
```