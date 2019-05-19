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
Let's have a look at what I've learned.

## What is a cold start?
AWS Lambda can dynamically scale your function up and down even to zero instances.  
This is really great since you only have to pay for the compute time you really use.   
 
When the first requests hits your function, AWS Lambda will immediately spin up an instance of your function to handle the request. 
The subsequent requests will be handled by the same instance until the load goes up, then AWS Lambda will spin up extra instance(s) to process them fast enough.

Each time AWS Lambda has to spin up an instance to process a request, that's called a cold start, which could add several seconds to your start up.
These first requests take longer to be processed, requests after this to the same container are usually refered to as warm start.

![AWS Lambda cold start]({{ site.url }}/img/post/coldstart.png)

## When do these cold starts happen?
It's important to understand when these cold starts happen before we can see what we can and should do to minimize them.
Here are a few examples when AWS has to spin up a new instance and go through a cold start:
- First request on your function
- Concurrent invocations
- After AWS cleans up instances (resources)
- After function deployment or configuration change  


## Are cold starts really an issue? 
There are a lot of different types of applications, so understanding your use case is important before you can decide if cold starts are really an issue.

When you are building an asynchronous service, it might not a big issue when that function takes a while to cold start. 
When you are building a synchronous service (REST API) it's probably important to understand and see how big the impact could be.

Since only a small percentage of requests will incur a cold start, it's crucial to put it into perspective and focus on performance of warm instances.
Depending on your Service Level Agreements, you should check if your 95th-99th percentile requests are making the required latency.

## How do we minimize these cold starts?  
What can you do to make these cold starts as fast as possible? There a few things you should keep in mind.  

### 1. Avoid putting your Lambda in a VPC (Virtual Private Cloud)
In short: don't use a VPC unless you **absolutely have to**.

