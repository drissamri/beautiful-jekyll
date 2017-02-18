---
layout: post
title: Generate a self-signed certificate using keytool
author: Driss Amri
date: 2017-02-18
published: false
tags:
 - Java
 - SSL
 - TLS
 - Security 
 - X509
 - Certificate
---

When you are creating production grade web applications, usually they will be served over HTTPS. This requires a certificate to be present on your web server (Apache, Tomcat, or any other web server).

Digital certificates can be of these two categories. Trusted Certificates are signed by a CA (Certificate Authority) while Untrusted certificates are self-signed. Trusted certificates need to be signed by a Certificate Authorithy to ensure the highest level of trust. If you want to host your application over HTTPS on your local machine, or in an intranet application, usually the easiest solution is to generate a self-signed certificate. By default, this will shown the user a warning since the browser cannot validate the certificate.

![Untrusted certificate]({{ site.url }}/img/post/ssl-warning.png "Untrusted certificate")

To generate a self-signed SSL certificate you can use the [keytool](http://docs.oracle.com/javase/6/docs/technotes/tools/solaris/keytool.html) command on Windows, Mac, or Linux. This is a tool that comes with the Java Runtime Environment (JRE).

1. Open a command prompt or terminal.
2. Run this command:
    ```bash 
    keytool -genkey \ 
            -keyalg RSA \
            -alias mycertificate \
            -keystore mykeystore.jks \
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
    keytool -list -v -keystore mykeystore.jks
    ```
You now have a PKCS12 keystore containg your self-signed certificate, which can be used by your webserver to open an HTTPS connection.

