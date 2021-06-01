# Running PyTorch on AWS EC2 Instance
This document is a brief summary of the steps I took to set up AWS EC2 instance for running PyTorch models. It turns out that setting up the instnace is quite straightforward, and using the instance is the same as access a server remotely.
Below, I will walk you throught the key steps to configurate everything, and also recommend some tools that will make your life easier when working with AWS.

## Table of contents
* EC2 Instance Configuration
* Launching the Instance
* Tools and Softwares

## EC2 Instance Configuration
I will assume that you've created your AWS account, and you can access all EC2 services. Then, you will be able to login and open your EC2 dashboard (navigating to the EC2 from all AWS services), where you can do pretty much everything on it, it should look something like the screenshot below, and you can see that I have one running instance.

<div align="center">
  <img src="https://github.com/shuaiqi361/AdetSSInst/blob/main/EC2_dashboard.png"/>
</div>

### Pick your region
The instance type I am using is located in Canada (Central) ca-central-1, the instances are region specific, so if you launch an instance in one region, you will not be able to access it from a different region, so make sure the region information on the top right of the screen correctly reflects that.

### Configure instance
In your dashboard, click on the Orange ```Launch Instance``` button. You will need to go through several steps:
* Choose an Amazon Machine Image (AMI): Search for deep learning, and choose one of the machine from the list. For me, I used the ```Deep Learning AMI (Ubuntu 18.04) Version 44.0```.
* Choose an Instance Type: Different instance types have different purposes, some are used for computation, some are used for storage, some aree for web assess, so in my case I will use P3 which is a GPU machine optimized for deep learning computation. Scroll down the list and you will find ```p3.2xlarge```, ```p3.8xlarge```, and ```p3.16xlarge``` in the P3 family, they are GPU machines with 1, 4, and 8 Tesla V100 GPU cards respectively, and the corresponding memory is 16G per card, so choose wisely. In my case, I will be checking the box before ```p3.8xlarge``` and click next.
* Configure Instance Details: You don't need to change anything on this page unless you know what you are doing, note that by default in the first row, you are lauching one instance, but you can input any number and launch a batch of them.
* Add Storage: We will be increasing the size of our ROOT Volume to accomodate large datasets such as ImageNet, MS COCO. You can input any number in the Size box. Note that the Root Volume is EBS-backed, the default behavior is to delete it on termination, which means if you have any data on this disk, the life cycle of the data will be the same as the instance, in other words you data will persist if you just stop the instance, but it will be deleted if the instance is terminated for good. There is also an option of ```Add New Volume```, I'm not currently using it, but this is like a separare disk attached to this instance, and you can also access this disk from other instances. The default ```Delete on Termination``` is unchecked.
* Add Tag: You can skip this.
* Configure Security Group: This basically determines who can access this instance. If you want to minitor the training process with Tensorboard or Jupyter Notebook, you might want to add some entries to it,

<div align="center">
  <img src="https://github.com/shuaiqi361/AdetSSInst/blob/main/security_group.png"/>
</div>

* Review and Launch: If you have finished everything above you can now launch your instance. When launching the instance, you will need your private keypair, if you are using it for the first time, you can generate the keypair, make sure you know where your store the generated keypair, because you will need it later. The keypair will be a file ending in ```.pem```.

## Launching the Instance
Finally, we have our EC2 instance running and we are ready to ssh into it and start training our models. In your dashboard, you will now find that you have one running instance. To access the Instance, you will need the ```IPv4 Public IP```. Note that, everytime when you stop and resume the instance, the IP will change.

There are 2 ways to interact with your instance from your local terminal:
* Login with ssh: ```ssh -v -i X ubuntu@Y``` where X represents the path to the keypair file and Y represents the Public IP of your instance.
* Copy a file to it with scp: ```scp -i W -r X ubuntu@Y:Z``` where W is the path to the keypair file, X is the path to the local file, Y is the Public IP, and Z is the destination path on the instance.

It’s important to note that if you’re using the keypair file for the very first time, you’ll need to change its permission to read and write by running:

```chmod 600 ~/ec2_keypair.pem```

With all that being said, we can finally fire up a terminal and execute the following command:

```ssh -v -i ~/ec2_keypair.pem ubuntu@IP```

#### The first time when you launch an instance, it will be running immediately, so do not forget to stop it if you don't want to use if right away. More importantly, once your training is done, you need to manully stop your instance from the dashboard. Stopping the instance will keep your data on the disk, but terminating your instance will delete everything.

## Tools and Softwares
There are some useful tools and softwares that I've been using when working with a remote server, where you what to download or upload files, edit your code just like working on your local machines.

#### TMUX
If you fire up a terminal locally and ssh into your instance, and then you start doing some training, if unfortunately your local terminal session ends for some reason (or you just want to shut down your machine), you will find that the training session on your instance is also terminated. This is where this nifty little package called ```tmux``` comes in, they can save your life when running your instance for a long period of time. Tmux makes it so that anything running within a session persists even if the connection drops or the terminal gets killed. 

To see it in action, I’d suggest you watch the following [video](https://www.youtube.com/watch?v=BHhA_ZKjyxo). 

For a more complete list of tmux commands, you should refer to this lovely [cheatsheet](https://gist.github.com/MohamedAlaa/2961058).

#### FileZilla
This software is for data transfer between your local machine and the EC2 instance. Use the ```Site Manager``` to create a new entry for connection. Under the general tag, input the IP to the Host field, select SFTP as the Protocol, from the bottom select Key file as the login type, the User is going to be ubuntu for ubuntu AMIs, finally upload your keypair file with the browse button, the settings should look like this,

<div align="center">
  <img src="https://github.com/shuaiqi361/AdetSSInst/blob/main/filezilla%20settings.png"/>
</div>

Click connect, and you are good to go.

#### Atom with Remote-FTP
Atom itself is an editor with some nice features, such as text highlighting for coding, auto completion of code, and etc. If you don't feel comfortable using tools, such as vim, to debug on the remote server like me, Atom is your cure. 
* First you will need to install an add-on package called ```remote-FTP``` in Atom. When remote-FTP is installed, you will be able to see it under the Packages tag.
* Create a project folder and create a ```.ftpconfig``` file under it, this can be done by clicking on the Packages tag select remote-FTP and then select create sftp config file, fill in the fields accordingly in that file and save.
* Reboot atom, you will find that there is a Remote tag besides the Project tag you just created, click connect, and you will be able to see your remote directory.
* You can open and edit any file from atom, once you have saved your edits locally, the remote will be automatically syncronized.

Here is an example of the config file:

<div align="center">
  <img src="https://github.com/shuaiqi361/AdetSSInst/blob/main/atom_config_remote_ftp.png"/>
</div>
