---
title: License plate recognition using Attention based OCR
date: 2019-11-28 12:06:56
author: Piyush Jain
category: AI
tags:
- Machine Learning
- Deep Learning
- OCR
- AOCR
- License Plates
- License Plates Recognition
---



## Introduction

If you clicked on this post title then it is certain that you are working on some kind of License plate recognition task and working on a specific kind of License Plate, if that’s the case you have landed in right post. In this post i will explain how to train Attention based OCR (AOCR) model for a specific License Plate.

I will first share some brief information of AOCR model followed by steps which will help you train the model and after that we will use the trained model to test its performance.
## Attention OCR Model architecture

First of all the source code of this model is available on this [Tensorflow](https://github.com/tensorflow/models/tree/master/research/attention_ocr) Github repository. I will suggest you to try [this](https://github.com/emedvedev/attention-ocr) repository if you want/can modify code.

{% asset_img p1.png %} <center>Figure 1.  AOCR model architecture</center>

Source: [https://arxiv.org/pdf/1609.04938v2.pdf](https://arxiv.org/pdf/1609.04938v2.pdf)

Now the model structure, it has 7 Convolutional layer and 3 bi-directional Long short-term memory (LSTM) layers followed by a Seq2Seq model which translates image features to characters(act as a decoder). Convolutional layers can be seen in the below image.

{% asset_img p2.png %} <center>Figure 2. CNN architecture </center>

<br>

## How to train this model?

First of all use this pip command to install aocr on your system.

    pip install aocr

Now you need dataset of License Plates images with its annotation. Annotation file should have file path and its label in a text file.

    datasets/image1.jpg label1
    datasets/image2.jpg label2

After the dataset is ready along with annotation file you have to run this command to create tfrecords file for training of AOCR model. Separate some images for testing and create separate tfrecords file using test annotation file which contains paths of test images and corresponding labels.

    aocr dataset /path/to/training_annotation.txt path/to/training.tfrecords

    aocr dataset /path/to/testing_annotation.txt path/to/testing.tfrecords

The above command need annotation file path as input and creates tfrecords file on the given path i.e. last argument of above command. Now just run the below command to start training procedure.

By default maximum width is set to 160 and maximum height is set to 60. If your images has width or height more than the default maximum then model will not use those images for training. Either you resize all your images or you can set maximum width and height just make sure all the images are below those values.

Default checkpoint directory is ‘./checkpoints’ and it will create this where you will execute below command(You can set this too). Just make sure when you test you are giving correct checkpoint path, width and height.

Maximum prediction length is 8 by default and again you can change it according to your License plates. Default epoch is 1000 change it according to quantity of your dataset if it is small run it for default value otherwise you can set it to 500.

    aocr train /path/to/training.tfrecords --max-width 200 --max-height 100 --model-dir /path/to/checkpoint --max-prediction 6 --num-epoch 500
## Test the model

Once the training procedure is finished use this command to test the model. Just make sure checkpoint directory is created when the training starts.

    aocr test /path/to/testing.tfrecords --visualize --max-width 200 --max-height 100 --model-dir /path/to/checkpoint --max-prediction 6 --output-dir ./results

Now you can see all the prediction inside result folder. For each file there will be one folder which will contain the GIF which will have attention mapped on the images and a text file which will have prediction in the first line and label in the second line. Prediction is placed on the folder name too by default.

{% asset_img p3.png %} <center>Figure 3.  Result folder directory</center>
<br>
{% asset_img p4.png %} <center>Figure 4.  Text file which contains prediction and label</center>
<br>
{% asset_img p5.gif %} <center>Gif 1.  GIF with attention map</center>
<br>

## Conclusion

If you have successfully trained and test the model then you can skip this part. If you have all the training images in one folder and their labels are in the filename itself then you can run this simple script to train your model.

### Note: 
In case of same label for different images filename will be label_1.extension, label_2.extension etc.

Execute this script using this command “python3 Train_AOCR.py -d /home/some/path/”

```
import cv2
import os
import shutil
import sys
from pathlib import Path
import optparse

#python3 Train_AOCR.py -d /DIR_PATH/

# Give checkpoint path, steps per checkpoints and number of epoch in line 61

#give images max width and height here
dim = (210, 70)

parser = optparse.OptionParser()
parser.add_option('-d', '--dir_path',
    action="store", dest="dirpath",
    help="Enter test image directory", default="Empty")

parser.add_option('-i', '--image',
    action="store", dest="image",
    help="Input image", default="Empty")

options, args = parser.parse_args()

if os.path.exists("/home/username/path/annotations-training.txt"):
  os.remove("/home/username/path/annotations-training.txt")

if os.path.exists("/home/username/path/train.tfrecords"):
  os.remove("/home/username/path/train.tfrecords")



f_veh = open('/home/username/path/annotations-training.txt', 'w+')

if options.dirpath != 'Empty':
	for filename in os.listdir(options.dirpath):
	
		name, ext = os.path.splitext(filename)
		name = name.split('_')
		img = cv2.imread(options.dirpath+filename)
		img = cv2.resize(img, dim, interpolation = cv2.INTER_AREA)	
		cv2.imwrite(options.dirpath+filename,img)
		#os.rename(options.dirpath+filename,options.dirpath+temp[0]+ext)
		if ext in ['.png', '.jpg','.jpeg']:
			f_veh.write(options.dirpath+filename+ ' '+name[0]+ '\n')			
				
comm = 'aocr dataset /home/username/path/annotations-training.txt /home/username/path/train.tfrecords'
comm1 = 'aocr train /home/username/path/train.tfrecords --model-dir /home/username/path/checkpoints --max-height 70 --max-width 210 --max-prediction 6 --num-epoch 1000' 
os.system(comm)
os.system(comm1)
```

Now you can test the model on a test set using the above code only same format goes as for the training set and its label.

You just need to run below code using this command “python3 Run_AOCR.py -d /home/some/test_set_path/”

```
import cv2
import os
import shutil
import sys
from pathlib import Path
import optparse

#python3 Run_AOCR.py -d /DIR_PATH/

# Give checkpoint path, steps per checkpoints and number of epoch in line 61

#give images max width and height here
dim = (210, 70)

parser = optparse.OptionParser()
parser.add_option('-d', '--dir_path',
    action="store", dest="dirpath",
    help="Enter test image directory", default="Empty")

parser.add_option('-i', '--image',
    action="store", dest="image",
    help="Input image", default="Empty")

options, args = parser.parse_args()


p = Path("/home/username/path/results")
if p.is_dir():
	shutil.rmtree('/home/username/path/results')	

if os.path.exists("/home/username/path/annotations-testing.txt"):
  os.remove("/home/username/path/annotations-testing.txt")

if os.path.exists("/home/username/path/test.tfrecords"):
  os.remove("/home/username/path/test.tfrecords")



f_veh = open('/home/username/path/annotations-testing.txt', 'w+')

if options.dirpath != 'Empty':
	for filename in os.listdir(options.dirpath):
	
		name, ext = os.path.splitext(filename)
		name = name.split('_')
		img = cv2.imread(options.dirpath+filename)
		img = cv2.resize(img, dim, interpolation = cv2.INTER_AREA)	
		cv2.imwrite(options.dirpath+filename,img)
		#os.rename(options.dirpath+filename,options.dirpath+temp[0]+ext)
		if ext in ['.png', '.jpg','.jpeg']:
			f_veh.write(options.dirpath+filename+ ' '+name[0]+ '\n')			
				
comm = 'aocr dataset /home/username/path/annotations-testing.txt /home/username/path/test.tfrecords'
comm1 = 'aocr test --visualize /home/username/path/test.tfrecords --model-dir /home/username/path/checkpoints --max-height 70 --max-width 210 --max-prediction 6 --output-dir ./results' 
os.system(comm)
os.system(comm1)
```

After running this script you can find all the prediction in the output directory.
