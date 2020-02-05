---
title: boost-up-neural-networks-using-docker-containers-and-spark
date: 2020-01-09 15:13:20
author: Ajay Singh
tags:
- spark
- Docker
- neural-network
category: AI
---




# Boost up neural networks using docker containers and pyspark part : 1


![alt_text](Boost-up0.jpg "image_tooltip")


src: [https://images.app.goo.gl/itH6Cbq8LK7cNZxM8](https://images.app.goo.gl/itH6Cbq8LK7cNZxM8)

Introduction:

In this blog we will learn how we can use spark to boost up the inference speed of the neural network model. The whole topic is too long to cover in a single blog so we will divide it in two parts



1. Create spark cluster on which we will run multiple instances of the model 
2. Run the neural networks on the cluster

**Lets begin,**

For creating cluster we will use docker containers and create a common network where they can communicate with each other

Lets start with introduction of Docker:

_Docker is a set of platform as a service products that use OS-level virtualization to deliver software in packages called containers. Containers are isolated from one another and bundle their own software, libraries and configuration files; they can communicate with each other through well-defined channels. [https://en.wikipedia.org/wiki/Docker_(software)](https://en.wikipedia.org/wiki/Docker_(software))_

Thanks to docker containers we will be able to create several worker nodes on which we can run spark for distributed processing.

**<span style="text-decoration:underline;">Docker Installation:</span>**

Docker installation is very easy just follow the steps in the following link

[Get Docker Engine - Community for Ubuntu](https://docs.docker.com/install/linux/docker-ce/ubuntu/)

**<span style="text-decoration:underline;">Create Docker image :</span>**

To create a docker image we need to first create a Dockerfile which upon building will install all the libraries that we want.

_Sample docker file:_


```

FROM nvidia/cuda:10.2-cudnn7-devel-ubuntu18.04
MAINTAINER XYZ
RUN apt-get update -y && apt-get install -y python3-pip python3-dev libsm6 libxext6 libxrender-dev
# addons
RUN \
	apt-get install -y \
	wget \
	unzip \
	ffmpeg \ 
	git
RUN pip3 install opencv-python
RUN pip3 install moviepy
RUN pip3 install pandas
RUN pip3 install requests
RUN pip3 install numba
RUN pip3 install imutils
RUN pip3 install filterpy
RUN pip3 install sklearn
RUN pip3 install kafka-python


```


Now build the docker image using the following command


```
$ docker build -t "my_docker_image" .
```
This will create a docker image with name my_docker_image

![fig_1](1.png "image_tooltip")

You can see the list of images by using the following command
```
 $ docker images

```



You should see your newly created image.

Now we are ready to create containers using the above image.

**<span style="text-decoration:underline;">Create docker network:</span>**

Docker network is essential when containers wants to communicate . The most common and default network is bridge.

Run the following command to create a network


```
$ docker network create "my_network"
```


Run the following command to list all existing networks


```
$ docker network ls
```

You should see your newly created networks

![fig_2](2.png "img_tooltip")

Now that we have created the network , we can now create containers and bind them with the network that we created.

Create docker container:

Let's first create a master node of the cluster
 
Run the following command to create the master docker container

```
$ docker run -it --name master-docker --network my_network my_image /bin/sh
```

This command will create the container and attach a shell with the container

To check if the docker is running , run the following command to list down all running container

```
$ docker ps -a

```
You should see the container master-docker running

![fig_3](3.png)

Now lets create 2 worker nodes

The following commands will create two worker nodes with names worker_docker_1 and worker_docker_2

```
$ docker run -it --name worker_docker_1 --network my_network my_image /bin/sh

```
```
$ docker run -it --name worker_docker_2 --network my_network my_image /bin/sh

```
Now again check the running container now you will be able to see one master node and 2 worker nodes.

![fig_4](4.png)

That's it for this part , tune in for part 2 of the blog where we will run spark over the cluster and run neural network over it

```


