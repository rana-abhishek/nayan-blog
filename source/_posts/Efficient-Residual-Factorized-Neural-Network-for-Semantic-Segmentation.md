---
title: Efficient-Residual-Factorized-Neural-Network-for-Semantic-Segmentation
date: 2020-01-09 08:37:31
author: Anand Kummari
category: AI
tags: 
- Machine Learning
- Deep Learning
- Residual Networks
- Dilated Convolutions
- Semantic segmentation
- Autonomous driving
---

<!-- # Efficient Residual Factorized Neural Network for Semantic Segmentation -->

> This post explains a research paper ERFNET, a real time and accurate ConvNet for semantic segmentation and the underlying concepts.

## Semantic Segmentation

Semantic segmentation is the task of classifying each image pixel to a class label. It is a classification task but at pixel level instead of image level. The labels could include a person, car, flower, piece of furniture, etc. For example, in the below image, all the cars will have the same labels. However, one can differentiate between the same class objects, this is called instance segmentation. For example, in an image that has many cars, **instance segmentation** can differentiate between each car object.

<!-- ![Image 1](https://cdn-images-1.medium.com/max/3840/1*CUp00rA4mSlJ8CRflVvKcA.jpeg)*Image 1*
 -->
{% asset_img  image_1.jpeg Image 1 %}

Semantic segmentation has many applications in autonomous vehicles, Advanced Driver Assistance Systems (ADAS), robotics, self-driving cars because it is important to know the context in which the agent operates.

Convolutional Neural Networks (CNN), which initially designed for classification tasks, have impressive capabilities in solving complex segmentation tasks as well. **Residual layers** have created a new trend in ConvNets design. Their reformulation of the convolutional layers to avoid the degradation problem of deep architectures allowed neural networks to achieve very high accuracies with large amounts of layers.

Computation resources are key factors in self-driving and autonomous vehicles. Algorithms are not only required to operate reliably, but they are required to operate fast (real-time), fit in embedded devices due to space constraints (compactness), and have low power consumption to affect as minimum as possible the vehicle autonomy. Considering a reasonable amount of layers, enlarging the depth with more convolutions achieves only small gains in accuracy while significantly increasing the required computational resources.

### Residual Layer

The paper proposes a new architecture design that leverages skip connections and convolutions with 1D kernels. While the skip connections allow the convolutions to learn residual functions that facilitate training, the 1D factorized convolutions allow a significant reduction of the computational costs while retaining a similar accuracy compared to the 2D ones.

<!-- ![Residual Block](https://cdn-images-1.medium.com/max/2000/1*6WlIo8W1_Qc01hjWdZy-1Q.png)*Residual Block*
 -->
{% asset_img residual_block.png Residual Block %}

Residual blocks allow convolutional layers to learn the residual functions. For example, in the above image, x is the input vector and F(X)+x is the output vector of the y vector. F(X) is the residual function to be learned. This residual formulation facilitates learning and significantly reduces the degradation problem present in architectures that stack a large number of layers.

### Non-bottleneck Residual Layers

There can be two instances of residual layer: the non-bottleneck design with two 3x3 convolutions as depicted in Fig. 1(a), and the bottleneck version as depicted in Fig. 1(b). Both versions have a similar number of parameters and almost equivalent accuracy. However, the bottleneck requires less computational resources and these scale in a more economical way as depth increases. Hence, the bottleneck design has been commonly adopted in state-of-the-art networks. However, it has been reported that non-bottleneck ResNets gain more accuracy from increased depth than the bottleneck versions, which indicates that they are not entirely equivalent and that the bottleneck design still suffers from the degradation problem

<!-- ![Figure 1](https://cdn-images-1.medium.com/max/2000/1*gu-iuZvJS1-8w-UlWGXmww.png)*Figure 1*
 -->
{% asset_img figure_1.png Figure 1 %}

The paper proposed a new implementation of the residual layer that decomposes 2D convolution into a pair of 1D convolutions to accelerate and reduce the parameters of the original non-bottleneck layer. We refer to this proposed module as “non-bottleneck-1D” (non-bt-1D), which is depicted in Fig. 1(c). This module is faster (as in computation time) and has fewer parameters than the bottleneck design while keeping a learning capacity and accuracy equivalent to the non-bottleneck one.

### Dilated Convolutions

Dilated convolutions are convolutions applied to input images with gaps. The standard convolution is 1-Dilated convolution. Dilated convolutions other than standard convolutions increase the receptive field of the network. Dilated convolutions are more effective in terms of computational cost and parameters than the convolutions with larger kernel size. The paper proposes a network that uses dilated convolution.

<!-- ![standard convolution](https://cdn-images-1.medium.com/max/2000/1*aIPu6hDHHWFatmOCYP9YPg.gif)*standard convolution*
 -->
<!-- ![dilated convolution](https://cdn-images-1.medium.com/max/2000/1*wz4x8BcAOFBPNL6nX4tx-g.gif)*dilated convolution*
 -->

{% asset_img standard_conv.gif Standard Convolution %}
{% asset_img dilated_conv.gif Dilated Convolution %}


### Network Architecture

The paper presents an encoder-decoder architecture for semantic segmentation. The encoder segment produces downsampled feature maps and the decoder segments upsample the features to match input image resolution. Full network architecture is given in Figure 2
<!-- 
![architecture with feature maps](https://cdn-images-1.medium.com/max/2000/1*YMWnwx78KluFYgcV5KGB0g.png)*architecture with feature maps*

![Figure 2](https://cdn-images-1.medium.com/max/2000/1*9Bbsq9_xHHImVSqtFkJrtw.png)*Figure 2*
 -->

{% asset_img image_2.png Architecture with feature maps %}
{% asset_img figure_2.png figure 2 %}


### References

1. [https://pixabay.com/photos/traffic-locomotion-roadway-mobility-3612474/](https://pixabay.com/photos/traffic-locomotion-roadway-mobility-3612474/)

2. ERFNet: Efficient Residual Factorized ConvNet for Real-time Semantic Segmentation.

3. [https://cdn-images-1.medium.com/max/800/1\*wz4x8BcAOFBPNL6nX4tx-g.gif](https://cdn-images-1.medium.com/max/800/1*aIPu6hDHHWFatmOCYP9YPg.gif)

4. [https://miro.medium.com/max/395/0\*3cTXIemm0k3Sbask.gif](https://miro.medium.com/max/395/0*3cTXIemm0k3Sbask.gif)


