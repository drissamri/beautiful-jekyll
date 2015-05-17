---
layout: post
title: Modern Java web applications with Spring Boot, Thymeleaf and AngularJS
permalink: /blog/technology/modern-java-web-applications-spring-boot-thymeleaf-angularjs/
date: 2015-01-26 22:10:22
tags:
 - Java
 - AngularJS
 - Spring Boot
 - Thymeleaf
---
I've already blogged about [Spring Boot](http://www.drissamri.be/blog/architecture/what-are-microservices/) and how easy it makes your life as a Spring developer, if you haven't looked into it you really should stop now and do that first.

If you're using Java in an enterprise environment, you've most likely been using JSP quite a bit which does the job pretty well. I'm here to show you there are other templating technologies you can use that have much more modern features. Today I'll use [Thymeleaf](http://www.thymeleaf.org/), a templating engine which has been around for quite a while. It has flawless support for Spring MVC and Spring Security, which plays a big role in its success.

Now I don't want to create a boring old completely server-side rendered page, we want to be hip and trendy and minimize the page refreshes so this is why I'll introduce [AngularJS](https://angularjs.org/), a self claimed _Superheroic JavaScript MVW Framework_. For minimal styling I'll use [Bootstrap](http://getbootstrap.com/).

It's pretty likely you have heard about AngularJS since it's been extremely popular last year and it doesn't seem like it's going away anytime soon. (and no, v2.0 won't change a thing). It's created and maintained by Google and it's completely open source. It's not only for hipsters since the enterprise is grabbing on and it's gaining a lot of traction, or at least in Belgium it is. If you haven't played around with it, make sure you do since it's a nice skill to have in your toolbox.

With all these popular JavaScript frameworks nowadays, you'll see they are accompanied by new tools like:

*   [Bower](http://bower.io/): package manager to manage your dependencies
*   [Gulp](http://gulpjs.com/) or [Grunt](http://gruntjs.com/): Task runners to help your build process

These tools are really great and are evolving at a extremely fast pace, but not everybody wants to get more technologies and complexity involved into their development process. A lot of Java developers already have tools in their toolbelt that offer similar features, like Maven and Gradle. For these people there's a great alternative to have more control over your frontend assets like you do over your Java dependencies: [WebJars](http://www.webjars.org/).

**WebJars** allow you to manage frontend assets (AngularJS, Bootstrap, jQuery, ...) and include them in your project as **Java Archive (JAR)** Maven dependencies. This gives you the advantages of staying with your known build environment and have the benefits of versioning your frontend assets like would (and should) do with your Java libraries. If you're still including jQuery and such as static files in your projects, shame on you! You wouldn't dare to do that for any Java library, would you? If you are using Spring Boot, you can get this working without any configuration at all! Most libraries are already packaged for you and you can just take a look at their website to see if you can find what you need, if it's not there, it's pretty easy to create a package yourself and share it with the community.

### Setup Spring Boot

You can get a skeleton project by going to the [Spring Boot Initializr](http://start.spring.io/) and choosing the **Web** as Project Dependency and **Thymeleaf** under Template Engines. You can see I've added AngularJS and Bootstrap dependencies in my pom.xml, using WebJars.

{% gist drissamri/66a3c559261fe23250c8 %}

Once this is done, you can start creating your `Application.java` which is the entry point of the Spring Boot application.

{% gist drissamri/19ac28ec2c752ae1b527 %}

Next you can add a `HomeController.java` which will be map the root of the server to an `index.html` page that we'll create right away.
{% gist drissamri/7427776dfe16cac7a77e %}

Let's put all our files in one of the default resource folders in Spring Boot: `src/main/resource/public/`. In this folder I created subfolders for `js`, `css` and `templates` (Thymeleaf templates)

A few things to note about our index.html using Thymeleaf and HTML. We included Bootstrap and AngularJS as WebJars in Maven, but how can we actually use them? With Spring Boot and Thymeleaf this is extreme trivial. Using a normal `<link>` tag, we'll use `[th:href](http://www.thymeleaf.org/doc/articles/standardurlsyntax.html)` instead of the normal `href` and the `@{}` syntax we can refer to the webjar defined that has been retrieved by Maven. For static prototyping a normal href is still added refering to the same file but on a Content Delivery Network (CDN), so if you are viewing statically HTML without running in the Spring Boot application you still get a working page. This is a huge benefit by using Thymeleaf you can easily change your prototype and make sure that your page always as it will be when you serve it on a container.

`<link th:href="@{/webjars/bootstrap/3.3.1/css/bootstrap.min.css}" href="http://cdn.jsdelivr.net/webjars/bootstrap/3.3.1/css/bootstrap.css" rel="stylesheet" media="screen">`

The same is done for Bootstrap as a stylesheet. For now this is the only thing we are using of Thymeleaf, but there is plenty more to come. Don't forget to add the `xmlns:th="http://www.thymeleaf.org"` to your HTML tag so you can get autocomplete in your editor.

{% gist drissamri/3939ad7d11dd87a52408 %}

This post is not really meant to explain AngularJS, so if you're interested I'll refer you to my buddy [g00glen00b](http://g00glen00b.be/) who has [plenty of articles](http://g00glen00b.be/category/t/?s=angularjs) on AngularJS.

This post shows you how easy it is to manage your frontend assets and use a more modern templating language for your Java application.
