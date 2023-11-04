---
title: "Large Scale Serverless Structure"
description: "How do I structure my serverless application for large scale apps?"
meta_title: ""
date: 2022-12-02 10:23:39
image: "/images/blogs/programming/serverless_scale_3.png"
categories: ["Cloud"]
author: "Chapi Menge"
tags: ["cloud", "aws", "serverless"]
draft: true
---

How do I structure my serverless application for large scale apps?


- How do you think you can manage to work on relatively large code base with serverless architecture? 
- What is the best way to structure your serverless project?

Hey Everyone, this time it has been really long time since i write a blog post. I have got bunch of backlog of blog post that i want to write but i was in vacation and i was busy with other stuff.

Today I wanna share my experience on how to structure your serverless project. I have been working on a project that has a lot of serverless functions. I have been working on this project for a while and i have been trying to figure out the best way to structure the project. I have tried different ways to structure the project and i think i have found the best way to structure the project.

Let's break down why does this structuring matters while building any project. So for longest time developer spend their time on maintaining code base rather than building new projects. So, when you are spending most of the time on the code that is already written, you want to make sure that the code is not a mess or they say unorganized/unstructured code base.

So in my serverless project had around 20 lambda function for the first release and 30 endpoints.

Does why does the number of lambda and the endpoint differ? Well i was reusing the lambda function for very similar endpoints. For example sometimes the CREATE/UPDATE endpoint shares a lot in common. so by just using `AWS SAM Events` i was able to reuse the lambda function for both endpoints with different HTTP method.

```yaml

resource:
    CreatUpdateTodo:
        Type: AWS::Serverless::Function
        Events:
            CreateTodo:
                Type: Api
                Properties:
                    Path: /todo
                    Method: post
            UpdateTodo:
                Type: Api
                Properties:
                    Path: /todo
                    Method: put
```