A VPC can add up to 10-15 seconds to your function's cold start. Yes you read that right.
When you do this, an Elastic Network Interface (ENI) must be created, 
an IP address must be allocated to the ENI, then the ENI is attached to the Lambda. Definitely read up on the [AWS documentation](https://docs.aws.amazon.com/lambda/latest/dg/vpc.html) on this.

It's important to note that putting your Lambda inside doesn't make your Lambda more secure unlike it does for EC2.
For EC2 you need a VPC to shield your machine from malicious traffic.
Lambda functions are protected by AWS Identity and Access Management (IAM) service, which provides both authentication and authorization. 
All incoming requests have to be signed with a valid AWS credential, and the caller must have the correct IAM permission.

So why would you use a VPC then?
- You need to access Amazon RDS, ElastiCache, ... 
- You need a Private AWS API Gateway
- You need access to on-premise resources over VPN/Direct Connect

In the [AWS documentation](https://docs.aws.amazon.com/lambda/latest/dg/best-practices.html#lambda-vpc) you can find a flowchart to help you decide when to use a VPC:  

 ![AWS Lambda VPC decision tree]({{ site.url }}/img/post/coldstart-vpc-decision.png "AWS Lambda VPC decision tree")

At re:Invent 2018 there was an excellent deep dive session - [A Serverless Journey: AWS Lambda Under the Hood (SRV409-R1)](https://www.youtube.com/watch?v=QdzV04T_kec) - in which they announced a new architecture for VPC Lambda's that should solve the slow starts.
This should be released in 2019 but hasn't happened so far.

**Current architecture:**  
![AWS Lambda VPC architecture]({{ site.url }}/img/post/coldstart-vpc-2018.png "AWS Lambda VPC architecture")

**Future architecture:**  
![AWS Lambda VPC architecture 2019]({{ site.url }}/img/post/coldstart-vpc-2019.png "AWS Lambda VPC architecture 2019")


### 2. Keep your function small and single purposed
Let your function do one thing and let it do it well. 
This helps you keep the codebase small and clean, which is a good design practise as well.
You can't load what you don't have.


### 3. Minimize your function's dependencies and use lightweight frameworks
Try to minimize the dependencies you use and be critical about the ones you end up choosing.  

Prefer lightweight frameworks over (slow) feature rich frameworks that contain a lot of things you don't need.
Your default toolbox as a Java developer most likely contains Spring, even though Pivotal is working hard to optimize startup, it's currently still one of the slower options.

When integrating to AWS services like S3, DynamoDB and so on it helps to use the [AWS SDK for Java 2.x](https://aws.amazon.com/blogs/developer/aws-sdk-for-java-2-x-released/) over the older one. 
This can shave off a few hundred milliseconds your startup. AWS provides a no-dependency lightweight HTTP client that starts up faster than the default Apache HTTP client.

Most of my Lambda's are built with Dagger (Dependency Injection) and Gson (JSON parsing, faster than Jackson) and a few extras.
This has served me well so far but I'm evaluating newer frameworks like Quarkus and Micronaut that have been built with fast start ups and low memory environments in mind.


### 4. Prefer low overhead runtimes
Dare to ask if your services could be implemented in a 
language that has less problems with slow cold starts like nodejs, Python or Go. 

Java is still my go-to language but functions should be small services, which could help in experimenting with other language.
Nowadays the JavaScript quirks are much less when using nodejs with TypeScript. 
This could be a good candidate for a Java developer to experiment with.


### 5. Choose the right memory settings
The higher you set the memory settings, the more you are billed for execution time.
The important remark is, when you double your memory on AWS Lambda you get **more** than double the CPU capacity.
This could possibly mean your Lambda will execute faster and it'll costs less than you would have with lower memory settings.

It's crucial to fine-tune and find the best memory settings for your function. A tool like [AWS Lambda Power Tuning](https://github.com/alexcasalboni/aws-lambda-power-tuning) could help you do this automatically.

## Real function cold starts examined
When comparing different languages and frameworks, it's important to do more than a `Hello World` example. 
Especially in Java adding one dependency could potentially have a big impact on your start up already depending on what gets initialized.

As a simple example I will use a function that has one HTTP endpoint `addProduct`, which will add a new product in a database. 
We will use DynamoDB since it's very easy to setup and integrate from a Lambda. We will use X-Ray to examine the cold start timing of a few different examples.

![Function example]({{ site.url }}/img/post/coldstart-example.png "Function example")

When inspecting your Lambda with AWS X-Ray you can get a good understanding of the different phases of the instance start up.
After your function was triggered, you can check CloudWatch Logs to see how long your function ran and how much you time you are billed for:    
`REPORT Duration: 2170.02 ms  Billed Duration: 2200 ms  Memory Size: 1024 MB  Max Memory Used: 158 MB`

The most important fact I learned while doing this, is that the reported duration is **NOT** how long your function needed to actually complete. 
This is the time how long your code inside the handle method needed to finish. All the time your function needed to initialize like logic 
in your class `Constructor`, `static initializers` and `global variables` are not shown in the reported duration. 
When a framework does all it's heavy lifting in the constructor, you could potentially see a low reported duration in CloudWatch but actually duration could've been much higher.
This is why it's important to compare actual end to end duration instead of the reported duration in CloudWatch to have a fair comparison.

### Java 8
This example is how I usually built Lambda's in the last year.
- Dagger 2: lightweight compile time dependency injection
- Gson: fast startup JSON processing - faster than Jackson
- AWS SDK 2.0: Integration with DynamoDB + lightweight AWS HTTP client instead of the Apache default


Cold start: **3.9 seconds**  

![Lightweight Java]({{ site.url }}/img/post/coldstart-java-lightweight.png "Lightweight Java")
```
CloudWatch REPORT Duration: 2170.02 ms  Billed Duration: 2200 ms  Memory Size: 1024 MB  Max Memory Used: 158 MB	
```


### Java 8 - Micronaut 
This example I got to play around with Micronaut, a young new promising Framework that also has good support for Serverless applications.
- Micronaut 1.x
- AWS SDK 2.0: Integration with DynamoDB + lightweight AWS HTTP client

Cold start: **5.3 seconds**  

![Micronaut]({{ site.url }}/img/post/coldstart-java-micronaut.png "Micronaut")
```
CloudWatch REPORT Duration: 3210.99 ms  Billed Duration: 3300 ms  Memory Size: 1024 MB  Max Memory Used: 172 MB	
```

## How to push the boundries?
What if you need more? This year an interesting feature has been launched allowing [Custom AWS Lambda Runtimes](https://docs.aws.amazon.com/lambda/latest/dg/runtimes-custom.html) to run any language you want.
That in combination with the growing popularity of GraalVM makes an interesting mix.

### What is GraalVM?

### How to get started?

### GraalVM cold start examined

### Limitations

## Conclusion


### Node.js
Just to broaden your perspective I would like to add one optimized node.js example:

Cold start: **539 milliseconds**  

![node.js]({{ site.url }}/img/post/coldstart-nodejs.png "node.js")
``` 
CloudWatch REPORT Duration: 69.36 ms  Billed Duration: 100 ms  Memory Size: 1024 MB  Max Memory Used: 106 MB	
```

**Ouch**. This is why I really encourage you to investigate and experiment with node.js (or Python/Go, but that might be a bigger leap as a Java developer).
I understand this is not always an option, depending on your company requirements and existing tools/frameworks that need to be integrated.
