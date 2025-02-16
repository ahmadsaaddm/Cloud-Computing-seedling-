---
title: "Cloud Computing Project report"
author: "Meli Paolo 1920140, Saad Ahmad 2133825"
date: "2024-08-28"
output: pdf_document
---


\newpage
```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```

## Abstract

  This report details the development and deployment of a scalable cloud-based application for plant seedling classification using machine learning. The application aims to assist in early weed identification in agricultural settings, particularly for species similar to maize.    

  The project included some frontend with HTML and primarily backend development, utilizing Flask for the web application and Docker for containerization. The deployment process involved multiple AWS services, including IAM, VPC, ECR, ECS with Fargate, Load Balancing, and AutoScaling.

   A significant portion of our effort focused on overcoming key challenges inherent in cloud-based deployments. Among these were the configuration of network settings to ensure proper communication between services, implementation of robust cross-origin resource sharing (CORS) functionality to enable secure cross-domain interactions, and the  fine-tuning of auto-scaling policies to optimize resource utilization and application performance.  

  To  validate the application's scalability and performance under various load conditions, we conducted comprehensive performance testing using Gatling.  



  This report outlines the step-by-step process of building and deploying the application, highlighting some obstacles encountered . The project demonstrates the practical application of cloud computing concepts in creating a robust, scalable machine learning application for real-world agricultural use.


\newpage


## Introduction  
  In agriculture weeds that may subtract nutrients or infest a cultivation is something that needs to be dealt with. One of the best way is identifying very soon from the seedling if the plant that is growing from the seed is wanted or not.  

  Thanks to a data set we found on kaggle we will use for training we aim at creating a deployable model able to recognize (for now only among some specific species that were available in the data set) the plant from it seedlings.
    
  This data set was put together specifically to recognize weeds similar to maize (corn) that are very crucial to recognize in a maize fields in the seedling period of the plants, but in general given more species on which to train the model we will obtain would work also for other more general seedling recognizing tasks, maybe related to gardening also or for wildlife exploring.

\vspace{2cm}


## The application

### The data  
  As anticipated in the introduction data were taken by kaggle ([v2 plant seedlings dataset](https://www.kaggle.com/datasets/vbookshelf/v2-plant-seedlings-dataset)).  
It consists of 5539 photos of 12 different species of seedlings, taken with different zoom, quality or a different stages of the seedling grow.

\vspace{1cm}

### The training  

  We decided to go for a Neural Network as the classification model using tensorflow to process the images and resorted to data augmentation, distorting and creating new images to have more data and to make the algorithm more robust. 
  
\vspace{1cm}

### Interface

The first step was to create our application folder, containing our Flask app python folder, as well as a docker file, a requirements file, and an upload file to allow the user to upload an image of the seedling. The interface allows us to load the model on the site and return an answer for the predicted class with a confidence percentage.

```{r, echo=FALSE}
knitr::include_graphics("C:/Users/ahmad/Desktop/sap/S2/CLC/Screenshot 2024-09-03 170753.png")
```

\vspace{1cm}

### Backend

  Initially, anyone could access the app but only users on the host internet connection were able to upload an image and get their prediction. We had to backtrack to change our backend file and to use 'CORS()' function in order to make our application functional to all users. 
  
  
\vspace{2cm}

## The deployment


### Services

In order to distribute our application through AWS, we had to make use of several different services which are:

- IAM
- VPC
- AWS ECR
- AWS Fargate 
- AWS ECS
- Load Balancing
- AutoScaling

 The first step was to create a docker container and push it to a repository on ECR using AWS command line.
 
\vspace{1cm}

### IAM Role
  It was necessary to have a proper role to be able to execute ECS services, so we had to add AmazonECSTaskExecutionRolePolicy  to our user permissions.
  
\vspace{1cm}

### VPC

  We created a VPC with 2 availability zones 2 public and private subnets, and we made sure that the internet gateway was properly configured for the VPC and the public subnets.
  Furthermore, we created a security group to allow all traffic to be directed to port 80.
  

```{r, echo=FALSE,fig.width=5, fig.height=5}
knitr::include_graphics("C:/Users/ahmad/Desktop/sap/S2/CLC/vpc.png")
```

\vspace{1cm}

### ECS

  We wrote a task definition that uses 2 containers of the same docker image to execute the task, and we deployed it as a service in the cluster. We also used Fargate instead of EC2 instance in order to facilitate application management and set appropriate resources easily.
  
  Regarding resources, we allocated in the task definition 2vCpu and 4Gb of memory, selected the IAM role we created earlier, and created 2 ambient variables for the targeted ports. 
  
Cluster details:

  - image : "149536476097.dkr.ecr.us-east-1.amazonaws.com/p-seedlingrep:latest"
  - Container Port: 80
  - Host Port : 80
  
\vspace{1cm}

### Load Balancing

 In the EC2 dasboard, after setting up target groups (where we configured the health checks), we created an application load balancer as internet facing and configured it with our VPC using the security groups to distribute load among several target IPs.
 

```{r echo=FALSE, message=FALSE, warning=FALSE}
library(magick)

# Read the image
img <- image_read("C:/Users/ahmad/Desktop/sap/S2/CLC/lb-1.png")

# Resize the image
img_resized <- image_resize(img, "2000x1000") # Specify the desired dimensions


# Save the resized and rotated image
image_write(img_resized, path = "C:/Users/ahmad/Desktop/sap/S2/CLC/lb-1-resized.png")

# Include the resized and rotated image
knitr::include_graphics("C:/Users/ahmad/Desktop/sap/S2/CLC/lb-1-resized.png")

```


### Autoscaling

  The first attempt with memory limitation autoscaling with 10 percent, and after utilizing a bigger portion of the memory for around 5 minutes, another task was executed to scale; and after 5 minutes with no hyper memory utilization, it also scaled down to a single task. 
  
  We then decided to implement a more meaningful autoscaling by setting up a policy which would scale according to a CPU utilization of 60 percent and memory for 50 percent, but manual tests were not sufficient to pass these thresholds.
  
  Therefore, we decided to deploy a scalability test using gattling and saw the following results:
  
  
```{r, echo=FALSE, fig.width=5, fig.height=5}
knitr::include_graphics( "C:/Users/ahmad/Desktop/sap/S2/CLC/response tst.png"
)
```

\newpage


## Conclusions

  In our journey to create a properly scalable application, there were a lot of problems with compatibility and continuity between steps to achieve our goal. A lot of backtracking was necessary to make this project happen.
  
  Several times we just had to tear down everything and restart from VPC; the first time our initial VPC  did not have enough availability zones, and another time we did all our work on Stockholm server and the next day it randomly switched to US server and had work on 2 server. The main obstacles were configuring the VPC and sub nets to match between load balancing, our application and auto scaling. 

  In the end, having a working application uploaded to the internet , that is accessible and scalable gives a sense of accomplishment and it was fun testing and asking friends to test it as well.
  
