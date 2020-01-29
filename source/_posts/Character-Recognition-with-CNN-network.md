---
title: Character-Recognition-with-CNN-network
date: 2020-01-29 11:47:04
author: Piyush Jain
category: AI
tags:
- Machine Learning
- Deep Learning
- OCR
- Character Recognition
---


## Introduction

This post talks about a simple Convolution Neural Network (CNN) which is used to
recognize characters i.e. Numeric and Alphabet. We have total 10 Numeric and 26
Alphabets that sums up the total number of classes in our network to 36. In
order to get characters from the License Plates we first need to use some kind
of License Plate detector which is followed by a Character segmentation method
in order to extract character from the License Plates (LP).

## Architecture of model

We have used very familiar CNN network for OCR, usually CNN consists of some
Convolution layers(All Convolution layers are followed by max pooling layers)
and fully connected layers.

We already know much about Convolution layers so i am gonna talk about max
pooling and fully connected layers here.

**Pooling** **layers** section would reduce the number of parameters when the
images are too large. Spatial pooling also called sub-sampling or down-sampling
which reduces the dimensionality of each map but retains important information.
Spatial pooling can be of different types:

* Max Pooling
* Average Pooling
* Sum Pooling

Max pooling takes the largest element from the rectified feature map. Taking the
largest element could also take the average pooling. Sum of all elements in the
feature map call as sum pooling.

The layer we call as **Fully Connected Layer (FC) layer**, we flattened our
matrix into vector and feed it into a fully connected layer like a neural
network. After the last max pooling layer there will be a sequence of FC layers.
Finally we will apply an activation function such as softmax or sigmoid to
classify the outputs between classes.

Model configuration is given below:

Total layer : 14

1.  Convolution with 64 different filters in size of (3x3)
2.  Max Pooling by 2
    * [ReLU](https://www.tensorflow.org/api_docs/python/tf/nn/relu) activation function
    * Batch Normalization
3.  Convolution with 128 different filters in size of (3x3)
4.  Max Pooling by 2
    * [ReLU](https://www.tensorflow.org/api_docs/python/tf/nn/relu) activation function
    * Batch Normalization
5.  Convolution with 256 different filters in size of (5x5)
6.  Max Pooling by 2
    * [ReLU](https://www.tensorflow.org/api_docs/python/tf/nn/relu) activation function 
    * Batch Normalization
7.  Convolution with 512 different filters in size of (5x5)
8.  Max Pooling by 2
    * [ReLU](https://www.tensorflow.org/api_docs/python/tf/nn/relu) activation function 
    * Batch Normalization
9.  Flattening the 3-D output of the last convolving operations.
10.  Fully Connected Layer with 128 units
11.  Fully Connected Layer with 256 units
12.  Fully Connected Layer with 512 units
13.  Fully Connected Layer with 1024 units
14.  Fully Connected Layer with 36 units (number of classes)
{% asset_img p1.png %} <center>Figure 1.  Architecture of model</center>


## Placeholders

Defining a placeholder in tensorflow is very common. When we want to declare our
input and output without initialization this method comes very useful. You can
use them during training of model by feeding them with training data and labels.
```
def create_placeholders(n_H0, n_W0, n_C0, n_y):
    X = tf.placeholder(tf.float32, shape = (None, n_H0, n_W0, n_C0), name='X')
    Y = tf.placeholder(tf.float32, shape = (None, n_y), name = 'Y')
    keep_prob = tf.placeholder(tf.float32, name="keep_prob")

    return X,Y,keep_prob

# X_train contains training data with shape (batch_size,height,widht,channel)

# Y_train contains labels of training data with shape (batch_size,num_classes,1)
m, n_H0, n_W0, n_C0 = X_train.shape
n_y = Y_train.shape[1]
X, Y, keep_prob = create_placeholders(n_H0, n_W0, n_C0, n_y)

```

<br> 

Once you have defined your model architecture you now need to define cost and
optimizer for your model which is defined in the next section.

## Cost function and optimizer

Cost function gives degree of error between predicted and expected values and
after that it represent it in form of a real number. Whereas optimizer update
the weight parameters to minimize the cost function.

Finally, you’ll define cost, optimizer, and accuracy. The [tf.reduce_mean](https://www.tensorflow.org/api_docs/python/tf/math/reduce_mean) 
takes an input tensor to reduce, and the input tensor is the results of  certain
loss functions between predicted results and ground truths. We have to measure
loss over 36 classes, [tf.nn.softmax_cross_entropy_with_logis](https://www.tensorflow.org/api_docs/python/tf/nn/softmax_cross_entropy_with_logits)  function is
used. 

When training the network, what you want is minimize  the cost by applying a
algorithm of your choice. It could be [SGD](https://www.tensorflow.org/api_docs/python/tf/train/GradientDescentOptimizer),[AdamOptimizer](https://www.tensorflow.org/api_docs/python/tf/train/AdamOptimizer)[,AdagradOptimizer](https://www.tensorflow.org/api_docs/python/tf/train/AdagradOptimizer) or 
something else. You have to study how each algorithm works to choose what 
to use, but AdamOptimizer works fine for most cases in general.

Please find cost and optimizer sample below:
```
learning_rate = 0.001
# X is placeholder you defined in previous section
Z3 = forward_propagation(X, keep_prob)

# Z3 has the model structure
# Loss and Optimizer
cost = tf.reduce_mean(tf.nn.softmax_cross_entropy_with_logits(logits=Z3, labels=Y))
optimizer = tf.train.AdamOptimizer(learning_rate=learning_rate).minimize(cost)

# Accuracy
y_pred = tf.nn.softmax(Z3)
y_pred_class = tf.argmax(y_pred, axis = 1)
y_true_class = tf.argmax(Y, axis = 1)

correct_prediction = tf.equal(y_pred_class, y_true_class)
accuracy = tf.reduce_mean(tf.cast(correct_prediction, tf.float32))
```

## Conclusion

So in this post i have explained basic steps to train simple CNN network for any
classification task i.e. OCR in this particular post. I have given all the steps
except the training part for that you just need to use session of tensorflow
while feeding image data and labels for those images to placeholder you have
created to the session.run function.

<br> 

<br> 
