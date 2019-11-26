---
title: Detecting Lanes using Deep Neural Networks
date: 2019-11-26 11:17:10
author: 
tags:
- Machine Learning
- Deep Learning
- Lane Detection
- Advance Driver Assistance
- Autonomous Driving
---

> This post explains how to use deep neural networks to detect highway lanes. Lane markings are the main static component on highways.**They instruct the vehicles to interactively and safely drive on the highways.** Lane detection is also an important task in autonomous driving, which provides localization information to the control of the car. It is also used in **ADAS(Advanced Driver Assistance System)**.

For the task of lane detection, we have two open-source datasets available. One is the Tusimple dataset and the other is the CULane dataset. Let’s have a brief look at one of the datasets.

## Tusimple Dataset

This dataset was released as part of the Tusimple Lane Detection Challenge. It contains 3626 video clips of 1 sec duration each. Each of these video clips contains 20 frames of which, the last frame is annotated. These videos were captured by mounting the cameras on a vehicle dashboard. You can download the dataset from [here](https://github.com/TuSimple/tusimple-benchmark/issues/3).

The directory structure looks like the figure below,
{% asset_img dir_structure.png Dataset directory structure %}

Each sub-directory contains 20 sequential images of which, the last frame is annotated. label_data_(date).json contains labels in JSON format for the last frame. Each line in the JSON file is a dictionary with key values…

**raw_file**: string type. the file path in the clip

**lanes**: it is a list of list of lanes. Each list corresponds to a lane and each element of the inner list is x-coordinate of ground truth lane.

**h_samples**: it is a list of height values corresponding to the lanes. Each element in this list is y-coordinate of ground truth lane

In this dataset, at most four lanes are annotated - the two ego lanes (two lane boundaries in which the vehicle is currently located) and the lanes to the right and left of ego lanes. All the lanes are annotated at an equal interval of height, therefore h_samples contain only one list whose elements correspond to y-coordinates for all lists in lanes. For a point in h_samples, if there is no lane at the location, its corresponding x-coordinate has -2. For example, a line in the JSON file looks like :

```
{
  "lanes": [
        [-2, -2, -2, -2, 632, 625, 617, 609, 601, 594, 586, 578, 570, 563, 555, 547, 539, 532, 524, 516, 508, 501, 493, 485, 477, 469, 462, 454, 446, 438, 431, 423, 415, 407, 400, 392, 384, 376, 369, 361, 353, 345, 338, 330, 322, 314, 307, 299],
        [-2, -2, -2, -2, 719, 734, 748, 762, 777, 791, 805, 820, 834, 848, 863, 877, 891, 906, 920, 934, 949, 963, 978, 992, 1006, 1021, 1035, 1049, 1064, 1078, 1092, 1107, 1121, 1135, 1150, 1164, 1178, 1193, 1207, 1221, 1236, 1250, 1265, -2, -2, -2, -2, -2],
        [-2, -2, -2, -2, -2, 532, 503, 474, 445, 416, 387, 358, 329, 300, 271, 241, 212, 183, 154, 125, 96, 67, 38, 9, -2, -2, -2, -2, -2, -2, -2, -2, -2, -2, -2, -2, -2, -2, -2, -2, -2, -2, -2, -2, -2, -2, -2, -2],
        [-2, -2, -2, 781, 822, 862, 903, 944, 984, 1025, 1066, 1107, 1147, 1188, 1229, 1269, -2, -2, -2, -2, -2, -2, -2, -2, -2, -2, -2, -2, -2, -2, -2, -2, -2, -2, -2, -2, -2, -2, -2, -2, -2, -2, -2, -2, -2, -2, -2, -2]
       ],
  "h_samples": [240, 250, 260, 270, 280, 290, 300, 310, 320, 330, 340, 350, 360, 370, 380, 390, 400, 410, 420, 430, 440, 450, 460, 470, 480, 490, 500, 510, 520, 530, 540, 550, 560, 570, 580, 590, 600, 610, 620, 630, 640, 650, 660, 670, 680, 690, 700, 710],
  "raw_file": "path_to_clip"
}
```

It says that there are four lanes in the image, and the first lane starts at (632,280), the second lane starts at (719,280), the third lane starts at (532,290) and the fourth lane starts at (781,270).

## DataSet Visualization

```
# import required packages
import json
import numpy as np
import cv2
import matplotlib.pyplot as plt
# read each line of json file
json_gt = [json.loads(line) for line in open('label_data.json')]
gt = json_gt[0]
gt_lanes = gt['lanes']
y_samples = gt['h_samples']
raw_file = gt['raw_file']
# see the image
img = cv2.imread(raw_file)
cv2.imshow('image',img)
cv2.WaitKey(0)
cv2.destroyAllWindows()
```

{% asset_img image_1.jpg Image 1. Raw image from the dataset %}

Now see the JSON points visualization on the image

```
gt_lanes_vis = [[(x, y) for (x, y) in zip(lane, y_samples)
                  if x >= 0] for lane in gt_lanes]
img_vis = img.copy()

for lane in gt_lanes_vis:
    cv2.polylines(img_vis, np.int32([lane]), isClosed=False,
                   color=(0,255,0), thickness=5)
```

{% asset_img image_2.jpg Image 2. Label visualications of image %}

Now, we have understood the dataset, but we can not pass the above image as a label for the neural network since **grayscale images with values ranging from zero to num_classes -1 are to be passed to the deep convolution neural network to outputs an image containing predicted lanes**. So, we need to generate label images for the JSON files. Label images can be generated using **OpenCV** by drawing lines passing through the points in the JSON file.

OpenCV has an inbuilt function to draw multiple lines through a set of points. OpenCV’s Polylines method can be used here. First, create a mask of all zeros with height and width equal to the raw file’s height and width using numpy. **The image size can be reduced to maintain lesser computations during training, but do not forget to maintain the same aspect ratio**.

## Generating Labels

The label should be a grayscale image. Generate one label for each clip from the JSON file. First, create a mask of black pixels with a shape similar to the raw_file image from the JSON file. Now, using OpenCV’s polylines method draw lines with different colors (each corresponding to each lane in lanes) on the mask image using the lanes and h_samples from the JSON file. From the three channeled mask image generate a gray scale mask image with values as class numbers. Likewise, create labels for all the images in the JSON file. You can resize the image and its label to a smaller size for lesser computations.

``
mask = np.zeros_like(img)
colors = [[255,0,0],[0,255,0],[0,0,255],[0,255,255]]
for i in range(len(gt_lanes_vis)):
    cv2.polylines(mask, np.int32([gt_lanes_vis[i]]), isClosed=False,color=colors[i], thickness=5)
# create grey-scale label image
label = np.zeros((720,1280),dtype = np.uint8)
for i in range(len(colors)):
   label[np.where((mask == colors[i]).all(axis = 2))] = i+1
```
{% asset_img image_3.jpg Image 3. Generated mask image %}

## Build and Train Model

Lane Detection is essentially an image segmentation problem. So I am using the **ERFNET model** for this task, which is efficient and fast. Originally ERFNET was proposed for semantic segmentation problems, but it can also be extended to other image segmentation problems. You can check out for its paper [here](https://ieeexplore.ieee.org/abstract/document/8063438). It is a CNN with Encoder, Decoder and dilated convolutions along with non-bottleneck residual layers. See Fig.1 for model architecture.

{% asset_img figure_1.png Figure 1. Model Architecture %}

Build and create an object of the model. Train it over the dataset created above, for a sufficient number of epochs with Binary Cross Entropy loss or custom loss function which minimizes the per-pixel error. For better memory usage, create a dataset generator and train the model over it. Generators remove the burden of loading all the images into memory (if your dataset is of large size, you should use a generator) which leads to eating up of all memory and the other processes can’t work properly. Fig 2 shows the layers of ERFNET with input and output dimensions.

{% asset_img figure_2.png Figure 2. Model layers with input and output shapes %}

## Evaluate Model

After training, get the model’s predictions using the code snippet below. I have implemented this in Pytorch. I use the color_lanes method to convert output images from the model (which are two channeled with values as class numbers) to three channeled images. im_seg is the final overlayed image shown in Image 4.

```
# using pytorch
import torch
from torchvision.transforms import ToTensor
def color_lanes(image, classes, i, color, HEIGHT, WIDTH):
    buffer_c1 = np.zeros((HEIGHT, WIDTH), dtype=np.uint8)
    buffer_c1[classes == i] = color[0]
    image[:, :, 0] += buffer_c1
    buffer_c2 = np.zeros((HEIGHT, WIDTH), dtype=np.uint8)
    buffer_c2[classes == i] = color[1]
    image[:, :, 1] += buffer_c2
    buffer_c3 = np.zeros((HEIGHT, WIDTH), dtype=np.uint8)
    buffer_c3[classes == i] = color[2]
    image[:, :, 2] += buffer_c3
    return image
img = cv2.imread('images/test.jpg') 
img = cv2.resize(img,(WIDTH, HEIGHT),interpolation = cv2.INETR_CUBIC)
op_transforms = transforms.Compose([transforms.ToTensor()])
device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')
im_tensor = torch.unsqueeze(op_transforms(img), dim=0)
im_tensor = im_tensor.to(device)
model = ERFNET(5)
model = model.to(device)
model = model.eval()
out = model(im_tensor)
out = out.max(dim=1)[1]
out_np = out.cpu().numpy()[0]
out_viz = np.zeros((HEIGHT, WIDTH, 3))
for i in range(1, NUM_LD_CLASSES):
    rand_c1 = random.randint(1, 255)
    rand_c2 = random.randint(1, 255)
    rand_c3 = random.randint(1, 255)
    out_viz = color_lanes(
            out_viz, out_np,
            i, (rand_c1, rand_c2, rand_c3), HEIGHT, WIDTH)
instance_im = out_viz.astype(np.uint8)
im_seg = cv2.addWeighted(img, 1, instance_im, 1, 0)
```

{% asset_img image_4.jpg Image 4. Final predicted image %}

Thanks for reading it…

## References

1. ERFNet: Efficient Residual Factorized ConvNet for Real-time Semantic 2. Segmentation.
2. Lane Detection and Classification using CNNs.
3. [https://www.mdpi.com/sensors/sensors-19-00503/article_deploy/html/images/sensors-19-00503-g004.png](https://www.mdpi.com/sensors/sensors-19-00503/article_deploy/html/images/sensors-19-00503-g004.png)

