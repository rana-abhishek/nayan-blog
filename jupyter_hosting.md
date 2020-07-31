
# Host any port on any machine.

In this short blog post, I will explain a great trick to expose various services like Jupyter-notebook, tensorboard, etc. to the entire internet.

Before following the steps I want you to understand some fundamentals behind the hosting of different services.

### **Some Basics:**

Whenever we host some services on a local server having URLs like ([https://localhost:8888](https://localhost:8888) or [http://127.0.0.1:8888](http://127.0.0.1:8888)) they are behind a NAT or firewall of our computer(Most of the hackers work to breach these).

![](https://cdn-images-1.medium.com/max/2800/0*-j_n3S1WfVi3_DLe.jpg)

To jump over the firewall, we will use [ngrok](http://ngrok.com).

### Ngrok:

Ngrok allows you to expose a web server running on your local machine to the internet. Just tell ngrok what port your web server is listening on.
something like this:

![](https://cdn-images-1.medium.com/max/2000/1*yyRGRBHIsXRHw_8LfL8CAA.png)

### **Steps to host Jupyter-Notebook on AWS EC2:**

1. First, we need to install tmux for running processes(jupyter in our case) in background and jupyter notebook.

    sudo apt-get install tmux
    pip3 install jupyter

2. Download ngrok using

    wget [https://bin.equinox.io/c/4VmDzA7iaHb/ngrok-stable-linux-amd64.zip](https://bin.equinox.io/c/4VmDzA7iaHb/ngrok-stable-linux-amd64.zip)

3. Unzip to install ngrok

    unzip ngrok.zip

4. Create an account on [ngrok](http://ngrok.com) and get authtoken.

![](https://cdn-images-1.medium.com/max/2000/1*C36pyfvzwZnTXSieoHNVEw.png)

    ./ngrok authtoken <your_auth_token>

5. Now on tmux we will run jupyter-notebook

    tmux 
    jupyter-notebook --ip=0.0.0.0 --allow-root 

note the port number

6. Host the noted port using ngrok. For my case it is 8890

    ./ngrok http 8890

Also, you can make a config file and can host multiple ports using the same account as mentioned [here](https://ngrok.com/docs#config).

7. Now note the URL you got on ngrok screen.

![](https://cdn-images-1.medium.com/max/2000/1*QLItoFPpvwq7VNYMVjEqYw.png)

8. Cheers and now hit the URL as many times as you can to access your favorite jupyter notebook.

### Conclusion:

In this cool blogpost, we understood to host jupyter notebook on the local machine. We can host other services like tensorboard or anything you want.
