---
layout: post
title: Trusting Let's Encrypt SSL certificate in a Java Spring application
author: Driss Amri
date: 2017-02-16
tags:
 - Java
 - SSL
 - TLS
 - Security 
 - Spring Boot
 - X509
 - Let's Encrypt
 - Certificate
---

You are trying to make a HTTP call to an HTTPS endpoint, that is using `Let's Encrypt` certificate in Java. This example is using Spring's RestTemplate, but this could be plain old Java, OkHttp or any other HTTP client implementation:

```java
 @RunWith(SpringRunner.class)
 @SpringBootTest
 public class SslApplicationTests {
    @Autowired
    private RestTemplate restTemplate;

    @Test
    public void callHTTPSWithLetsEncryptCertificate() {
        restTemplate.getForEntity("https://valid-isrgrootx1.letsencrypt.org/", String.class);
    }
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

This means that the Java JDK truststore, by default `cacerts` in your `JDK/jre/lib/security` folder,  does not contain the necessary certificate(s) to trust Let's Encrypt. This can happen because Let's Encrypt has not been a Certificate Authority for a long period of time.

The required `DST Root CA X3` certificate was added with versions JDK 7u111+ and JDK 8u101+, so you're probably using an older or different version of the JDK. You can manually add the necessary certificate(s) in your JDK truststore. You can find the up to date information about the current certificates in the [Let's Encrypt documentation](https://letsencrypt.org/certificates/). At the time of publishing this post, the active root certificate is called `ISRG Root X1.der`. If you are not able to upgrade to a new JDK, you can add the required certificate to you truststore manually or automatically.

## Manually adding a trusted certificate

The `cacerts` truststore that contains all trusted SSL certificates, directly or indirectly through intermediates, contains the certificate to trust the Let's Encrypt certificates. If you are using a version that does not contain this certificate, you can manually update it. 

The JDK ships with the `keytool` tool with which you can manipulate and read Java keystores. First download the current `isrgrootx1.der` certificate. Next add this certificate to your JDK `cacerts` truststore:

```shell
keytool -trustcacerts \
    -keystore $JAVA_HOME/jre/lib/security/cacerts \
    -storepass changeit \
    -noprompt \
    -importcert \
    -file isrgrootx1.der
```

## Automated truststore update

I found a bash script on GitHub and modified it a little bit to download the Let's Encrypt root certificate and cross-sign certificates and puts them in your JDK's `cacerts`. (Mac OSX)

```bash
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

Save this script as `install-certs.sh`. You can call this script using the following command: `sh install-certs.sh $(/usr/libexec/java_home -v '1.8*')`. Fill in the 1.8* with your current JDK version if you have multiple JDK 1.8.X on your machine, or if you are using an older JDK. After running this script and restarting your Java application, it should work fine without the certificate error.

## Package a custom truststore with the certificate in your application

If you are unable to upgrade the JDK, and not able to modify the server's JDK truststore (e.g. on a cloud environment) you can always package a custom truststore in your application. Either copy the cacerts from a local newer JDK or use the automated way above and package the resulting `cacerts` with your WAR/JAR. Here's an example of creating a RestTemplate using a custom cacerts truststore that is available on the classpath:

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

First I load in the truststore in memory, then I add it as the truststore in a new SSLContext. Finally I add the SSLContext to the HttpClient



