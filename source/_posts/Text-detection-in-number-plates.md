---
title: Text detection in number plates
date: 2019-11-26 15:21:29
author: Akshay Bajpai
tags: 
---

# Text detection in number plates
One of the vital modules in the optical character recognition(OCR) pipeline is text detectionand segmentation which is also called text localization. In this post, we will apply variedpreprocessing techniques to the input image and find out how to localize text in theenhanced image, so that we can feed the segments to our text recognition network.

## Image Preprocessing
Sometimes images can be distorted, noisy and other problems that can scale back the OCRaccuracy. To make a better OCR pipeline, we need to do some image preprocessing.

* Grayscale the image: Generally you will get an image which is having 3channels(color images), we need to convert this image into a grayscale form whichcontains only one channel. We can also process images with three channels but itonly increases the complexity of the model and increases the processing time.OpenCV provides a built-in function that can do it for you.


```
import cv2
grayscale_image = cv2.cvtColor(image, cv2.COLOR_BRG2GRAY)
```
Or you can convert the image to grayscale while reading the image.
```
#opencv reads image in BGR format
graysacle_image = cv2.imread(image_path, cv2.IMREAD_GRAYSCALE)
```

* Noise reduction: Images come with various types of noises. OpenCV provides a lot ofnoise reduction function. I am using the Non-local Means Denoising algorithm.

```
denoised_image = cv2.fastNlMeansDenoising(grayscale_img, None, 10, 7, 21)
```

{% asset_img Figure_1.png Denoising %}

* Contrast adjustment: Sometimes we have low contrast images. This makes it difficultto separate text from the image background. We need high contrast text images forthe localization process. We can increase image contrast using Contrast LimitedAdaptive Histogram Equalization (CLAHE) among many other contrast enhancementmethods provided by skimage.

```
from  skimage import exposure
contrast_enhanced_image = exposure.equalize_adapthist(denoised, clip_limit=0.03)
```

{% asset_img Figure_2.png Contrast Adjustment %}

So now we are done with image preprocessing let us move on to the second part, textlocalization.

## Text Localization

In this part, we will see how to detect a large number of text region candidates andprogressively removes those less likely to contain text. Using the MSER feature descriptor tofind text candidates in the image. It works well for text because the consistent color and highcontrast of text lead to stable intensity profiles.

```
#constructor for MSER detector
mser = cv2.MSER_create()
regions, mser_bboxes = mser.detectRegions(contrast_enhance_image)
```

Along with the text MSER picked up many other stable regions that are not text. Now, thegeometric properties of text can be used to filter out non-text regions using simplethresholds.

Before moving on with the filtering process, let's write some functions to display the results ina comprehensible manner.

```
import numpy as np
import matplotlib.pyplot as plt

#display images
def pltShow(*images):
    #count number of images to show
    count = len(images)
    #three images per columnn
    Row = np.ceil(count / 3.)
    for i in range(count):
        plt.subplot(nRow, 3, i+1)
        if len(images[i][0], cmap=’gray’)
            plt.imshow(images[i][0], cmap=’gray’)
        else:
            plt.imshow(images[i][0])
        #remove x-y axis from subplots
        plt.xticks([])
        plt.yticks([])
        plt.title(images[i][1])
    plt.show()

#color each MSER region in image
def colorRegion(image_like_arr, region):
    image_like_arr[region[:, 1], region[:, 0], 0] = np.random.randint(low=100, high=256)
    image_like_arr[region[:, 1], region[:, 0], 1] = np.random.randint(low=100, high=256)
    image_like_arr[region[:, 1], region[:, 0], 2] = np.random.randint(low=100, high=256)

    return image
```

The geometric properties we are going to use to discriminate between text and non-textregion are:

* Region area
* Region perimeter
* Aspect ratio
* Occupancy
* Compactness

We will apply simple thresholds over these parameters to eliminate non-text regions. Firstlet’s write method to compute these parameters.

