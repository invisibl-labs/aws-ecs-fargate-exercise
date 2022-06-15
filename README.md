# Exercise for AWS ECS - Fargate with Docker images

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


