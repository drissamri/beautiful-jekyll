---
layout: post
title: Microservices! Micro.. what?
permalink: /blog/architecture/what-are-microservices/
date: 2014-11-17 13:25:32
tags:
 - Java
 - Microservice
 - SOA
---
You've probably heard the term, ahum buzzword, `microservice` quite a lot lately. I'll quickly go over what this actually means. In in a series of future posts I'll be building a microservice and see how we can facilitate continuous delivery for these services.

## What is a microservice?

A microservice is basically a service like we've known it for years. This should be no shock to people who have heard about Service Oriented Architectures (SOA) before. What people usually try to express when they talk about microservice, are services that are small and focused on a specific feature. This means we'll have several lightweight services communicating with each other, each focusing own its specific feature and/or task.

This architecture gives us with a lot of benefits like make the service more maintainable and flexible. Developers are able to reason much better about these microservices compared to their traditional monolithic counterparts. It also makes the services much more scalable, without wasting to much unnecessary resources. If a certain part of your application is getting a traffic spike or is having other issues, it will be easy to just launch another instance of that service to share the load.

Like any other architecture or framework, it is no silver bullet. There are obvious down sides too. By implementing this architecture, you'll be handling much more deployment units and will be spending more time on maintaining them. This is why the DevOps movement is gaining much traction nowadays, because these downsides can be mitigated partly by automating as much as possible. Besides automating, it is also very important to get a good monitoring system in place so you don't have to check each of these services separately.

### Do we want this?

I do believe this approach makes a lot of sense in a lot of scenarios. There is no specific technology needed to get start building these kind of microservices, it's more about shift in how we think about developing and managing services. It's important to think and decide before hand how we want to split our services up in various lightweight independent services. If you take this approach you'll be dealing with more projects, this means you'll be doing the same default configuration to setup a web application over and over again. We want to minimize that overhead. This is where a few frameworks in the JVM space have shown up and try to get you started as fast as possible. Today we'll have a quick go with Spring boot, which is part of the Spring framework. This is not the only framework out there of course, Dropwizard and Play! 2 are other popular frameworks out there today.

## Getting started with Spring Boot

Spring Boot is a cool Spring framework that really simplifies getting started with your application. It favors convention over configuration and is designed to get you up and running as quickly as possible. They will try to configure as much as possible automatically depending on what dependencies and properties you have on your classpath based on best practises. This means you are able to get up and running really quickly with your application and focus on your business logic. This great for people who don't have much Spring experience, but even if you are a Spring veteran, it will vastly improve your developer experience.

### How to get started

To get started rapidly with Spring boot you can hop over to their [Initializr application](http://start.spring.io/), which helps you bootstrap your application Maven pom.xml or Gradle file. You are able to select a lot of Spring dependencies, which will automatically be added to your build file. For each of these dependencies, Spring boot will try its best to configure them so you don't have to worry too much about it when you are just getting started.

It is interesting to see that you can choose to do a web application and package it as a .jar file. If you choose this path, Spring will launch an embedded container (default Tomcat) which makes it much easier to debug your application straight in your IDE without configuring a container. Forget about starting up your application server and deploying to it, you can just right click your application and click on "**Run**"

![Spring Initializr Application]({{ site.url }}/img/post/spring-initializr.jpg)

When you are done filling in details about your application, you can download the bootstrapped application. This will give you a Maven project following the standard conventions.

There are a few things to point out. In the pom.xml two things will stand out, first there is a reference to a parent pom **spring-boot-starter-parent** . This pre-configures a lot of things for us, it also manages the version numbers of Spring and quite a few other frameworks to avoid getting trapped in the so called dependency hell. Everything that is configured out of the box can be modified with your needs. There is also a plugin defined **spring-boot-maven-plugin** which helps us create an executable .jar file that instead of the traditional .war file. If you execute this jar file it will start up a embedded container, default is Tomcat, which will host your project. This makes your project very portable and also gives your application its own Java process which you can manage better.

**pom.xml**
{% highlight xml %}
 <!-- Inherit defaults from Spring Boot -->
 <parent>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-parent</artifactId>
  <version>1.2.0.M2</version>
  <relativePath/>
 </parent>

<build>
 <plugins>
  <plugin>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-maven-plugin</artifactId>
 </plugin>
</build>
{% endhighlight %}

There's also an `Application.java` which has a main method, this is the entry point for our application. When you run this class the embedded Tomcat will start up. This works out of the box, only there isn't any content to serve yet so this doesn't do much yet. The `@Configuration` and `@ComponentScan` we already know from any other Spring application, the only new thing here is the `@EnableAutoConfiguration` annotation which is specific to Spring Boot. This annotation will try to auto configure as much as possible, also depending on what dependencies it finds on the classpath. If you have the HSQLDB as a maven dependency, it will auto configure a embedded HSQLDB database with default settings. This feature works for a lot dependencies, but of course you can always inject your Spring beans to have complete control. You can also just customize the existing auto configured beans by using the default application.properties file. The [Spring documentation](http://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/) is pretty extensive so make sure to take a look.

**Application.java**
{% highlight java %}
@Configuration
@ComponentScan
@EnableAutoConfiguration
public class Application {

public static void main(String[] args) {
 SpringApplication.run(Application.class, args);
 }
}
{% endhighlight %}

### Going for a test run

Let us add a simple Controller that will act as a REST endpoint for our application just to see that we can actually start this up as an application and see some sort of output. We will annotate the controller with the Spring 4 `@RestController` annotation that basically annotates all methods with `@ResponseBody` to specify that we'll be returning JSON objects.

**LinkController.java**
{% highlight java %}
import static org.springframework.http.MediaType.APPLICATION_JSON_VALUE;
import static org.springframework.web.bind.annotation.RequestMethod.*;

@RestController
@RequestMapping(LinkController.LINKS)
public class LinkController {
public static final String LINKS = "/links";

@RequestMapping(method = GET, produces = APPLICATION_JSON_VALUE)
 public ResponseEntity<String> find() {
  String result = "Welcome to Spring Boot";

  return ResponseEntity.ok().body(result);
 }
}
{% endhighlight %}

As soon as you've done this, you can go to the Application.java and right click on it and choose to **Run** it directly from your IDE. If you open your browser and go to `http://localhost:8080/links` you'll see the output `Welcome to Spring boot`.

That's how easy it is to get up and running with a web application nowadays. Because it is so easy to get up and running, it is perfect to get started building your own so called microservices. Hope this post was useful, if you have any questions or remarks don't hesitate to let me know!
