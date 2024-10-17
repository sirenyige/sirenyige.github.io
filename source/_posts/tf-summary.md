---
title: tf.summary
date: 2020-06-11 00:00:16
categories:
- 机器学习
tags:
- TensorFlow
---

# tf.summary

Summary是对网络中Tensor取值进行监测的一种`Operation`。这些操作在图中是“外围”操作，不影响数据流本身。**Summary本身也是一个op**。

<!-- more -->

## 一、Tensorboard的数据形式

Tensorboard可以记录与展示以下数据形式：

1. 标量Scalars
2. 图片Images
3. 音频Audio
4. 计算图Graph
5. 数据分布Distribution
6. 直方图Histograms
7. 嵌入向量Embeddings

## 二、记录方法

### 1、tf.summary.scalar

用来显示标量信息

```python
tf.summary.scalar(name, tensor, collections=None, family=None)
```

### 2、tf.summary.image

输出带图像的probuf

```python
tf.summary.image(name, tensor, max_outputs=3, collections=None, family=None)
```

### 3、tf.summary.audio

展示训练过程中记录的音频

```python
tf.summary.audio(name, tensor, sample_rate, max_outputs=3, collections=None, family=None)
```

### 4、tf.summary.histogram

用来显示直方图信息

```python
tf.summary.histogram(name, values, collections=None, family=None)
```

## 三、输出方法

### 1、选取要展示的summary项

- tf.summary.merge_all  展示所有的summary项
- tf.summary.merge([summary_op1, summary_op2...]) 展示指定summary项

### 2、定义 tf.summary.FileWriter

```python
tf.summary.FileWriter(logdir, 
		              graph=None,
           			  max_queue=10,
              		  flush_secs=120,
              		  graph_def=None,
              		  filename_suffix=None,
              		  session=None)
```

### 3、向FileWriter写入summary

```python
FileWriter.add_summary(summary, global_step=None)
```
### 4、示例代码
```python
#ops
loss = ...
tf.summary.scalar("loss", loss)
merged_summary = tf.summary.merge_all()

init = tf.global_variable_initializer()
with tf.Session() as sess:
  writer = tf.summary.FileWriter(your_dir, sess.graph)
  sess.run(init)
  for i in xrange(100):
    _,summary = sess.run([train_op,merged_summary], feed_dict)
    writer.add_summary(summary, i)
```