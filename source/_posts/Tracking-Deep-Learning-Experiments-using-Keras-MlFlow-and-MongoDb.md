
# Tracking Deep Learning Experiments using Keras, MlFlow and MongoDb


{% asset_img banner.jpeg Banner %}
It is late 2019 and ***Deep Learning*** is not a buzzword anymore. It is significantly used in the technology industry to attain feats of wonders which traditional machine learning and logic based techniques would take a longer time to achieve.

The main ingredient in Deep Learning are ***Neural Networks***, which are computation units called neurons, connected in a specific fashion to perform the task of learning and understanding data. When these networks become extremely deep and sophisticated, they are referred to as Deep Neural Networks and thus Deep Learning is performed.

Neural Networks are so called because they are speculated to be imitating the human brain in some manner. Though it is not entirely true, but the learning mechanism is mostly similar in nature.

A human brain learns about an object or concept when it visually experiences it for a longer amount of time. Similar to that, a neural network learns about objects and what they actually represent when it is fed with a large amount of data.

For example, let us consider the ***LeNet architecture*** . It is a small two layered CNN (Convolution Neural Network). Convolution Neural Networks are a special kind of neural network where the mathematical computation being done at every layer are convolution operations.

If enough images of a certain kind are fed to the ***LeNet*** architecture, it starts to understand and classify those images better.

That was a simple introduction to what neural networks are and how they behave.

In this article we will be mostly looking into three main frameworks which can ease out the developer experience of building these neural networks and tracking there performance efficiently.

Nowadays, neural networks are heavily used for classifying objects, predicting data and other similar tasks by many companies out there. When it comes it to training neural networks and keeping track of their performance, the experience is not too subtle.

When building a neural network, a developer would be trying out multiple datasets and experimenting with different hyperparameters. It is essential to keep a track a of these parameters and how they affect the output of the neural networks.

Also debugging neural networks is an extremely cumbersome task. The output performance of different neural networks may vary due to different reasons. Some of the possible causes maybe inadequate data pre-processing, incorrect optimizer, a learning rate which is too low or too high. The number of variables which affect the performance of a neural network are quite a few. Hence it is essential that every parameter is properly tracked and maintained.

Some of the available options present out there include the infamous ***Tensorboard, Bokeh ***to name a few.

In this project we will be using **MlFlow** ,an open source platform to manage the entire deep learning development cycle. **MlFlow** allows developers to log and track the outputs of every experiment performed with great precision. We will be looking into **MlFlow’s** components with more detail in the subsequent sections.

The framework we would be using for writing our neural networks and training them is **Keras**.

We will be using the **FashionMNIST** dataset. It contains a total of 70000 gray scale images (training:test = 60000:10000) , each scaled at 28x28 associated with one from 10 classes. (Fig 1)

