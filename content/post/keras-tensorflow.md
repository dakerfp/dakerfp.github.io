+++
title = "Interfacing Keras with tensorflow"
date = "2017-09-17"
tags = ["keras", "tensorflow", "python", "machine learning"]
+++

Keras is great for machine learning and describing models!
However, sometimes it lacks the flexibility power that Tensorflow has.
The great news is that you can interface while using Tensorflow as a backend
for Keras. 

You may for instance, pre-process your data before feeding into
a Keras learning model, to use GPUs capabilities with little programming costs.
The following example normalizes the channels of a image before training, then
it uses the transformed tensor as input to the model.

```python
# image channel normalization
x = tf.placeholder(tf.float32, shape=(None, width, width, depth))

xmax = tf.reduce_max(x, axis=(1,2))
xmin = tf.reduce_min(x, axis=(1,2))
minmax = xmax - xmin
xnorm = (x - xmin) / minmax

# Keras model
model = models.Sequential()
model.add(layers.InputLayer(input_tensor=xnorm))
model.add(layers.Dense(64, activation='sigmoid'))
model.add(layers.Dense(16, activation='sigmoid'))
model.add(layers.Dense(2, activation='sigmoid'))

# ... regular Keras code from now on
```
