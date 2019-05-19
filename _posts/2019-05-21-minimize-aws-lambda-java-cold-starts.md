---
layout: post
title: How to minimize AWS Lambda Java cold starts
author: Driss Amri
date: 2019-05-19
excerpt: You've heard about AWS Lambda and their cold starts and are curious what they are and how to deal with it? Let's have a look how bad they really are and what frameworks like Quarkus and Micronaut can do to help together with GraalVM. 
tags:
 - Serverless
 - AWS Lambda
 - Java
 - Quarkus
 - Micronaut
 - GraalVM
---

You've heard about AWS Lambda and you know they have a thing called cold starts, but what are they and how bad are they?
One year a go I was in this exact spot for a new project that would be built using Java, which apparently has almost the slowest cold starts.

### What is a cold start?
AWS Lambda can dynamically scale your function up and down even to zero instances.  
This is really great since you only have to pay for the compute time you really use.   
 
When you have your first incoming request to your function, AWS Lambda will spin up an instance automatically and handle that request. 
Subsequent requests will be handled by the same container until the load goes up, then AWS Lambda will spin up more instances of your function to process all requests.
Each time AWS Lambda has to spin up an instance to process a request, that's called a cold start. 
These requests take longer than requests that come after this to the same instance, which then is called a warm start.

![AWS Lambda cold start]({{ site.url }}/img/post/coldstart.png "AWS Lambda cold start")


### When do these cold starts happen?

1. First invocation to an instance   
When your first request hits your function, a new instances has to be setup: **cold start**.
2. Concurrent invocations  
When your function receives concurrent invocations, new instances have to be setup: **cold start**.
3. After provider resource clean up  
When AWS decides to clean up resources, they will shut down your instances 
and the next invocation has to setup a new instance: **cold start**.
4. After deployment & configuration change  
When you deploy your function or function configuration, all instances of will be destroyed 
and the next invocation will have to setup a new instance: **cold start**


### How do we optimize these cold starts? 