![Fig 1: Fashion Mnist Dataset](https://cdn-images-1.medium.com/max/2000/1*vIpkolZZ3jACO529n4QaJg.jpeg)
	

The folder structure of our project looks as shown in Fig 2 below.

![Fig 2: Project folder structure](https://cdn-images-1.medium.com/max/2000/1*NaIzdpiaiWAge7khYyRpmQ.png)

The data folder contains our ***fashion mnist*** dataset files which will be used for training the model. The db folder contains the python driver code to perform operations on ***MongoDb*** collections. ***MongoDB*** is an extremely easy to use ***NoSql*** database which has been built keeping in developer satisfaction. It is easily integrable with modern day applications and has a large developer community contributing to it’s extensions regularly. The model folder contains piece of code with the neural network model definition. The ***mlruns*** folder is created automatically once ***mlflow ***is invoked in the main code.

*The aim of the project is to track multiple deep learning experiments and check how the outputs are affected when parameters are changed and data is changed.* Since we have only one dataset, we will split it into equal parts in order to simulate a multiple dataset scenario.

Let’s start off with the ***create_dataset*** script, which is used to split the fashion mnist into equal parts and store them inside the data folder with proper serial number.

In Fig 3 shown below, we import fashion mnist from ***keras.datasets ***and perform the necessary normalization step

![Fig 3: Loading Fashion Mnist Dataset from the keras library](https://cdn-images-1.medium.com/max/2452/1*_GWdTw2PkJDBl5HW3-Yjbw.png)

Next, we write the methods to split the dataset into equal parts and save them with proper incremental serial numbers inside the data folder, inside the root project directory (Fig 4)

![Fig 4: The large dataset is equally split into 12 equal parts for training](https://cdn-images-1.medium.com/max/2060/1*NVW44h1BejUe25D0SwUYog.png) equal parts for training*

After this we go ahead and write the necessary driver code to manage our newly created dataset using ***MongoDb***. Some might say that using ***MongoDb*** for such a small project might be an overkill, but personally I find *MongoDb* to be an excellent tool for managing data with flexibility. Given ***NoSql***’s schema-less nature, managing collections and documents is a breeze.

The ease with which any document can be edited in ***MongoDB*** is superb. The best part is, whenever a collection is queried , the result returned is a ***json ***making it extremely easy to be parsed using any programming language. Aggregation queries in mongo are also very simple and allows users to cross reference collections in a swift manner.

In order to connect our python scripts with MongoDb we will be using **pymongo** which can be easily installed using the ***pip install pymongo***. To install MongoDb, follow this tutorial [***https://docs.mongodb.com/manual/tutorial/install-mongodb-on-ubuntu/](https://docs.mongodb.com/manual/tutorial/install-mongodb-on-ubuntu/)***

Once MongoDb is installed and tested to be running properly on your system, create a new database called fashion_mnist. Inside the database create a new collection named dataset as shown in Fig 5 below.

![Fig 5: Collection named as Dataset created in MongoDb](https://cdn-images-1.medium.com/max/2000/0*o3BTUrrPN6AeIogC)

A great GUI to interact with ***MongoDb*** is ***robo3t***. It’s free and easy to use. It can be downloaded from the following link [***https://robomongo.org/download](https://robomongo.org/download).***Since our DB is setup and datasets are created, we can progress with the task of inserting necessary information into the dataset collection

![Fig 6: MongoClient is configured in db_ops python script](https://cdn-images-1.medium.com/max/2000/1*ZRvtaMxnM8cP1tyeHtvUDA.png)

In Fig 6 shown above, we are importing ***MongoClient*** from the ***pymongo*** library which will essentially be used to connect to our ***mongoDB*** database.

Fig 7 below describes ***mongoQueue*** class which has been written in order to interact with our dataset collection. In line 18 and 19, the collection name is initialized, which is used in all the member functions. The **Enqueue*** *method in line 6 is used to insert dataset information into the dataset collection. The ***Dequeue** *method in line 10 fetches the first dataset which has a *status *field of ‘***Not** ***Processed**’*.* The **setAsProcessing*** and ***setAsProcessed*** *methods are used to set the status field of respective dataset documents in the collection.

![Fig 7: MongoQueue class for handling operations on the database](https://cdn-images-1.medium.com/max/3824/1*fI-pJh6D3dgLZJ5CfzNUow.png)

![Fig 8: Methods to insert data into MongoDB](https://cdn-images-1.medium.com/max/4096/1*YKzl3umNaRPxJsbSK8Q1lg.png)*Fig 8: Methods to insert data into MongoDB*

We use the ***insert_into_db*** method shown in Fig 8, line 1, to insert information about our newly created datasets into our **mongoDb** dataset collection. In line 23 of the *main* function, we iterate over the dataset folder and call ***insert_into_db** *to insert the necessary information for that dataset into the collection. Once every dataset is successfully inserted into the collection, the fields appear as shown in Fig 9 below.

![Fig 9: Status of document in Dataset collection after data insertion](https://cdn-images-1.medium.com/max/2452/0*qbkKHLQgdPkW-UNp)

We can now define our model for training our deep learning network. Inside ***model/model.py** *we import all necessary ***keras*** packages to build our CNN network (shown in Fig 10a)

![Fig 10a: Importing all necessary keras packages](https://cdn-images-1.medium.com/max/2312/1*XnED-4kpiekoGLMgVMRESA.png)

Fig 10b shows the model architecture. It is a simple two layer CNN, with two **MaxPool** layers and **RelU** activation in between. Two **Dense** layers are also added with 32 and 10 neurons respectively. I have also added a **Dropout** of 0.5 before the last Dense layer.

![Fig 10b: CNN model definintion](https://cdn-images-1.medium.com/max/4096/1*lmfvVEc3LxgxqeCGCCD-MQ.png)

Now, in our ***train.py*** script we import all the necessary modules needed from the ***keras*** library to get on with our training. Along with all the ***keras*** libraries we import ***mlflow ***as well (Fig 11)

![Fig 11: Importing packages in the main script](https://cdn-images-1.medium.com/max/2000/1*roE7f-YhPmBl5h9l1j_IMQ.png)

All the hyperparameters which will be used for training is stored in a config file named as ***train_config.json***. This file is read (Fig 12a) and used for defining training parameters

![Fig 12a: parameters required for training are loaded](https://cdn-images-1.medium.com/max/2000/1*A1cPbWr1ggscrwGyKNo-kw.png)

In Fig 12b, we have defined out training function , which takes arguments ***trainX** (*our training set) *,**trainY** (*training set labels*) *and the ***model** *(CNN model)

![Fig 12b: Function to start training](https://cdn-images-1.medium.com/max/3040/1*aq-dCVGk-s4nBAg6KLOxPg.png)

From line 38 (In Fig 13), the **main** function starts where we define our ***MongoQueue** *object using the ***MongoQueue** *class defined inside script mentioned before, ***db_ops.py **. *As it can be seen in line 41, ***mq*** is our object.

![Fig 13: Main function which trains and also logs data using MlFlow](https://cdn-images-1.medium.com/max/2396/1*jbtG75-OOC9yLrlAHJ-A2Q.png)

In line 42 (Fig 13) , a new CNN model is created by calling the ***model** *function which accepts the optimizer type as input. In this experiment we would be using the ‘**SGD**’ (**Stochastic Gradient Descent**) to train the network .

Everytime we invoke ***mlflow*** in our training code for logging, it is known as an ***mlflow*** run. ***MlFlow*** provides us with an API for starting and managing ***MlFlow*** runs. For example, Fig 14a and 14b

![Fig 14a: Mlflow example (importing and logging params)](https://cdn-images-1.medium.com/max/2000/0*gBpvWcli4WNQxv5a)

![Fig 14b: Context managers can be used to declare and Mlflow run](https://cdn-images-1.medium.com/max/2000/0*-QzXE_uwytc_IEmQ)

In our code we start the ***mlflow*** run using the python ***context manager*** as shown in Fig 14b.

At line 46 in Fig 13, we define our our ***mlflow*** run with the run name as ‘***fashion mnist***’.* *All data and metrics will be logged under this run name on the ***mlflow*** dashboard.

From line 47 we start a while loop, which continuously invokes the ***dequeue** *function from the **MongoQeueue** class. What this does is fetches every row corresponding to a particular dataset from the dataset collection which has a **status field = *Not Processed** *(Fig 9). As soon as this dataset is fetched, **setAsProcessing*** *function is called in line 51 which sets the status of that dataset to *Processing *in **MongoDb**. This enables us to understand which dataset is currently being trained by our system. This is particularly helpful is large systems where there are multiple datasets and many training instances running in parallel.

In lines 54 and 55, the datasets are loaded from the ***data** *folder corresponding to the ***dataset_id** *fetched from the db.

![Fig 15: Loading Test data and evaluating on it](https://cdn-images-1.medium.com/max/2004/1*5lWcOwjfGzwNATxdkJmmgA.png)

Lines 57 and 58 loads the test sets and the training is started at line 59 by calling the *train *function. We then use the trained model to predict our scores as shown in line 60 in Fig 15.

The training output looks as shown below (Fig 16a and Fig 16b)

![Fig 16a: Model summary when starting to train](https://cdn-images-1.medium.com/max/2000/0*BCmu4INlyL7gPnxX)

![Fig 16b : Outputs after first epoch](https://cdn-images-1.medium.com/max/2628/0*eRC8Zm0avYe38JlE)

As shown in Fig 16b, the training happens for an epoch and the evaluation metrics for the test dataset gets logged.

All outputs of the evaluation done using our trained model can be logged using the ***MlFlow*** tracking api (as shown in Fig 17). The ***tracking*** ***API*** comes with functions such as ***log_param*** and ***log_metric*** which enables us to log every hyperparameter and output values into ***mlflow***.

![Fig 17: Using Mlflow’s tracking API to log metrics and params](https://cdn-images-1.medium.com/max/2676/1*9IeXuffAu2mnwh21O35NUg.png)

The best feature about ***mlflow*** is the ***dashboard*** it provides. It has a very intuitive UI and can be used efficiently for tracking our experiments. The ***dashboard*** can be started easily by simply hitting ***mlflow** **ui** *in your terminal as shown in Fig 18 below.

![Fig 18 : Starting the mlflow ui from the command line](https://cdn-images-1.medium.com/max/2000/0*vhmPMa30-Gqnw6CQ)

To access the dashboard, just type [***http://localhost:5000](http://localhost:5000/)** *in your browser and hit enter.

![Fig 19: Mlflow dashboard view.](https://cdn-images-1.medium.com/max/3200/0*rHX5D660ryqeUf7B)

Fig 19 shows how the dashboard looks like. Each ***MlFlow*** run is logged using a ***run ID*** and a ***run name***. The Parameters and the Metrics column log display the parameters and the metrics which were logged while we were training our model

![Fig 20: Individual run information](https://cdn-images-1.medium.com/max/2288/0*DMhMMtoc1_ULtloJ)

Further clicking on a particular run, takes us to another page where we can display all information about our run (Fig 20)

![Fig 21a: Visualizing the test accuracy plot in mlflow](https://cdn-images-1.medium.com/max/3200/0*t5qk_JlbTdNG89sy)

***MlFlow*** provides us with this amazing feature to generate plots for our results. As you can see in Fig 21a, the test accuracy change can be visualized across different training datasets and time. We can also choose to display other metrics such as the ***eval*** loss, as shown in Fig 21b. The smoothness of the curve can also be controlled using the slider.

![Fig 21b: Visualizing the eval loss plot in mlflow](https://cdn-images-1.medium.com/max/3200/0*mPZ1nM4I0kAY2yUU)

We can also log important files or scripts in our project to MlFlow using the ***mlflow.log_artifact*** command . Fig 22a shows how to use it in your training script and Fig 22b shows how it is displayed on the mlflow dashboard.

![Fig 22a: Logging files as artifact on mlflow](https://cdn-images-1.medium.com/max/2000/1*JV0scppcjBZy_isq7HJOpQ.png)

![Fig 22b: model file and db_ops file logged as artifact on mlflow](https://cdn-images-1.medium.com/max/2682/0*iEQvDCKtnOFCidSS)

MlFlow also allows users to compare two runs simultaneously and generate plots for it. Just tick the check-boxes against the runs you want to compare and press on the blue ***Compare*** button (Fig 23)

![Fig 23: Mlflow dashboard which lists all the mlflow runs sequentially](https://cdn-images-1.medium.com/max/2096/0*5It-A9LbYnVJh9KM)

Once you click on compare, another page pops up where all metrics and parameters of two different runs can be viewed and studied in parallel (Fig 24a)

![Fig 24a: Comparing multiple mlflow runs in parallel](https://cdn-images-1.medium.com/max/2976/0*0sNCbZBb3U61SzlQ)

The user can also choose to display metrics such as accuracy and loss in parallel charts as shown in Fig 24b.

![FIg 24b: Comparing multiple mlflow runs using visual plots based on metrics](https://cdn-images-1.medium.com/max/3200/0*NQKwq3wMobxCLImI)

Users can add an MlFlow Project file (a text file in ***YAML*** syntax) to their MlFlow project allowing them to package their code better and run and recreate results from any machine. The MlFlow Project file for our ***fashion_mnsit*** project looks as shown below in Fig 25a

![Fig 25a: MLfow project file containing entry points and also conda path](https://cdn-images-1.medium.com/max/2000/0*JkMKeWYPlReYbuVn)

We can also specify a ***conda*** environment for our ***MlFlow*** project and specify a ***conda.yaml*** file (Fig 25b).

![Fig 25b: Conda file specifying packages and dependencies in the specific environment](https://cdn-images-1.medium.com/max/2000/0*Bb3n9hCnxU6hblOm)

Hence, with this we conclude our project. Developers face several on-demand requirements for monitoring metrics during training a neural network. These metrics can be extremely critical for predicting the output of their neural networks and are also critical in understanding how to modify a neural network for better performance. Traditionally when starting off with deep learning experiments, many developers are unaware of proper tools to help them. I hope this was piece of writing was helpful in understanding how deep learning experiments can be conducted in a proper manner during production when large numbers of datasets are needed to be managed and multiple training and evaluation instances are required to be monitored.

***MlFlow*** also has multiple other features which is beyond the scope of this tutorial and can be covered later. For any information on how to use ***MlFlow*** one can head to the [***https://mlflow.org/](https://mlflow.org/)***
