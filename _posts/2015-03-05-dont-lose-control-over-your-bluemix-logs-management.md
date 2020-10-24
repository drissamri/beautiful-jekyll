---
layout: post
title: "Don't lose control: Bluemix logs management"
permalink: /blog/cloud/dont-lose-control-over-your-bluemix-logs-management/
date: 2015-03-05 20:47:05
tags:
 - Bluemix
 - Cloud Foundry
 - DevOps
 - Papertrail
 - Loggregator
---
Lately I have been busy deploying more Cloud based applications, mostly on [Bluemix](https://console.ng.bluemix.net/). I try to following a [Microservices](http://www.drissamri.com/blog/architecture/what-are-microservices/) (or Service Oriented) Architecture approach so I have multiple applications working together.
 At a certain point following up on the status of these systems should not be a manual job anymore. This is why I decided take a first step to managing my infrastructure better by centralizing my all my log files in one system.

There are quite a few logging solutions out there. From the super popular [ElasticSearch/Logstash/Kibana (ELK)](http://www.elasticsearch.org/overview/elkdownloads/) open-source stack to online Software-as-a-Service (SaaS) log management systems. Today, I wanted to get started quickly with my Bluemix logs management so I decided to try out one of these SaaS logging services, but in the future I will also be playing around with the ELK stack.

If you are using WebSphere Liberty or node.js on Bluemix, you can easily get started with the provided [IBM Monitoring and Analytics](https://www.ng.bluemix.net/docs/#services/monana/index.html) service.
 This will not work for me since I'm working with my [Linkshortener](http://www.drissamri.com/blog/rest/building-your-own-linkshortener-api/) application, which is a Spring Boot application that runs on an embedded Tomcat application server.

I decided to take a look around to see what kind of other online services are available, and after quickly Googling I was directed to plenty of options:

*   [Logentries](https://logentries.com/)
*   [Loggly](https://www.loggly.com/)
*   [Papertrail](https://papertrailapp.com/)

Since Bluemix is based upon Cloud Foundry, I went to take a look at the Cloud Foundry documentation because it is so extensive and usually points you in the right direction. There is quite some information available about log management which is handled by their [Loggragator](http://docs.cloudfoundry.org/devguide/deploy-apps/streaming-logs.html) which is very capable of handling [3th party logging services](http://docs.cloudfoundry.org/devguide/services/log-management.html). It even has instructions about how to integrate your logging with certain [Third-Party Log management services](http://docs.cloudfoundry.org/devguide/services/log-management-thirdparty-svc.html).

I decided to give Papertrail a shot, but definitely will try out different services out in the future.

### Step 1: Setup your Papertrail account

Just go to [https://papertrailapp.com/plans](https://papertrailapp.com/plans) and sign up for your desired plan. I just went with their free account which lets me hold my logs for 7 days, which is sufficient for me, for now.

Add a system to your account.

![Choose to add a system]({{ site.url }}/img/post/papertrail-add.png)

Now you will see your Papertrail endpoint, which you will need to write down since it will be needed in a bit to point your logging service in Bluemix to this endpoint. Since we are using Bluemix, choose the alternatives logs method.

![Your Papertrail endpoint URL]({{ site.url }}/img/post/papertrail-endpoint.jpg)

Here you can choose Heroku, which is another online Platform-as-a-Service which is comparable to Bluemix. This option will also work for Bluemix and Cloud Foundry. So choose this option and give it a name.

![Pick the Heroku option, which works in the same way as Bluemix and Cloud Foundry]({{ site.url }}/img/post/papertrail-cf.jpg)

That's all there is to it in Papertrail. Your account is now ready to receive logs.

### Step 2: Create a logging service in Bluemix

You can't create the logging service through the web dashboard, so start up your console so you can work with the CF command line interface.
 We will need to create a User-Provided Service, or also known as cups (create-user-provided-service), this is done by issuing the following command: `cf cups SERVICE_NAME -l syslog://PAPERTRAIL_HOST:PORT`.

{% highlight bash %}
 cf cups papertrail syslog://logs2.papertrailapp.com:10163
{% endhighlight %}

### Step 3: Bind the new logging service to your application

Now your service is created and ready to be used by your applications. Now add this service to your application by using the Dashboard or the CF command line again: `cf bind-service APPLICATION_NAME SERVICE_NAME`

{% highlight bash %}
 cf bind-service linkshortener papertrail
 {% endhighlight %}

After this you will be asked to restage or repush your application so the service is binded to your application. You can restage your application with the following command: `cf restage APPLICATION_NAME`

{% highlight bash %}
 cf restage linkshortener
 {% endhighlight %}

## Bluemix logs management

When you completed the previous steps, as soon as your application starts logging it will be forwarded to Papertrail and it should appear in your dashboard.

![Papertrail dashboard]({{ site.url }}/img/post/papertrail-dashboard.jpg)

This is all you need to do to centralize your log files for easier management and control of your systems. If you have any suggestions for other online logging services or any tips or tips do let me know!
