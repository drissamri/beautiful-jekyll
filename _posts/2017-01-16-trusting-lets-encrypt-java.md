---
layout: post
title: Trusting Let's Encrypt SSL certificate in a Java Spring application
author: Driss Amri
date: 2017-02-15
tags:
 - Java
 - SSL
 - Security 
 - Spring Boot
 - X509
 - Let's Encrypt
---

You are opening a connection to a HTTPS resource that is using a [Let's Encrypt (free)](https://letsencrypt.org/) SSL certificate, like the following example:

```java
    public void callLetsEncryptEndpointWithHTTPS() {
      restTemplate.getForEntity("https://valid-isrgrootx1.letsencrypt.org/", String.class);
    }
```

This sample code is with Spring's RestTemplate, but this could be the standard JDK HTTP infrastructure, OkHttp, Apache HttpClient or any other framework. The following exception is thrown:

```java
sun.security.provider.certpath.SunCertPathBuilderException: unable to find valid certification path to requested target
    at sun.security.provider.certpath.SunCertPathBuilder.engineBuild(SunCertPathBuilder.java:196) ~[na:1.7.0_79]
    at java.security.cert.CertPathBuilder.build(CertPathBuilder.java:268) ~[na:1.7.0_79]
    at sun.security.validator.PKIXValidator.doBuild(PKIXValidator.java:380) ~[na:1.7.0_79]
    at sun.security.validator.PKIXValidator.engineValidate(PKIXValidator.java:292) ~[na:1.7.0_79]
    at sun.security.validator.Validator.validate(Validator.java:260) ~[na:1.7.0_79]
    at sun.security.ssl.X509TrustManagerImpl.validate(X509TrustManagerImpl.java:326) ~[na:1.7.0_79]
```

This happens because Let's Encrypt's certificates are pretty recent and not available in all applications and software. In this case, the JDK's truststore `cacerts`does not contain the necessary certificate to open a secure HTTP connection. This truststore is located in `$JAVA_HOME/jre/lib/security/cacerts`. The issue [JDK-8154757](https://bugs.openjdk.java.net/browse/JDK-8154757) was raised to support Let's Encrypt certificates in the JDK and this ticket has been implemented in JDK 7u111+ and JDK 8u101+. If you are getting this issue, it means you are using an older or different version of the JDK.





If you are running a site and it is still not using HTTPS, shame on you. There are several reasons why you should start enabling HTTPS, even if you are not transmitting sensitive data. In 2014, Google announced that [HTTPS was a ranking signal](https://webmasters.googleblog.com/2014/08/https-as-ranking-signal.html) for it's search results. Protecting you users data, higher ranked in Google AND that nice green bar infront of your website? What's keeping you, partner? 

It used to be expensive and tedious process getting a SSL certificate for your custom domain. If your site was not dealing with credentials or sensitive personal data you could say it wasn't worth the effort. Nowadays, this is no longer an excuse not to get a certificate. With providers like [SSLMate (paid)](https://www.sslmate) and initiatives like [Let's Encrypt (free)](https://letsencrypt.org/) make requesting your certificate this a non-trivial task. Since [Let's Encrypt](https://letsencrypt.org/) is a fully automated and free solution to get your certificates, a lot of people have started using it. 

Since [Let's Encrypt](https://letsencrypt.org/) has only been around 2014, it's taking time before all applications and software trusts their certificates. The support nowadays is pretty widespread, but especially with older applications you can run into problems. A few weeks ago colleagues were working on a Java application, which was using Spring Boot ran into issue due to this. They were invoking a HTTPS Authentication Service on Amazon, which was using [Let's Encrypt](https://letsencrypt.org/) HTTPS certificates using Spring's RestTemplate: 

You are making a HTTP call to a HTTPS endpoint that is using a `Let's Encrypt` certificate in Java. This example is using Spring's RestTemplate, but this could be plain old Java, OkHttp or any other HTTP client implementation:

```java
    public void callLetsEncryptEndpointWithHTTPS() {
      restTemplate.getForEntity("https://valid-isrgrootx1.letsencrypt.org/", String.class);
    }
```

If you get the following error:

```java
sun.security.provider.certpath.SunCertPathBuilderException: unable to find valid certification path to requested target
    at sun.security.provider.certpath.SunCertPathBuilder.engineBuild(SunCertPathBuilder.java:196) ~[na:1.7.0_79]
    at java.security.cert.CertPathBuilder.build(CertPathBuilder.java:268) ~[na:1.7.0_79]
    at sun.security.validator.PKIXValidator.doBuild(PKIXValidator.java:380) ~[na:1.7.0_79]
    at sun.security.validator.PKIXValidator.engineValidate(PKIXValidator.java:292) ~[na:1.7.0_79]
    at sun.security.validator.Validator.validate(Validator.java:260) ~[na:1.7.0_79]
    at sun.security.ssl.X509TrustManagerImpl.validate(X509TrustManagerImpl.java:326) ~[na:1.7.0_79]
```

It means your JDK truststore, `cacerts`, does not contain the necessary certificate(s) yet to trust Let's Encrypt. The `DST Root CA X3` certificate was added with versions JDK 7u111+ and JDK 8u101+, so you're probably using an older or different version of the JDK. You can manually add the necessary certificate(s) in your JDK truststore. You can find the up to date information about the current certificates in the [Let's Encrypt documentation](https://letsencrypt.org/certificates/). At the time of publishing this post, the active root certificate is called `ISRG Root X1.der`. 

## Manually adding a trusted certificate

The JDK ships with the `keytool` tool with which you can manipulate and read Java keystores. First download the current `isrgrootx1.der` certificate. Next add this certificate to your JDK `cacerts` truststore:

```shell
keytool -trustcacerts \
    -keystore $JAVA_HOME/jre/lib/security/cacerts \
    -storepass changeit \
    -noprompt \
    -importcert \
    -file isrgrootx1.der
```

This means the `cacerts` truststore that contains all trusted SSL certificates, directly or indirectly through intermediates, contains the certificate to trust the Let's Encrypt certificates. If you are using a version that does not contain this certificate, you can manually update it. I found a bash script on GitHub and modified it a little bit to download the Let's Encrypt root certificate and cross-sign certificates and puts them in your JDK's `cacerts`. 

```shell
#!/bin/bash
set -e

JAVA_HOME=${1-text}
[ $# -eq 0 ] && { echo "Usage: sudo $0 \$(/usr/libexec/java_home -v '1.8*')" ; exit 1; }

KEYSTORE=$JAVA_HOME/jre/lib/security/cacerts

wget https://letsencrypt.org/certs/isrgrootx1.der
wget https://letsencrypt.org/certs/lets-encrypt-x3-cross-signed.der
wget https://letsencrypt.org/certs/lets-encrypt-x4-cross-signed.der

# to be idempotent
keytool -delete -alias isrgrootx1 -keystore $KEYSTORE -storepass changeit 2> /dev/null || true
keytool -delete -alias letsencryptauthorityx3 -keystore $KEYSTORE -storepass changeit 2> /dev/null || true
keytool -delete -alias letsencryptauthorityx4 -keystore $KEYSTORE -storepass changeit 2> /dev/null || true

keytool -trustcacerts -keystore $KEYSTORE -storepass changeit -noprompt -importcert -alias isrgrootx1 -file isrgrootx1.der
keytool -trustcacerts -keystore $KEYSTORE -storepass changeit -noprompt -importcert -alias letsencryptauthorityx3 -file lets-encrypt-x3-cross-signed.der
keytool -trustcacerts -keystore $KEYSTORE -storepass changeit -noprompt -importcert -alias letsencryptauthorityx4 -file lets-encrypt-x4-cross-signed.der

rm -f isrgrootx1.der lets-encrypt-x3-cross-signed.der lets-encrypt-x4-cross-signed.der
```

You can call this script using the following command: `sh install-certs.sh $(/usr/libexec/java_home -v '1.8*')`. Fill in the 1.8* with your current JDK version if you have multiple JDK 1.8.X on your machine, or if you are using an older JDK. After running this script and restarting your Java application, it should work fine without the certificate error.

```java
   /**
     ** Add custom SSL TrustStore because Let's Encrypt root is not in all JDK stores yet.
     ** This is the cacerts from JDK7.0_79 and manually added root isrgrootx1.der
     */
    @Bean
    public RestTemplate restTemplate() throws Exception {
        ClassPathResource classPathResource = new ClassPathResource("cacerts");
        SSLContext sslContext = SSLContexts
            .custom()
            .loadTrustMaterial(classPathResource.getFile())
            .build();
        final CloseableHttpClient client = HttpClients
            .custom()
            .setSSLContext(sslContext)
            .build();
        return new RestTemplate(new HttpComponentsClientHttpRequestFactory(client));
    }
```


https://gist.github.com/drissamri/70dc58a7b736e71086aac8f23b172b71