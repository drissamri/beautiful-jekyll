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

If you are running a site and it is still not using HTTPS, shame on you! Even if you are not transmitting sensitive data, there are still several reasons why you should enable HTTPS.

In 2014, Google announced that [HTTPS was a ranking signal](https://webmasters.googleblog.com/2014/08/https-as-ranking-signal.html) for its search results. Protecting your user's data, higher ranked in Google AND that nice green bar in front of your website? Still not convinced? On top of that we see that [HTTP/2](https://en.wikipedia.org/wiki/HTTP/2), which gives users a nice speed increase, is mostly supported in combination with HTTPS for now. What's keeping you, partner? 

It used to be expensive and tedious process getting a SSL certificate for your domain. If your site was not dealing with credentials or sensitive personal data you could say it wasn't worth the effort. Nowadays, this is no longer an excuse not to get a certificate. With providers like [SSLMate (paid)](https://www.sslmate) and initiatives like [Let's Encrypt (free)](https://letsencrypt.org/), creating your certificate this a trivial task. Since Let's Encrypt is a fully automated and free solution to get your certificates, a lot of people are using it.

In this post I will link to several smaller posts on HTTPS and certificates. First let's get a bit of terminology out of the way that you might see in these posts.

### Terminology
#### TLS vs SSL
TLS is the successor to SSL. It is a protocol that ensures privacy between communicating applications. Unless otherwise stated, in this document consider TLS and SSL as interchangable. The most secure and recent version is TLS v1.3.

#### Certificate
You will see a lot of information about public and private key pairs when reading up on HTTPS/TLS. The certificate is the public key part, which can be freely given to anyone.

#### Private Key
A private key can verify that its corresponding certificate/public key was used to encrypt data. It is never given out publicly.

#### Certificate Authority (CA)
A company that issues digital certificates. For SSL/TLS certificates, there are a small number of providers like Symantec, Versign, Comodo, GoDaddy, LetsEncrypt, whose certificates are included by most browsers and Operating Systems. They serve the purpose of a “trusted third party”.

#### Trust store
Each client (browser, OS, JDK, etc.) ships with a list of trusted CAs. This list of trusted CA's and their public certificates are stored, this collection is usually referred to as a Trust Store.

#### SSL Certificate Chain
All SSL/TLS connections rely on a chain of trust called the SSL Certificate Chain. 
This chain of trust is established by certificate authorities (CAs) who serve as trust anchors that verify the validity of the systems being communicated with. If a certificate is signed by a certificate, that on its own turn has been signed by a Certificate Authority, you can trust the certificate because it's in the chain of trust.

![Certificate Chain]({{ site.url }}/img/post/ssl-chain.png "Certificate Chain")


### How does it work?

When you make a request to [https://www.google.be](https://www.google.be), you are indicating you want to start a secured HTTP connection. The server will send its **certificate (Public Key)** to your browser and your browser will verify whether this certificate is trusted. To verify the server certificate, your browser will verify  if the certificate, or one of its parent certificates (certificate chain) is found in its Trust Store. Here's a screenshot from my OSX trust store, that is used by Chrome.

![Chrome Trust Store]({{ site.url }}/img/post/osx-keystore.png "Chrome Trust Store")

You can see I highlighted the **GeoTrust Global CA** certificate, this is the Certificate Authority used to validate my secure connection to Google: 

![Google certificate chain]({{ site.url }}/img/post/https-google.png "Google certificate chain")

This process is called **one-way SSL** authentication*, in which only the server has to be validated for authenticity. This is what you will come across most of the time.
**Two-way SSL** also exists in which the client is also required to have and send a certificate to the server.  This allows the server to validate the client’s certificate.

### Certificates in Java

The JDK ships with a tool called [Keytool](http://docs.oracle.com/javase/6/docs/technotes/tools/solaris/keytool.html) to create certificates or manipulate key stores/trust stores. For some of the most commonly used commands go check out this [site](https://www.sslshopper.com/article-most-common-java-keytool-keystore-commands.html). 

Every JDK has its own keystore, which contains all Certificate Authorities it trusts. (also referred to as truststore). This trust store is stored as `cacerts` under your `JDK_DIR/jre/lib/security/`. On my Mac this is located at `/Library/Java/JavaVirtualMachines/jdk1.8.0_60.jdk/Contents/Home/jre/lib/security`
 The default password for this keystore is `changeit`. By using the following command, you can have a look at the JDK’s keystore:

```shell
 keytool -list -keystore cacerts -storepass changeit
```

If you don't want use the default trust store of your JDK, there are several ways you override the default behaviour. 

1. Start your Java application with a System property or Environment variable `javax.net.ssl.trustStore=<LOCATION TO YOUR CUSTOM TRUSTSTORE>` that refers to your custom keystore
2. Create a trust store named `jssecacerts`, with `Keytool`, in your `java-home/lib/security` folder. If this file exists, it takes precedence over `cacerts`
3. If none of the above are present, the default `cacerts`is used.

### Tell me more
 
* Generate self-signed HTTPS certificates
* Generate Let's Encrypt HTTPS certificates
* [Let's Encrypt certificates in Java throw SunCertPathBuilderException]({{ site.url }}/blog/2017/02/16/trusting-lets-encrypt-java/)
* Enabling HTTPS on Cloud Foundry (Bluemix)