```
#values for the parameters
AREA_LIM = 2.0e-4
PERIMETER_LIM = 1e-4
ASPECT_RATIO_LIM = 5.0
OCCUPATION_LIM = (0.23, 0.90)
COMPACTNESS_LIM = (3e-3, 1e-1)

def getRegionShape(self, region): 
    return (max(region[:, 1]) - min(region[:, 1]), max(region[:, 0]) - min(region[:, 0]))
    
#compute area
def getRegionArea(region):
    return len(list(region))

#compute perimeter
def getRegionPerimeter(image, region):
    #get top-left coordinate, width and height of the box enclosing the region
    x, y, w, h = cv2.boundingRect(region)
    return len(np.where(image[y:y+h, x:x+w] != 0)[0]))
    
#compute aspect ratio
def getAspectRatio(region):    
    return (1.0 * max(getRegionShape(region))) / (min(getRegionShape(region)) + 1e-4)

#compute area occupied by the region area in the shape
def getOccupyRate(region):
    return (1.0 * getRegionArea(region)) / (getRegionShape(region)[0] *  \getRegionShape(region)[1] + 1.0e-10)
    
#compute compactness of the regio
ndef getCompactness(region):    
    return (1.0 * getRegionArea(region)) / (1.0 * getRegionPerimeter(region) ** 2)
```

Now apply these methods to filter out text regions  as follows:

```
#total number of MSER regions
n1 = len(regions)
bboxes=[]
for i, region in enumerate(regions):
    self.colorRegion(res, region)
    if self.getRegionArea(region) > self.grayImg.shape[0] * self.grayImg.shape[1] * AREA_LIM:
   	    #number of regions meeting the area criteria
    n2 += 1
   		 self.colorRegion(res2, region)

    if self.getRegionPerimeter(region) > 2 * (self.grayImg.shape[0] + \
        self.grayImg.shape[1]) * PERIMETER_LIM:
   		#number of regions meeting the perimeter criteria
        n3 += 1
   		self.colorRegion(res3, region)
			 
        if self.getAspectRatio(region) < ASPECT_RATIO_LIM:
   				#number of regions meeting the aspect ratio criteria 
                n4 += 1
   				self.colorRegion(res4, region)

   				if (self.getOccupyRate(region) > OCCUPATION_LIM[0]) and \ (self.getOccupyRate(region) < OCCUPATION_LIM[1]):
   					n5 += 1
   					self.colorRegion(res5, region)

   					if (self.getCompactness(region) > \COMPACTNESS_LIM[0]) and \(self.getCompactness(region) < \COMPACTNESS_LIM[1]):
   						#final number of regions left 
                        n6 += 1
   						self.colorRegion(res6, region)
                        bboxes.append(mser_bboxes[i])
```

After eliminating non-text regions, I draw bounding boxes on the remaining regions andvoila, we have successfully detected and segmented the characters on the number plate.
Note: Apply NMS to remove overlapping bounding boxes.

```
for bbox in bboxes:
   cv2.rectangle(img,(bbox[0]-1,bbox[1]-1),(bbox[0]+bbox[2]+1,box[1]+bbox[3]+1),(255,0,0), 1)
```

Enough coding. Let’s see some results.

```
pltShow("MSER Result Analysis", \
   	   (self.img, "Image"), \
   	   (self.cannyImg, "Canny"), \
   	   (res, "MSER,({} regions)".format(n1)), \
   	   (res2, "Area={},({} regions)".format(config.mser_areaLimit, n2)), \
   	   (res3, "Perimeter={},({} regions)".format(config.mser_perimeterLimit, n3)), \
   	   (res4, "Aspect Ratio={},({} regions)".format(config.mser_aspectRatioLimit, n4)), \
   	   (res5, "Occupation={},({} regions)".format(config.mser_occupationLimit, n5)), \
   	   (res6, "Compactness={},({} regions)".format(config.mser_compactnessLimit, n6)), \
   	   (boxRes, "Segmented Image") \
   	)
```

{% asset_img Figure_3.png MSER Result Analysis %}

## Conclusion

In this post, we covered the various image preprocessing techniques and learned about howto perform text localization on number plates.