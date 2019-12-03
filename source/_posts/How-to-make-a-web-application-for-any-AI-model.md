---
title: How to make a web application for any AI model
date: 2019-12-03 11:15:16
author: Himanshu Garg
category: AI
tags: 
- Image-Classification
- Keras
---

Yes! you read the title right. So, today in this post I’ll show you how to setup a basic image-classifier in the form of a web application.

## Basic requirements

Before we getting dive more into it, I am listing down the basic ingredients which are required to make a web application in python.

* Flask

* Flask Bootstrap

Please note that, I am not showing about how to create an AI classifier model, so make sure you have your classifier already before seeking into this post, if not then you can download a pre-trained model.

Let’s get started !

## Installation

You need to have installed the above mentioned libraries. You can easily install them by using __pip__.
```
    pip install flask
    pip install flask-bootstrap
```
## Getting started

So, firstly we arrange our files and folders in the below shown order.

![Figure 1: Showing basic folder structure for the project](https://cdn-images-1.medium.com/max/2000/1*UcQSsTBtUdYL4x-HffjTNg.png)

You can change the main folder name *image-classifier (In my case)* to any other name as you like.

So, firstly we will write a basic flask app structure in `__init__.py`. This file can be found inside the classifier. Inside the `__init__.py` file write the below code
```
    from flask import Flask
    from flask_bootstrap import Bootstrap
    from classifier import routes

    app = Flask(__name__) ## defining our flask application
    Bootstrap(app) ## giving a nice bootstrap touch to our application
```
Now, writing our run.py file. You can find this run.py file inside the main folder. Open the file and write the below code
```
    from classifier import app as  application
    application.config.from_pyfile('config/config.py')  

    if __name__ == "__main__":
        application.run(host='0.0.0.0', port=8000)  ## This tells our
          ##application will run on this host and on this port.
```
Now, we will create write a config.py file. This file contains the configuration for the application. The basic configuration we can put now is that, we can just put our application in DEBUG mode. So, open the config.py file and write the below line.
```
    import os
    from os.path import join, dirname, realpath

    DEBUG = True
    ## If True, then it refresh the server after making any changes to the code.
    UPLOAD_FOLDER = join(dirname(realpath(__file__)), 'uploaded_images/')
```
Make a **uploaded_images** folder inside the config folder, this folder contains the image which will be uploaded on the server via user.

## Writing a basic route for our flask app

We have completely setup our basic flask environment. Now, its time to write a very basic flask api for hello world. Open the routes.py inside the classifier folder. Add the below lines to it
```
    from classifier import app
    import flask

    @app.route('/testing')
    def testing():
        return "<h1>Hello world</h1>"
```
Here, the **route()** function of the Flask class is a decorator, which tells the application which URL should call the associated function.
```
    app.route(rule, options)
```
* The **rule** parameter represents URL binding with the function.

* The **options** is a list of parameters to be forwarded to the underlying Rule object.

and in the end we just returned a simple message using some HTML tags.

To run this code, follows below steps.
```
    cd path/to/main-folder
    python run.py
```
![Figure 2: After running the above command you should see above like messages](https://cdn-images-1.medium.com/max/2000/1*kJBAht4yhvPymR1dyV9Eow.png)

Just open the browser and type localhost:8000/testing. You should see a screen just like below

![Figure 3: This is the route which we wrote for “/testing”](https://cdn-images-1.medium.com/max/2000/1*vLAQroEaj-4mJwLDFe4Tlg.png)

## Lets start our main route

Lets make our template. For this, firstly makes a folder called **template** inside the **classifier** folder. Now inside the template, create a home.html file and paste the code from this [link](https://raw.githubusercontent.com/hghimanshu/Blog/master/image-classifier/classifier/templates/home.html) in it.

Now, we make our home url and our backend part !!. So we will rewrite our routes.py file
```
    @app.route('/home')
    def home():
        return render_template('home.html')  ## The template which we created above
```
Now, our image processing function will be like below
```
    from flask import render_template
    from werkzeug import secure_filename
    from classifier.backend.prediction import image_prediction

    @app.route("/fetchingImage", methods = ['POST'])
    def fetchingImage():
        if flask.request.method == 'POST'
            image = flask.request.files['image']
            image.save(app.config['UPLOAD_FOLDER'] + secure_filename(image.filename))
            full_img = app.config['UPLOAD_FOLDER'] + image.filename
            data = image_prediction(full_img)
            if len(data)==2:
                return render_template('prediction.html', results = data)
            else:
                return render_template('error.html', results = data)
```
Here, firstly we get the image from the form in the **home.html**, then save the image into the system and then fetch that file and send it to another function *image_prediction* for processing. Then, we simply render the response from the model to the webpage. If there is no error, then we display the **prediction.html** template or else, we render the **error.html**. Now, working on our *image_prediction* function.

For making the this, firstly create a folder named **backend** inside the *classifier* folder, then inside the **backend** folder, create a **prediction.py** file and write the below code into it.
```
    import keras
    import numpy as np
    import tensorflow as tf
    import cv2
    from keras.models import load_model
    from keras.applications.vgg19 import VGG19
    from keras.applications.vgg19 import decode_predictions

    
    def image_prediction(image):
        MODEL = VGG19()
        try:
            image = cv2.imread(image)
            image = cv2.resize(image, (224, 224))
            image = image.reshape((1, image.shape[0], image.shape[1], image.shape[2]))
            yhat = MODEL.predict(image)
            label = decode_predictions(yhat)
            label = label[0][0]
            label, conf = label[1], label[2]*100
            results = [label, conf]
        except Exception as e:
            results = "Please check the image."
        return results
```
Now, let’s make our **error.html** and **prediction.html** templates. These templates are also, saved inside the *templates* folder. So, you can get the code for both the templates from [here](https://github.com/hghimanshu/Blog/tree/master/image-classifier/classifier/templates).

Well the coding part is mostly done, now we will test our web application. Now open your console and run the below command
```
    python run.py
```
After this, it will firstly download the VGG19 pretrained weights *(if you are following my code.)*, then it will start the server. Now, open the browser and go to **localhost:8000/home**, you will see something like this

![Figure 4: Our classifier’s home page](https://cdn-images-1.medium.com/max/2454/1*D8SVbfL_3Mdj_00zhNFSuQ.png)

Now, click on browse to upload any image and click the **predict** button. You’ll see some message like this (based on your image)

![Figure 5: Our Image Classification result](https://cdn-images-1.medium.com/max/2000/1*IH_y0QAKul0HqqtZxboYrA.png)

If there is any some issue with your image, then below screen will appear

![Figure 6: Error message](https://cdn-images-1.medium.com/max/2000/1*ZHXZWF3g6hnKU7cSvhknuA.png)

## Conclusion

So, this is how we can make a very basic web application for our image classifier. You can also find the whole code from my [github](https://github.com/hghimanshu/Blog/tree/master/image-classifier).

## References

* [https://www.tutorialspoint.com/flask/index.htm](https://www.tutorialspoint.com/flask/index.htm)

* [https://www.w3schools.com/bootstrap/bootstrap_templates.asp](https://www.w3schools.com/bootstrap/bootstrap_templates.asp)

* [https://pythonprogramming.net/flask-send-file-tutorial/](https://pythonprogramming.net/flask-send-file-tutorial/)
