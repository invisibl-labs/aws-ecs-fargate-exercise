# Exercise for AWS ECS - Fargate with Docker images

In this exercise you will be building a simple python flask Docker image, store it in AWS ECR(Container Regsitry) and run the container in Fargate managed by AWS ECS. 

## Setup

A valid AWS account is required.

The IAM user account should have the Administrator Access to management console. 

VPC setup (Can use the default if it is already setup)

[Setup Steps Here](https://docs.aws.amazon.com/AmazonECS/latest/userguide/get-set-up-for-amazon-ecs.html)

[AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-quickstart.html) must be setup to configure AWS config

## Demo Project

Use the following Github link to clone the Python Flask app from the repo.

```close
git@github.com:invisibl-labs/python-flask-docker.git
```

Use these commands to run the app and check. Make changes to the commands based on your OS.
```close
// install dependencies
python3 -m pip install -r requirements.txt
// run the app
flask run
```
Use the URL shown in the output.

<img width="593" alt="Screenshot 2022-06-15 at 7 39 18 AM" src="https://user-images.githubusercontent.com/104984822/173721448-b3a16fb9-f5f5-49da-91f3-77bd56223b81.png">

Should see a page like this.

<img width="1119" alt="Screenshot 2022-06-15 at 7 38 57 AM" src="https://user-images.githubusercontent.com/104984822/173721564-ea8cccee-a40b-453d-971f-947a7887ac16.png">



##
