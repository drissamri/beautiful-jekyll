---
layout: post
title: Java keystore and keytool essentials
author: Driss Amri
date: 2017-02-22
excerpt: keytool is a tool to easily view and manipulate keystores and certificates in Java.
tags: 
 - Java
 - TLS
 - SSL
 - Security
 - keytool
---

1. [Getting started with HTTPS: Overview]({{site.url}}{% link _posts/2017-02-22-getting-started-https.md %})
2. **[Getting started with HTTPS: Java keystore and keytool essentials]()**
3. [Getting started with HTTPS: Let's Encrypt on Cloud Foundry and Bluemix]({{site.url}}{% link _posts/2017-02-22-lets-encrypt-cloudfoundry-bluemix.md %})
4. [Getting started with HTTPS: Trusting Let's Encrypt certificates in Java]({{site.url}}{% link _posts/2017-02-22-trusting-lets-encrypt-java.md %})

The **Java Runtime Environment (JRE)** ships with a tool called [keytool](http://docs.oracle.com/javase/6/docs/technotes/tools/solaris/keytool.html) to create certificates and manipulate key stores. For some of the most commonly used commands go check out this [site](https://www.sslshopper.com/article-most-common-java-keytool-keystore-commands.html). 

Every JRE has its own keystore, which contains all Certificate Authorities it trusts. This is also refered to as a trust store. This trust store is stored as file called `cacerts` under your `JAVA_JRE_HOME/lib/security/`. On my Mac this is located at `/Library/Java/JavaVirtualMachines/jdk1.8.0_60.jdk/Contents/Home/jre/lib/security`

The default password for this keystore is `changeit`. By using the following command, you can have a look at the JDK’s keystore:

```shell
 keytool -list -keystore cacerts -storepass changeit
```

If you want to use your own custom trust store, you'll need to override the default behaviour. This can be done in several different methods:

1. Start your Java application with a System property or Environment variable `javax.net.ssl.trustStore=<LOCATION TO YOUR CUSTOM TRUSTSTORE>` that refers to your custom keystore
2. Create a trust store named `jssecacerts`, with `keytool`, in your `JAVA_JRE_HOME/lib/security` folder. If this file exists, it takes precedence over `cacerts`
3.  if none of the previous methods are used, the default behaviour is to take the `cacerts`.

## Generate Self-Signed (untrusted) Certificate

When creating production grade web applications, usually they will be served over HTTPS. This requires a certificate to be present on your web server (Apache, Tomcat, Undertow, ...) to establish a secured connection.

Digital certificates can be of these two categories. Trusted Certificates are signed by a CA (Certificate Authority) while Untrusted certificates are self-signed. If you want to host your application over HTTPS locally, or perhaps in an intranet application, the most common solution is to generate a self-signed certificate. By default, a warning will be shown in the browser since it cannot validate the certificate.

![Untrusted certificate]({{ site.url }}/img/post/ssl-warning.png "Untrusted certificate")

To generate a self-signed certificate, you can use `keytool`:

1. Open a command prompt or terminal.
2. Run this command:
    ```bash 
    keytool -genkey \ 
            -keyalg RSA \
            -alias mycertificate \
            -keystore mykeystore.p12 \
            -validity 365 \ 
            -keysize 2048
    ```
3. Enter a password for the keystore.
4. When prompted for `first name` and `last name`, enter the `domain name` of the server. For example, `myserver` or `myserver.mycompany.com`.
5. Enter the other details, such as `Organizational Unit`, `Organization`, `City`, `State`, and `Country.
6. Confirm that the information entered is correct.
When prompted with Enter key password for `mycertificate`, press Enter to use the same password as the keystore password.
7. Run this command to verify the contents of the keystore:
    ```bash 
    keytool -list -v -keystore mykeystore.p12
    ```

> **NOTE**: Java 8 generates [PKCS12 keystores by default](http://openjdk.java.net/jeps/229). Java 7 and earlier defaulted to JKS keystores.
> You now have a PKCS12 keystore containing your self-signed certificate, which can be used by your webserver to open an HTTPS connection.

## Import Signed/Root/Intermediate Certificate

Use this method if you want to import a signed certificate, e.g. a certificate signed by a CA, into your keystore; it must match the private key that exists in the specified alias. You may also use this same command to import root or intermediate certificates that your CA may require to complete a chain of trust. Simply specify a unique alias, such as root instead of domain, and the certificate that you want to import.

This command imports the certificate (`domain.crt`) into the keystore (`keystore.jks`), under the specified alias (`domain`). If you are importing a signed certificate, it must correspond to the private key in the specified alias:

```shell
keytool -importcert \
        -trustcacerts -file domain.crt \
        -alias domain \
        -keystore keystore.jks
```

You will be prompted for the keystore password, then for a confirmation of the import action.

> **NOTE**: You may also use the command to import a CA's certificates into your Java truststore, which is typically located in $JAVA_JRE_HOME/lib/security/ cacerts assuming $JAVA_JRE_HOME is where your Java Runtime Environment is installed.

## Continue your HTTPS journey

1. [Getting started with HTTPS: Overview]({{site.url}}{% link _posts/2017-02-22-getting-started-https.md %})
2. **[Getting started with HTTPS: Java keystore and keytool essentials]()**
3. [Getting started with HTTPS: Let's Encrypt on Cloud Foundry and Bluemix]({{site.url}}{% link _posts/2017-02-22-lets-encrypt-cloudfoundry-bluemix.md %})
4. [Getting started with HTTPS: Trusting Let's Encrypt certificates in Java]({{site.url}}{% link _posts/2017-02-22-trusting-lets-encrypt-java.md %})
