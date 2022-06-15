# Exercise for AWS ECS - Fargate with Docker images - Beginner's step by step guide.

In this exercise you will be building a simple python flask Docker image, store it in AWS ECR(Container Regsitry) and run the container in Fargate managed by AWS ECS. 

## Setup

A valid AWS account is required.

The IAM user account should have the Administrator Access to management console. 

VPC setup (Can use the default if it is already setup)

[Setup Steps Here](https://docs.aws.amazon.com/AmazonECS/latest/userguide/get-set-up-for-amazon-ecs.html)

[AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-quickstart.html) must be setup to configure AWS config

## Demo Project

Use the following Github link to clone the Python Flask app from the repo. A very simple app with a single path mapping that returns a hello world page with a counter.

```close
git@github.com:invisibl-labs/python-flask-docker.git
```

Use these commands to run the app and check. Make changes to the commands based on your OS.
```close
cd python-flask-docker
// install dependencies
python3 -m pip install -r requirements.txt
// run the app
flask run
```
Use the URL shown in the output.

<img width="593" alt="Screenshot 2022-06-15 at 7 39 18 AM" src="https://user-images.githubusercontent.com/104984822/173721448-b3a16fb9-f5f5-49da-91f3-77bd56223b81.png">

Should see a page like this.

<img width="1119" alt="Screenshot 2022-06-15 at 7 38 57 AM" src="https://user-images.githubusercontent.com/104984822/173721564-ea8cccee-a40b-453d-971f-947a7887ac16.png">


## Create a Docker Image

We need to create a Docker image to run the container on AWS Fargate. A basic knowledge of Docker commands is required for this step. 
Dockerfile is available in the project root folder. The docker file uses python 3.9.2 base image.

```code
FROM python:3.9.2

WORKDIR python-docker

COPY requirements.txt requirements.txt

RUN pip3 install -r requirements.txt

COPY . .

CMD [ "python3", "-m" , "flask", "run", "--host=0.0.0.0"]
```

Run the following docker commands to build and check the images.
```console
// build the image. By default uses the Dockerfile in the "." current path
docker build -t python-flask-app:1.0 .
// check images
docker images

Should see the python-flask-app image in the list.

## Run the Docker Image

Test the image by running it.

```console
// Run the docker image
docker run -d -p 5000:5000 python-flask-app

// Check if container has started
docker ps

// Check container logs to see if the app is running. use the container id listed
docker logs <container id>
```

Check if you are able to access the app using [http://localhost:5000](http://localhost:5000)

## Push the Docker image to AWS ECR(Elastic Container Registry)

[AWS ECR](https://ap-south-1.console.aws.amazon.com/ecr/home?region=ap-south-1) is a container repository where the docker images can be stored and retrieved and also used to run the containers on AWS ECS.

This requires an account with IAM role having Administrator access.

### Create a repository
You can create a repository using AWS console or AWS CLI

### Using AWS console

Navigate to ECR service page and click on 'Get Started'. Create a private repository as shown below.

<img width="762" alt="Screenshot 2022-06-15 at 8 12 06 AM" src="https://user-images.githubusercontent.com/104984822/173725436-91afbd86-c83f-4b46-aecc-9e3f44446cbf.png">
<img width="735" alt="Screenshot 2022-06-15 at 8 12 18 AM" src="https://user-images.githubusercontent.com/104984822/173725450-d14af5e0-6878-4674-93ad-b25156d546c4.png">

Should see the repository created.
<img width="1180" alt="Screenshot 2022-06-15 at 8 15 16 AM" src="https://user-images.githubusercontent.com/104984822/173725791-e8bb8dcc-9564-40be-b1e1-a88e59a07138.png">

### Using AWS CLI

First, you need to authenticate to AWS ECR. Replace the region and aws_account_id in the following command

```console
aws ecr get-login-password --region <region> | docker login --username AWS --password-stdin <aws_account_id>.dkr.ecr.<region>.amazonaws.com
```

After login, create a new repository using the following command.

```console
aws ecr create-repository --repository-name python-flask-app --image-scanning-configuration scanOnPush=false --image-tag-mutability IMMUTABLE --region ap-south-1
```

Should see the response in CLI
<img width="650" alt="Screenshot 2022-06-15 at 8 25 30 AM" src="https://user-images.githubusercontent.com/104984822/173733542-fbd5caac-4acf-41e1-ab36-4a05ac665d23.png">

You can check the AWS console as well.

### Tag and Push the docker image to ECR repository

Tag the image
```console
docker tag python-flask-app:1.0 812455318414.dkr.ecr.ap-south-1.amazonaws.com/python-flask-app:1.0
```

Push the image
```console
docker push 812455318414.dkr.ecr.ap-south-1.amazonaws.com/python-flask-app:latest
```

You can also refer to the "Push Commands" shown by ECR console.
<img width="747" alt="Screenshot 2022-06-15 at 9 30 36 AM" src="https://user-images.githubusercontent.com/104984822/173734176-36d7eab2-7ed1-494b-adcc-ca302d833cf9.png">

After pushing the docker image, you can view the image in the AWS ECR console.

<img width="1144" alt="Screenshot 2022-06-15 at 9 50 05 AM" src="https://user-images.githubusercontent.com/104984822/173736001-71e53c61-f440-4e3f-83f5-2cd3f2e9457b.png">

## Deploy the image to AWS ECS

AWS ECS (Elastic Container Service) allows easy deployment and management of containers. Schedules the containers to run on the cluster according to the resource needs. Integrates seamlessly with other AWS services like ELB, Security groups, EBS and IAM roles. 

Recommend reading the documentation [AWS ECS](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/Welcome.html)

### Setup the required IAM role permissions

The container instances in the cluster are managed by agents that use ECS API. The agent services require access to the APIs. Setup the necessary IAM role policies for the same

<img width="1108" alt="Screenshot 2022-06-15 at 10 21 54 AM" src="https://user-images.githubusercontent.com/104984822/173739310-7f22dbb2-f62f-4a8d-9b38-fb15b71aad76.png">

### Setup the ECS cluster

Before running through the next steps, recommend recollecting the main concepts like Cluster, Service, Tasks, Containers. Refer to the documentation [ECS Components](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/welcome-features.html)

For this exercise we will be creating a ECS cluster with Fargate to run the containers. Recommend reading about Fargate to recollect the concepts. [AWS Fargate](https://docs.aws.amazon.com/AmazonECS/latest/userguide/what-is-fargate.html)

Create a new cluster. Select the one with AWS Fargate.

<img width="978" alt="Screenshot 2022-06-15 at 10 48 45 AM" src="https://user-images.githubusercontent.com/104984822/173742388-00b13356-bfcf-40f3-894d-c74645b8f0da.png">

Provide a cluster name and create

<img width="982" alt="Screenshot 2022-06-15 at 10 51 32 AM" src="https://user-images.githubusercontent.com/104984822/173746439-714d8e5b-c510-4f28-b23c-5064846dbb5e.png">

Created cluster details

<img width="1210" alt="Screenshot 2022-06-15 at 11 00 48 AM" src="https://user-images.githubusercontent.com/104984822/173746556-73ef0f85-0df6-4d26-850c-1b20466d835d.png">

### Create a Task definition

<img width="1422" alt="Screenshot 2022-06-15 at 11 26 42 AM" src="https://user-images.githubusercontent.com/104984822/173754275-d30ea4c1-9e7a-49e0-b0e6-a5ba9a7717e6.png">

Select Fargate from the options

<img width="1152" alt="Screenshot 2022-06-15 at 11 27 11 AM" src="https://user-images.githubusercontent.com/104984822/173754386-f5260648-2ae2-48d9-873a-e484e98afd76.png">


Enter Task definition details

<img width="1149" alt="Screenshot 2022-06-15 at 11 34 57 AM" src="https://user-images.githubusercontent.com/104984822/173754348-189f54b5-0e0d-4d9a-9227-2c1a7dee10bd.png">

Enter Task size

<img width="803" alt="Screenshot 2022-06-15 at 11 40 46 AM" src="https://user-images.githubusercontent.com/104984822/173755011-38fc193c-7bfd-46aa-99f9-229a6a647705.png">

#### Add Container

<img width="794" alt="Screenshot 2022-06-15 at 11 41 46 AM" src="https://user-images.githubusercontent.com/104984822/173755233-3bd38107-6cf6-443b-abc6-83459811e934.png">

Provide the container service details. Change the image url as per your setup.

<img width="939" alt="Screenshot 2022-06-15 at 12 00 25 PM" src="https://user-images.githubusercontent.com/104984822/173758131-c6a5eddc-0e57-499f-be3e-3974fb78c62f.png">

Leave the rest of the container details to default and hit 'Add'. Should see the container created. 

Leave the rest of the Task definition fields and hit 'Create'. Should create Exceution role, Task definition and CloudWatch log group.

<img width="1411" alt="Screenshot 2022-06-15 at 12 05 32 PM" src="https://user-images.githubusercontent.com/104984822/173758822-cc099559-f8a0-4977-8d4d-9ab869d3027b.png">

View the Task defintion details

<img width="1211" alt="Screenshot 2022-06-15 at 12 10 44 PM" src="https://user-images.githubusercontent.com/104984822/173759748-e3b5a3a7-3045-4607-9b9b-45177900091f.png">

### Create a Service

Once the Task definition and Container are configured, we are ready to create a service.

Navigate to the 'python-app' cluster details page and hit 'Create' under the 'Service' tab.

Enter the service details

<img width="1147" alt="Screenshot 2022-06-15 at 1 01 35 PM" src="https://user-images.githubusercontent.com/104984822/173770519-ecbb96f5-25e4-4b18-b728-f9c1ccaf18a0.png">

<img width="1096" alt="Screenshot 2022-06-15 at 1 01 49 PM" src="https://user-images.githubusercontent.com/104984822/173770583-72b3916f-340d-4901-9ffd-6196f49ddec6.png">

<img width="884" alt="Screenshot 2022-06-15 at 1 01 58 PM" src="https://user-images.githubusercontent.com/104984822/173770636-4c22ebbb-1869-43a4-af4e-4bbc2b4ce0f4.png">

<img width="1090" alt="Screenshot 2022-06-15 at 1 04 59 PM" src="https://user-images.githubusercontent.com/104984822/173770673-7f9cbff8-88b0-4247-9ac4-cbb1cb49f6eb.png">

<img width="869" alt="Screenshot 2022-06-15 at 1 05 10 PM" src="https://user-images.githubusercontent.com/104984822/173770721-e54ad42c-deab-4485-a6d0-2dc9ec64fd3d.png">

<img width="1099" alt="Screenshot 2022-06-15 at 1 05 26 PM" src="https://user-images.githubusercontent.com/104984822/173770766-e00f32d3-5e95-4c29-83d9-fbc67e73108f.png">

Once the service is created, you can see the Service and Tasks created under the respective tabs in the cluster details.

<img width="1414" alt="Screenshot 2022-06-15 at 1 06 14 PM" src="https://user-images.githubusercontent.com/104984822/173770899-ad21dd01-015b-4d07-b862-95878ee80e6e.png">

You can see the 'Task' is in running state.

<img width="1399" alt="Screenshot 2022-06-15 at 1 11 30 PM" src="https://user-images.githubusercontent.com/104984822/173771018-d4adf125-9982-4a01-9129-e8912065e28e.png">

### Updating the Security Groups

Navigate to the python-service details page and click on the security groups link to open the security groups page.

<img width="1402" alt="Screenshot 2022-06-15 at 1 17 21 PM" src="https://user-images.githubusercontent.com/104984822/173772390-14739ba0-efc5-424f-951c-9004cdf65bb3.png">

Add an inbound rule to allow port 5000 to allow access to the python app.

<img width="1347" alt="Screenshot 2022-06-15 at 1 20 05 PM" src="https://user-images.githubusercontent.com/104984822/173772665-8572df16-2f16-4449-9d3b-308e74419b41.png">

### Access the python app

We need the public IP to access the app as we have not setup the load balancer.

Open the 'Task' details from the cluster details page and look for public IP under the Network section.

<img width="811" alt="Screenshot 2022-06-15 at 1 13 57 PM" src="https://user-images.githubusercontent.com/104984822/173772968-66039f65-ecbe-4a56-b2a7-91e1b0b9bc8c.png">

Use the public IP to access the python app on browser. If you see the page as shown below, your ECS Fargate setup is complete.

http://<publice IP>:5000
  
<img width="1180" alt="Screenshot 2022-06-15 at 1 24 22 PM" src="https://user-images.githubusercontent.com/104984822/173773630-f0086fe8-5e89-48db-859d-e1334fdba2a3.png">

## Don't forget to clean up the resources
  
## Clean up the repository created in ECR
## Clean up the ECS cluster


