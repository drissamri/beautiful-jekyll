---
layout: post
title: Getting started with TLS/SSL (HTTPS) in Java
author: Driss Amri
date: 2017-02-15
tags:
 - Java
 - TLS
 - SSL
 - Security
---

If you are running a site and it is still not using HTTPS, shame on you. There are several reasons why you should start enabling HTTPS, even if you are not transmitting sensitive data. In 2014, Google announced that [HTTPS was a ranking signal](https://webmasters.googleblog.com/2014/08/https-as-ranking-signal.html) for it's search results. Protecting your user's data, higher ranked in Google AND that nice green bar infront of your website? On top of that, now [HTTP/2](https://en.wikipedia.org/wiki/HTTP/2) is starting to show up in more platforms which gives users a nice speed increase, it usually is still only supported on HTTPS for now. What's keeping you, partner? 

It used to be expensive and tedious process getting a SSL certificate for your domain. If your site was not dealing with credentials or sensitive personal data you could say it wasn't worth the effort. Nowadays, this is no longer an excuse not to get a certificate. With providers like [SSLMate (paid)](https://www.sslmate) and initiatives like [Let's Encrypt (free)](https://letsencrypt.org/) creating your certificate this a trivial task. Since Let's Encrypt is a fully automated and free solution to get your certificates, a lot of people have started using it.

In this post I will link to several smaller posts on HTTPS and certificates. First let's get a bit of terminology out of the way that you might see in these posts.

### Terminology
#### TLS vs SSL
TLS is the successor to SSL. It is a protocol that ensures privacy between communicating applications. Unless otherwise stated, in this document consider TLS and SSL as interchangable. The most secure and recent version is TLS v1.3.

#### Certificate
You will see a lot of information about public and private key pairs when reading up on HTTPS/TLS. The certificate is the public key part, which can be freely giving to anyone.

#### Private Key
A private key can verify that its corresponding certificate/public key was used to encrypt data. It is never given out publicly.

#### Certificate Authority (CA)
A company that issues digital certificates. For SSL/TLS certificates, there are a small number of providers like Symantec, Versign, Comodo, GoDaddy, LetsEncrypt, whose certificates are included by most browsers and Operating Systems. They serve the purpose of a “trusted third party”.

#### Trust store
Each client (browser, OS, JDK, etc.) ships with a list of trusted CAs. This list of trusted CA's and their public certificates are stored, this collection is usually referred to as a Trust Store.

#### SSL Certificate Chain
All SSL/TLS connections rely on a chain of trust called the SSL Certificate Chain. Part of PKI (Public Key Infrastructure), this chain of trust is established by certificate authorities (CAs) who serve as trust anchors that verify the validity of the systems being communicated with.

### How does it work?

When you make a request to [https://www.google.be](https://www.google.be), you are indicating you want to start a secured HTTP connection. The server will send its **certificate (Public Key)** to your browser and your browser will verify if this certificate is trusted. To verify the server certificate, your browser will verify  if the certificate, or one of its parent certificates (certificate chain) is found in its Trust Store. Here's a screenshot from my OSX trust store, that is used by Chrome.

![Chrome Trust Store]({{ site.url }}/img/post/osx-keystore.png "Chrome Trust Store")

You can see I highlighted the **GeoTrust Global CA** certificate, this is the Certificate Authority used to validate my secure connection to Google: 

![Google certificate chain]({{ site.url }}/img/post/https-google.png "Google certificate chain")

This process is called **one-way SSL** authentication*, in which only the server has to be validated for authenticity. This is what you will come across most of the time. You also have **two-way SSL**, in that case the client also has to have a certificate and send it to the server so it can validate it as well.

### Certificates in Java

The JDK ships with a tool called [Keytool](http://docs.oracle.com/javase/6/docs/technotes/tools/solaris/keytool.html) to create certificates or manipulate key stores/trust stores. For some of the most commonly used commands go check out this [site](https://www.sslshopper.com/article-most-common-java-keytool-keystore-commands.html). 

Every JDK has it's own keystore, which contains all certificates authorities that it trusts (also referred to as truststore). This trust store is stored as `cacerts` under your `JDK_DIR/jre/lib/security/`. On my Mac this is located at `/Library/Java/JavaVirtualMachines/jdk1.8.0_60.jdk/Contents/Home/jre/lib/security`
 The default password for this keystore is `changeit`. You can have look with the keytool with the following command: 

```shell
 keytool -list -keystore cacerts -storepass changeit
```

If you want to override the default truststore with your own, you can do this in two different ways.

1. System property `javax.net.ssl.trustStore` that refers to your custom keystore
2. `java-home/lib/security/jssecacerts` if this file exists, it takes precedence over `cacerts`
3. If none of the above are present, the default `cacerts`is used.

### Tell me more

 
* Generate self-signed HTTPS certificates
* Generate Let's Encrypt HTTPS certificates
* Let's Encrypt certificates in Java cause SunCertPathBuilderException
* Enabling HTTPS on Cloud Foundry (Bluemix)
