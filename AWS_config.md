# Running PyTorch on AWS EC2 Instance
This document is a brief summary of the steps I took to set up AWS EC2 instance for running PyTorch models. It turns out that setting up the instnace is quite straightforward, and using the instance is the same as access a server remotely.
Below, I will walk you throught the key steps to configurate everything, and also recommend some tools that will make your life easier when working with AWS.

## Table of contents
* EC2 Instance Configuration
* Launching the Instance
* Tools and Softwares

## EC2 Instance Configuration
I will assume that you've created your AWS account, and you can access all EC2 services.
#### Pick your region
The instance type I am using is located in Canada (Central) ca-central-1, the instances are region specific, so if you launch an instance in one region, you will not be able to access it from a different region, so make sure the region information on the top right of the screen correctly reflects that.
