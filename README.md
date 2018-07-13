# Demo solution for Assembly Payments

This document describes the steps necessary to build a solution to automate a golang application deployment on ECS cluster hosted in AWS with two availability zones.

## Overview
There are two parts for this solution:
* Infrastructure provisioning
* Application deployment

CloudFormations templates are being used for Infrastructure provisioning, Docker for creating images and ECR repository for storing images.

## Architecture Diagram

![Architecture Diagram](https://github.com/rinoshkk/assembly-demo/blob/master/Assembly_Demo.png)

### Prerequisites

* AWS account with aws cli already configured with aws_access_key_id, aws_secret_access_key and aws_default_region
* S3 bucket with cfn_templates directory copied to the S3 bucket
* IAM user with programatic access to ECR (Elastic Container Repository) for pushing docker images

### Assumptions
* ECR: 069934997449.dkr.ecr.ap-southeast-2.amazonaws.com/assembly_demo:latest
* Running docker on a Linux machine for generating the docker image
* S3 bucket for storing CF templates located at: https://s3-ap-southeast-2.amazonaws.com/cfn_templates/

### Setup

First step is to create an image which can be used for deployment

```
docker build -t assembly_demo -f Dockerfile.build . 
```
Dockerfile.build is located on the GitHub root dir.

lists the images we just built
```
docker images
```
Login to ECR with the output generated by this command and then push the image to ECR.
```
aws ecr get-login --no-include-email 

docker tag assembly_demo:latest 069934997449.dkr.ecr.ap-southeast-2.amazonaws.com/assembly_demo:latest

docker push 069934997449.dkr.ecr.ap-southeast-2.amazonaws.com/assembly_demo:latest
```

Now our image is ready and deployed on AWS ECR!

Next step is, running the cloudformation template to create the infrastructure stack including the ECS cluster.

```
aws cloudformation create-stack --stack-name assembly-demo --template-url https://s3-ap-southeast-2.amazonaws.com/cfn_templates/go-app-demo.yml
```
This template file has all other required templates like VPC, ALB, ECS nested inside it.

If all steps are executed correctly, we will have the complete stack ready with the image on ECR deployed on the ECS containers ready with the application running.

## Future Enhancement

Still working on the CodePipeline part to make the whole workflow automated, starting from taking the go application from GitHub repo and make a docker image on the fly and deploy it on ECR through CodeBuild

