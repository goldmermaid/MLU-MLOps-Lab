<img align="center" src="imgs/MLU_Logo.png">

---



# MLU-MLOps Lab
For lecture usage on the ML Operation (MLOps) course in Amazon Machine Learning University (MLU).

##### Table of Contents  
- [MLU MLOps Introduction](#Introduction)  
- [Pre-requisites](#Pre-requisites)  
- [Lab Overview](#Lab-Overview)
  - [The Pipeline Architecture](#The-Pipeline-Architecture)
- [Lab Instructions](#Lab-Instructions)
- [Supplementary](#Supplementary)




<a name="Introduction"/>

## MLU MLOps Introduction

In this course, we will first overview how to architect end-to-end ML systems: from initial ML project scoping to data ingestion, from model training and deployment, to model serving, monitoring and maintenance. Through the lectures, we will explore the ML Systems of several key products and services within Amazon to get you inspired with a varied of ML system decisions. Beside, by reviewing some ML pitfalls, we provide some practical solutions for you to debug the ML systems.

We structure the course as following:

* Lecture 1: *what components* needed for an ML system (via an MiniAmazonGo example)
* Lecture 2: what *questions (choices) to ask* for each component
* Lecture 3-4: what *options* (answers to above questions) do you have for each component
* Lecture 5: what *consequence* to be aware of (monitoring, social impact, etc.)

<img align="central" src="imgs/mlops_syllabus.png">


<a name="Pre-requisites"/>

## Pre-requisites

### Knowledge Check

You should have some basic experience with:
  - Train/test a ML model
  - Python ([scikit-learn](https://scikit-learn.org/stable/#))
  - [Jupyter Notebook](https://jupyter.org/)
  - [AWS CodePipeline](https://aws.amazon.com/codepipeline/)
  - [AWS CodeCommit](https://aws.amazon.com/codecommit/)
  - [AWS CodeBuild](https://aws.amazon.com/codebuild/)
  - [Amazon ECR](https://aws.amazon.com/ecr/)
  - [Amazon SageMaker](https://aws.amazon.com/sagemaker/)
  - [AWS CloudFormation](https://aws.amazon.com/cloudformation/)


Some experience working with the AWS console is helpful as well.

### AWS Account

 You'll need an AWS Account with access to the services above. There are resources required by this workshop that are eligible for the [AWS Free Tier](https://aws.amazon.com/free/) if your account is less than 12 months old. 

<a name="Lab-Overview"/>


## The ML Pipeline Architecture of this Lab

You'll make use of the following structure for training the model, testing it, deploying it in two different environments: DEV - QA/Development (simple endpoint) and PRD - Production (HA/Elastic endpoint).

![Train Deploy and Test a ML Model](imgs/MLOps_Train_Deploy_TestModel.jpg)


1. An ETL process or the ML Developer, prepares a new dataset for training the model and copies it into an S3 Bucket;
2. CodePipeline listens to this S3 Bucket, calls a Lambda function for start training a job in Sagemaker;
3. The lambda function sends a training job request to Sagemaker;
4. When the training is finished, CodePipeline gets its status goes to the next stage if there is no error;
5. CodePipeline calls CloudFormation to deploy a model in a Development/QA environment into Sagemaker;
6. After finishing the deployment in DEV/QA, CodePipeline awaits for a manual approval
7. An approver approves or rejects the deployment. If rejected the pipeline stops here; If approved it goes to the next stage;
8. CodePipeline calls CloudFormation to deploy a model into production. This time, the endpoint will count with an AutoScaling policy for HA and Elasticity.
9. Done.


<a name="Lab-Instructions"/>

## Lab Instructions

### Overview

In this course, you'll implement and experiment a basic MLOps process, supported by an automated infrastructure for training/testing/deploying/integrating ML Models. It is comprised into four parts:

1. You'll start with a **WarmUp**, for reviewing the basic features of Amazon Sagemaker;
2. After that, you will train the model (using the buil-in XGBoost or the a custom container if you ran the step 2), deploy them into a **DEV** environment, approve and deploy them into a **PRD** environment with **High Availability** and **Elasticity**;
3. Finally, you'll run a Stress test on your production endpoint to test the elasticity and simulate a situation where the number of requests on your ML model can vary.

Parts 2 and 3 are supported by automated pipelines that reads the assets produced by the ML devoloper and execute/control the whole process.


<a name="The-Pipeline-Architecture"/>


### Step 0. Upload files to S3

As we wish to build a repeatable, testable and auditable end-to-end pipelien, we will use CloudFormation, *an infrastructure automation platform for AWS*, to create all the components required for the exercises. CloudFormantion requires some templates (e.g. `.yaml` files) to launch a repeatable ML pipeline. Hence, we will need to upload the templates to a S3 bucket as showing below:

First, login to the AWS account and navigate to [S3 Buckets main page](https://s3.console.aws.amazon.com/s3/home?region=us-west-2#), and create an S3 bucket (Create Bucket) as below:

  * the bucket name is “mlops-launch-template-YOURALIAS” (Please replace YOURALIAS with your own alias, e.g., mlops-launch-template-rlhu as shown in the below image);
  * the S3 bucket region will be *us-west-2*;
  * you don’t need to change other settings but go ahead to “Create bucket”.


Then, 

1. *download* `MLOps Labs.zip (https://quip-amazon.com/-/blob/KQc9AAqsMUK/M983C8oVL8o0iorRgkIbOg?name=MLOps%20Labs.zip&s=DrrjAL7Fb9kj) (includes all the CloudFormation .yml and .zip files )  and *unzip it*;
2. *upload* the above .yml and .zip files to the S3 bucket you just created:

    * After *unzipped* and uploaded the files, the S3 bucket looks like:







First, you need to execute a CloudFormation script to create all the components required for the exercises.

1. Select the below to launch CloudFormation stack.

Region| Launch
------|-----
US East (N. Virginia) | [![Launch MLOps solution in us-east-1](imgs/cloudformation-launch-stack.png)](https://console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks/new?stackName=AIWorkshop&templateURL=https://s3.amazonaws.com/aws-ai-ml-aod-latam/mlops-workshop/m.yml)

1. Then open the Jupyter Notebook instance in Sagemaker and start doing the exercises:

    1. [Warmup](lab/01_Warmup): This is a basic exercise for exploring the Sagemaker features like: training, deploying and optmizing a model. If you already have experience with Sagemaker, you can skip this exercise.
    2.[Train your models](lab/02_MLOpsPipeline): In this exercise you'll use the training pipeline. You'll see how to train or retrain a particular model by just copying a zip file with the required assets to a given S3 bucket.
    3. [Stress Test](lab/03_Testing): Here you can execute stress tests to see how well your model is performing.


----

<a name="Supplementary"/>

## Supplementary

If you would like to use a customized environment (i.e. rather than Sagemaker built-in ones), you may create a customized Docker image on your own. In this case, you will need to create a `Dockerfile` with 

- `pip install ...` selected packages;
- training and inference code.

Then, build the customized doker by running `!docker build -f Dockerfile -t ...`. 


Please see [sample code](https://github.com/awslabs/amazon-sagemaker-mlops-workshop/blob/master/lab/01_CreateAlgorithmContainer/01_Creating%20a%20Classifier%20Container.ipynb) about how to create a customized Docker image.



----
# Cleaning

First delete the folowing stacks:
 - mlops-deploy-iris-model-dev
 - mlops-deploy-iris-model-prd
 - mlops-training-iris-model-job

Then delete the stack you created. **WARNING**: All the assets will be deleted, including the S3 Bucket and the ECR Docker images created during the execution of this whole lab.


 
----
## License Summary
This sample code is made available under a modified MIT license. See the LICENSE file.

