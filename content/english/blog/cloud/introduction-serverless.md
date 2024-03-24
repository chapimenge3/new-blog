---
title: "What is Serverless Architectures?"
description: "You must been hearing about serverless architectures. But what is it? Let's find out."
meta_title: ""
date: 2022-12-02 10:23:39
image: "/images/blogs/aws/serverless.png"
categories: ["Cloud"]
author: "Chapi Menge"
tags: ["cloud", "aws", "serverless"]
draft: false
---

You must been hearing about serverless architectures. But what is it? Let's find out.

# What is Serverless Architectures?

Hey fam! I am back with another blog post which is one of the most hot topics in the cloud computing world. In this blog post, I will be talking about what is serverless architectures specially in AWS.

I will be talking about the following topics:

- [What is serverless architectures?](#what-is-serverless)
- [Why use serverless architectures?](#why-use-serverless-architectures)
- [How does it work?](#how-does-it-work)
- [What are the pros and cons of serverless architectures?](#what-are-the-pros-and-cons-of-serverless-architectures)
- [How can you use serverless architectures in AWS?](#how-can-you-use-serverless-architectures-in-aws)
- [What are the different serverless services in AWS?](#what-are-the-different-serverless-services-in-aws)
- [What's next?](#whats-next)

## What is serverless?

Serverless is a cloud computing execution model where the cloud provider dynamically manages the allocation and provisioning of servers. A serverless application runs in stateless containers that are event-triggered, ephemeral (may last for one invocation), and fully managed by the cloud provider.

## Why use serverless architectures?

A serverless architecture is a way to build and run applications and services without having to manage infrastructure. Your application still runs on servers, but all the server management is done by Cloud provider. You no longer have to provision, scale, and maintain servers to run your applications and storage systems. Serverless architectures allow you to focus on your core product and business logic.

So why use serverless architectures? Here are some of the reasons:

    - You don't have to manage servers.
    - You don't have to worry about scaling.
    - You don't have to worry about patching.
    - You don't have to worry about high availability.
    - You can focus on your core business logic.
    - Rapid development.
    - You can save money from idle servers.
    - Less complexity.
    - and the list goes on...

## How does it work?

So by now we know a little bit about what is serverless architectures and why we should use it. But how does it work? Let's take a look at the diagram below:

{{< image src="images/blogs/aws/aws-lambda.webp" caption="" alt="alter-text" height="" width="" position="center" command="fill" option="q100" class="img-fluid" title="AWS Lambda"  webp="false" >}}


As you can see in the diagram above, there are 3 main components:

    - Event
    - Function
    - Service

Let's break them down one by one:

### 1. **Event**

Event is a trigger that invokes a function. It can be a HTTP request, a file upload, a database update, a message queue, or any other event. For example, when you request url like

    https://example.com/api/users/5f971013-8a1a-4276-b58c-36951cb43b53

then it will trigger a function. Or when you upload a file to S3 bucker(AWS File Simple Storage Service) then it will trigger a function to do like image processing, file compression, etc.

so event is the cause of the function to run.

### 2. **Function**

Function is a piece of code that runs in response to an event. It is a piece of code that runs when an event occurs. It can be a HTTP request, a file upload, a database update, a message queue, or any other event. This code is written in a programming language like Python, Node.js, Java, C#, etc. It can be a single function or a group of functions which is being called by the main function. This piece of code can do things like

    - Process data
    - Get user details
    - Send email
    - Run a query
    - Search for a blog in your service.
    - and the list goes on...

This function is the main part of the serverless architecture. Hoping that you have some idea on how to make a simple web application on any framework like Django, Flask, etc. Then you can think of this function as function in your web application which will be run when your request some endpoint.

So to summarize, function is a piece of code that runs in response to an event. Event can be a HTTP request, a file upload, a database update, a message queue, or any other event.

### 3. **Service**

Service means anything that you will use to utilize the function. This means it can be AWS SNS(AWS Simple Notification Service), AWS SQS(AWS Simple Queue Service),, Apache Kafka, etc.

So Service is the thing that can be used to along side with the function to make it more useful. For example, if you want to send an email to your user when they register to your service. Then you can use AWS SNS(AWS Simple Notification Service) to send an email to your user. So AWS SNS is the service that you will use to send an email to your user.

To summarize, service is the thing that can be used to along side with the function to make it more useful.

## What are the pros and cons of serverless architectures?

Serverless architectures is getting more and more popular these days. Even though it is getting more and more popular, it comes with some pros and cons. Let's take a look at the pros and cons of serverless architectures.

I have mentioned some pros in the beginning of the blog nevertheless Let's start with the pros:

    - Scalability
    - High Availability
    - Less Complexity
    - Reduced operational costs
    - Efficient usage of resources
    - Focus on business, not on infrastructure
    - System security is outsourced
    - Continuous delivery
    - Cost model is startup friendly
    - and many more

To move on the pros. Even though serverless architectures is very interesting thing to learn and use. It comes with some cons. Let's take a look at the cons:

    - Cold start
    - Vendor lock-in
    - Limited runtime
    - Limited language support
    - Limited debugging capabilities
    - list goes on...

I wanna take a moment to describe the cons a little bit deeper.

1. **Cold start**

The way Serverless model works is when some event triggers the function, new machine will be startup and install all the necessary dependencies and then run the function. This process is called cold start. This process can take some time to startup and install all the dependencies. So if you have a lot of traffic, then this cold start can be a problem.

We have a fix for this problem or let's say improvement. So There is a thing called Warm up server. What it will do is it will keep the server warm so that when the event triggers the function, it will not have to go through the cold start process. This will improve the performance of the serverless architecture.

2. **Vendor lock-in**

Vendor lock-in is a situation where a user of a product or service is dependent on a specific vendor for their ability to operate. So in this case, if you are using AWS Lambda, then you are dependent on AWS. If you want to migrate to another cloud provider, then you will have to rewrite your code. So this is a problem if you plan to migrate to another cloud provider.

3. **Limited runtime**

Serverless architectures are limited to the runtime that is supported by the cloud provider. For example, if you are using AWS Lambda, then you can only use the runtime that is supported by AWS Lambda which is max 15 min currently. So if you want to use more than the max runtime, then you will have to find another solution.

4. **Limited language support**

Serverless architectures are limited to the language that is supported by the cloud provider. For example, if you are using AWS Lambda, then you can only use the language that is supported by AWS Lambda which is Python, Node.js, Java, C#, etc.

But AWS Lambda introduced the support for container image which will give you the ultimate superpower to use anything you want. This is a really good thing because if you want to build some library or framework that is not supported by AWS Lambda, then you can use container image to run your code. Recently I use this because i need to build postgresql client which need to be build to C extension. So I use container image to build it and then use it in my function.

## How can you use serverless architectures in AWS?

To use serverless in AWS you can use AWS Lambda. AWS Lambda is a compute service that lets you run code without provisioning or managing servers. AWS Lambda executes your code only when needed and scales automatically, from a few requests per day to thousands per second. You can use AWS Lambda to run code for virtually any type of application or backend service - all with zero administration. Just upload your code and Lambda takes care of everything required to run and scale your code with high availability. You can set up your code to automatically trigger from other AWS services or call it directly from any web or mobile app.

There are other aws serverless services to list some of them

- [AWS API Gateway(AWS Application Programming Interface)](https://aws.amazon.com/api-gateway/)
- [AWS SNS(AWS Simple Notification Service)](https://aws.amazon.com/sns/)
- [AWS SQS(AWS Simple Queue Service)](https://aws.amazon.com/sqs/)
- [AWS S3(AWS Simple Storage Service)](https://aws.amazon.com/s3/)
- [AWS DynamoDB(AWS NoSQL Database)](https://aws.amazon.com/dynamodb)
- [AWS Step Functions(AWS Workflow Service)](https://aws.amazon.com/step-functions/)
- [AWS Fargate(AWS Container Service)](https://aws.amazon.com/fargate/)
- [AWS AppSync(AWS GraphQL Service)](https://aws.amazon.com/appsync/)
- [AWS EventBridge(AWS Event Service)](https://aws.amazon.com/eventbridge/)
- [AWS CloudWatch(AWS Monitoring Service)](https://aws.amazon.com/cloudwatch/)
- [AWS CloudFormation(AWS Infrastructure as Code Service)](https://aws.amazon.com/cloudformation/)
- the list goes on...

In the next couple of blog i will take you to a project where we build completely using serverless architectures

## What are the different serverless services in AWS?

In the previous topic i have list some of the serverless services in AWS. But i wanna take a moment to describe them a little bit deeper.

Many company now a days are using serverless architectures to build their application. So AWS is one of the cloud provider that provides serverless services. In the world where serverless is adopted there is also a lot of serverless services that you can use to build your application. So let's take a look at some of the serverless services that you can use to build your application.

### **Serverless services on AWS**

#### 1. **Compute**

- [AWS Lambda](https://aws.amazon.com/lambda/) - is event-driven, serverless computing platform provided by Amazon as a part of Amazon Web Services. It is a computing service that runs code in response to events and automatically manages the computing resources required by that code.

- [AWS FarGate](https://aws.amazon.com/fargate/) - is a compute engine for containers that works with both Amazon Elastic Container Service (Amazon ECS) and Amazon Elastic Kubernetes Service (Amazon EKS). You can

#### 2. **Application integration**

- [Amazon EventBridge](https://aws.amazon.com/eventbridge/) - is a serverless event bus that makes it easy to connect applications together using data from your own applications, Software-as-a-Service (SaaS) applications, and AWS services.

- [AWS Step Functions](https://aws.amazon.com/step-functions/) - is a web service that enables you to coordinate the components of distributed applications and microservices using visual workflows. You build applications from individual components that each perform a discrete function, or task, allowing you to scale and change applications quickly.

- [Amazon SQS](https://aws.amazon.com/sqs/) - is a fully managed message queuing service that enables you to decouple and scale microservices, distributed systems, and serverless applications.

- [Amazon SNS](https://aws.amazon.com/sns/) - is a fully managed messaging service for both application-to-application (A2A) and application-to-person (A2P) communication. SNS provides topics for high-throughput, push-based, many-to-many messaging. It is simple and cost-effective, and it makes it easy to decouple senders and receivers of messages.

- [Amazon API Gateway](https://aws.amazon.com/api-gateway/) - is a fully managed service that makes it easy for developers to create, publish, maintain, monitor, and secure APIs at any scale. With a few clicks in the AWS Management Console, you can create an API that acts as a “front door” for applications to access data, business logic, or functionality from your back-end services, such as workloads running on Amazon Elastic Compute Cloud (Amazon EC2), code running on AWS Lambda, or any web application.

- [AWS AppSync](https://aws.amazon.com/appsync/) - is a managed GraphQL service that makes it easy to develop APIs and data-driven applications. AppSync provides a flexible data modeling platform that allows you to easily create and update data models and connect to your data sources. AppSync also provides built-in security, data transformation, and access control features to help keep your data safe and your applications fast.

#### 3. **Storage**

- [Amazon S3](https://aws.amazon.com/s3/) - is object storage built to store and retrieve any amount of data from anywhere on the Internet. You can use Amazon S3 to store and protect any amount of data for a range of use cases, such as websites, mobile applications, backup and restore, archive, enterprise applications, IoT devices, and big data analytics.

- [Amazon EFS](https://aws.amazon.com/efs/) - is a fully managed elastic NFS file system for use with AWS Cloud services and on-premises resources. It is built to scale on demand to petabytes without disrupting applications, growing and shrinking automatically as you add and remove files, eliminating the need to provision and manage capacity to accommodate growth.

- [Amazon DynamoDB](https://aws.amazon.com/dynamodb/) - is a key-value and document database that delivers single-digit millisecond performance at any scale. It's a fully managed, multiregion, multimaster, durable database with built-in security, backup and restore, and in-memory caching for internet-scale applications.

- [Amazon RDS Proxy](https://aws.amazon.com/rds/proxy/) - is a proxy service that makes it easy to scale your Amazon Relational Database Service (Amazon RDS) deployments. It provides a connection pooling layer that can handle thousands of database connections, making it easier to manage database load and scale your applications.

- [Amazon Aurora Serverless](https://aws.amazon.com/rds/aurora/serverless/) - is a new deployment option for Amazon Aurora that automatically starts, scales, and shuts down your database capacity with per-second billing. It is a fully managed, auto-scaling configuration for Amazon Aurora that can automatically start up, shut down, and scale your database capacity up or down based on your application’s needs.

- [Amazon Redshift Serverless](https://aws.amazon.com/redshift/serverless/) - It makes it easy to run and scale analytics without having to manage your data warehouse infrastructure while only paying for what you use.

- [Amazon Neptune Serverless](https://aws.amazon.com/neptune/serverless/) - is an on-demand, scalable graph database that automatically provides customers with capacity based on an application's needs.

Here is the image that shows the different serverless services in AWS

{{< image src="images/blogs/aws/awsserverless-services.webp" caption="" alt="alter-text" height="" width="" position="center" command="fill" option="q100" class="img-fluid" title="Serverless Services"  webp="true" >}}

## What's next?

In the next blog i will take you to a project where we build completely using serverless architectures. So stay tuned for the next blog.

You can also recommend me some project ideas that you want me to build using serverless architectures. I will try to build it and share it with you guys.

If you have any questions or comments please feel free to leave a comment below. I will be happy to answer your questions.

Also if you like this blog please share it with your friends and colleagues. It will help me a lot.

Until next time, wish you the best.

Chapi Menge.
