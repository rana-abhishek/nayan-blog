---
title: Flask + PyMongo Integration
date: 2020-02-05 11:09:31
author: Himanshu Garg
tags:
---

{% asset_img maxresdefault.jpg %}

Flask + PyMongo Integration

In my college days I found quite difficulty regarding “How to integrate my application with the database”. May be most of us (mostly college students) still have the same problem. So today in this post I will show you how to integrate a Flask application with PyMongo.

What is **PyMongo**?

As according to its official site,“**PyMongo is a Python distribution containing tools for working with [MongoDB](http://www.mongodb.org), and is the recommended way to work with MongoDB from Python**”. The PyMongo is very easy to use and quite easy to integrate with Flask. For this you must have install MongoDb in your machine.

**Let’s get started.**

If you are new to flask then before diving more into it, I highly recommend to check out my [previous](https://medium.com/@hghimanshu81/how-to-represent-any-trained-model-in-the-form-of-a-web-application-e5af87d9731d) post in which I discussed about how to create your first application.

![Fig. 1 Folder structure for the project](https://cdn-images-1.medium.com/max/2000/1*Sf16kjWkKCSGNfR3mVRcUQ.png)

Firstly we will write some basic files for the project before heading towards the main backend logic. Firstly we write the ***run.py*** file and write the code to start our flask server

	from labeler import app as  application
    from labeler import config

    application.config.from_object(config)
    application.config.from_pyfile('config/config.py')

    if __name__ == "__main__":
        application.run(host='0.0.0.0', port=8000)

Now, write the **__init__.py**

	from flask import Flask
    from flask_bootstrap import Bootstrap

	app = Flask(__name__)
    Bootstrap(app)

    from labeler import routes

Now make a **settings.py** file inside the **config** folder and write the below code in it.

	import pymongo
    from pymongo import MongoClient

    ENV = "test"

    if ENV.lower() == "production":
        MONGO_DB_NAME = 'image_search'
        MONGO_DB_URL = 'localhost'

    else:
        MONGO_DB_NAME = 'image_search_local'
        MONGO_DB_URL = 'localhost'

    CLIENT = MongoClient()
    CLIENT = MongoClient(MONGO_DB_URL, 27017)
    DB = CLIENT[MONGO_DB_NAME]

In this script, we simple configure our mongo database and connect pymongo with the DB.

Now, its time to make a script for mongo queries. Lets create a **mongo.py** file inside **mongodb** folder and write the below lines in it.

	import pymongo
    from pymongo import MongoClient
    import sys
    from labeler.config.settings import DB

    
    class settingupDb:
        def __init__(self, query, coll_name):
            self.query = query
            self.coll_name = coll_name

        def constructDb(self):
            self.coll = DB[self.coll_name]
            return self.coll
       
        def insertsToDb(self,db,coll,query):
            self.post_id = coll.insert(self.query, check_keys=False)
            print('Data inserted for Object ID:: ',self.post_id)
       
        def updatesInfo(self, db, coll, query, newVal):
            self.query = query
            self.newVal = newVal
            self.updatedColl = coll.update_many(self.query, self.newVal)

       def fetchInfo(self, db, coll, query):
            self.results = coll.find(query)
            return self.results

       def aggregateQuery(self, db, coll, query_in_list):
           self.results = coll.aggregate(query_in_list)
           return self.results

    
    def insertData(query, collection):
        c_db = settingupDb(query, collection)
        coll = c_db.constructDb()
        c_db.insertsToDb(DB, coll, query)

    def fetchData(collection, query):
        c_db = settingupDb(query, collection)
        coll = c_db.constructDb()
        res = c_db.fetchInfo(DB, coll, query)
        return res

    def groupingData(collection, query):
        c_db = settingupDb(query, collection)
        coll = c_db.constructDb()
        res = c_db.aggregateQuery(DB, coll, query)
        return res

    def updateData(query, newVal, collection):
        c_db = settingupDb(query, collection)
        coll = c_db.constructDb()
        c_db.updatesInfo(DB, coll, query, newVal)

In this script, I created a class *settingupDb*, it basically sets up the db. Then defines some methods based on the queries. In this project we use some basic queries like :-

* Insert

* Find

* Update

* Aggregate

Will explain the use of these queries when we use them in the project.

Now lets make our **routes.py** file.

	from labeler import app
    import json
    import os
    from werkzeug import secure_filename
    import flask
    from flask import render_template
    from labeler.backend.handle_requests import 

    STATIC_FOLDER = os.path.dirname(os.path.abspath(__file__)) + '/static/'

Firstly we will import everything in the script. The last line is for the static folder where we server our media files.

Since our database is empty, so firstly we will write a **Data Insertion** endpoint. Now, write a **handle_requests.py** file inside the **backend** folder.

    from labeler.mongodb.mongo import fetchData, insertData

    
    COLL = "Image-Data"

    def isLabelInDb(label, image_path):
        query = {"label": label}
        res = fetchData(COLL, query)
        alreadyPresent = False
        
        if res.count() == 0:
            insert_q = {"label": label, "image_path": image_path}   

            insertData(insert_q, COLL)

        else:
            alreadyPresent = True
        return alreadyPresent

This function deals with the insertion and fetching part. The input is the label and the image. Firstly it checks the given label is already present in the database or not, if the label is not present then it inserts it into the database along with the image and if the label is already there in the database then it sets the “alreadyPresent” flag. Now according to the this logic we write our endpoint in the **routes.py** file.

    @app.route("/createLabels", methods=['GET', 'POST'])
    def createLabels():
        if flask.request.method == 'POST':
            image = flask.request.files['image']
            label = flask.request.form['label']
            image.save(STATIC_FOLDER + secure_filename(image.filename))
            
            alreadyPresent = isLabelInDb(label, image.filename)
            
            if alreadyPresent:
                message = "The label is already in the database. Try with other label"
                return render_template('error.html', message = message)
            else:
                message = "The label is successfully inserted to the database"
                return render_template('success.html', message = message)
        else:
            return render_template('createLabels.html')

This endpoint takes two input for a *POST* request. The “image” and “label”. It calls the function described above and it renders a template with a message.

Now we defines our “home” endpoint. In **routes.py** write the below code.

![Fig. 2. Page which creates the label and upload the associated image](https://cdn-images-1.medium.com/max/2000/1*xcCmqMUDBsx78ryCtf5NHg.png)

    @app.route('/home')
    def home():
        allImages = getAllImages()
        data = {}
        if len(list(allImages._CommandCursor__data)) != 0:
            for r in allImages:
                label = r['_id']
                images = r['image_path']
                data[label] = images
        return render_template('home.html', results=data)

This renders all the images with their function to the home page. Lets create a “getAllImages” function for this code. In the **handle_requests.py** write below code.

![Fig. 3. Home page of the application shows the available labels](https://cdn-images-1.medium.com/max/2074/1*a0ZuZwhq8SCJJmJKwhVXjg.png)

    def getAllImages():
        group_q = {"$group": {"_id": "$label", "image_path": {"$push": "$image_path"}}}
        project_q = {"$project": {"label": 1, "image_path":1}}
        pipeline = [group_q, project_q]
        res = groupingData(COLL, pipeline)
        
        return res

This function is basically grouping all the data based on the labels name present in the database and all image values associated with that are pushed into an array.

Now, we will write a function for fetching of image from database. In the **handle_requests.py** write the below code.

    def getRequiredImages(label):
        query = {"label": label}
        res = fetchData(COLL, query)
        totalImages = []
        if res.count() != 0:

            for i in res:
                image_name = i['image_path']
                totalImages.append(image_name)

        return totalImages

This function fetches all the images from the database for the given label name and returns them as a list. Now lets create its endpoint in the **routes.py**.

    @app.route('/fetchImages', methods=['POST'])
    def fetchImages():
        if flask.request.method == 'POST':
            label = flask.request.form['label']
            totalImages = getRequiredImages(label)
            if len(totalImages) == 0:
                message = "No image is present in the database with the label " + str(label)
                return render_template('error.html', message = message)
            else:
                data = [totalImages, label]
                return render_template('show_images.html', results=data)

This endpoint takes a label name as input and fetches its images from the database and renders the image on the template.

![Fig. 4. Fetching a particular label from the database](https://cdn-images-1.medium.com/max/2440/1*deLzuuUyotCHuYGIz3isfg.png)

Now the last type of operation left is **UPDATE**. For this write the below code in **handle_requests.py**.

    def updateInfo(image_path, curr_label, new_label):
        curr_label_q = {"label": curr_label}
        new_label_q = {"$set": {"label": new_label}}
        updateData(curr_label_q, new_label_q, COLL)

This function takes three input :-

* the image name

* the current label name

* the new label name

Now its corresponding endpoint in the **routes.py**

    @app.route('/updateLabel', methods=['POST'])
    def updateLabel():
        if flask.request.method == 'POST':
            image = flask.request.form['image']
            curr_value = flask.request.form['current_label']
            new_value = flask.request.form['new_label']
            updateInfo(image, curr_value, new_value)
            message = "Label is updated !!"
            return render_template('success.html', message=message)

This endpoint shows a message after successful updation of labels

![Fig. 5. Message showing that label is changed](https://cdn-images-1.medium.com/max/2000/1*4d_3cmuiPbHOUqW8TLfjgQ.png)

**Conclusion**

This is the very basic application which is made by integrating flask with pymongo. The code is available on [github.](https://github.com/hghimanshu/Blog/tree/master/image-labeler) The necessary templates are also uploaded there.
