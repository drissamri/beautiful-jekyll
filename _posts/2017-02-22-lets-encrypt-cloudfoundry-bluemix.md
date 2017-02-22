---
layout: post
title: Let's Encrypt SSL on Cloud Foundry and Bluemix
author: Driss Amri
date: 2017-02-22
tags:
 - Java
 - SSL
 - Security 
 - Let's Encrypt
 - Cloud Foundry
 - Bluemix
---
1. [Getting started with HTTPS: Overview]({{site.url}}{% link _posts/2017-02-22-getting-started-https.md %})
2. [Getting started with HTTPS: Java keystore and keytool essentials]({{site.url}}{% link _posts/2017-02-22-java-keystore-keytool-essentials.md %})
3. **[Getting started with HTTPS: Let's Encrypt on Cloud Foundry and Bluemix]()**
4. [Getting started with HTTPS: Trusting Let's Encrypt certificates in Java]({{site.url}}{% link _posts/2017-02-22-trusting-lets-encrypt-java.md %})

Let's Encrypt has made it terrible easy and free to get trusted certificates to setup HTTPS connections to your application. I will show you how to create a certificate and add it to your Cloud Foundry and Bluemix applications. Cloud Foundry is an open source `Platform-as-a-Serivce` which is used by many cloud providers. Bluemix is one of these providers that uses Cloud Foundry as a foundation for their platform. This also means that the Cloud Foundry method described below will also work for Bluemix. 

Since every implementation of Cloud Foundry will have a different graphical interface, I will try to do as much as possible through the `Command Line Interface (CLI)`. The prerequisite for all methods is that you have a Cloud Foundry application running. 

### Prerequisites
There are a few prerequisites for all methods described below. 
- Recent version of the Cloud Foundry Command Line Interface installed
- A working running Cloud Foundry application
- Added your custom domain in Cloud Foundry and setup your DNS at your provider to forward to Cloud Foundry.
- Install the Let's Encrypt [certbot](https://certbot.eff.org/) - make sure you choose `Software`: `none of the above` since we are doing a manual install of the certificate.
![Certbot]({{ site.url }}/img/post/certbot-manual.png "Certbot")

## Manual Let's Encrypt certificates on Cloud Foundry

MORE TO COME

## Automated Let's Encrypt certificates on Cloud Foundry

There are a few extra prerequisites for this automated step.  
Install the [Cloud Foundry Command Line interface](http://docs.cloudfoundry.org/cf-cli/install-go-cli.html).  
Download the [cloudfoundry-letsencrypt](https://github.com/bsyk/cf-letsencrypt) script  
Make sure you have [Python](https://www.python.org/downloads/) installed on your computer.  
Make sure you have [pip](https://pip.pypa.io/en/stable/installing/) installed on your computer.  

When you have the necessary prerequisites on your compouter, go to the downloaded `cf-letsencrypt` folder. Rename the `domains.yml.example` file to `domains.yml`. Add your `email`, `domain(s)` and `host(s)` for which you want to generate and install certificates. You can set the `staging` property to `true`, when trying out the script, so you don't hit the Let's Encrypt rate limits. When set to true, certificates that are generated will be signed by a fake/test certificate and are not trusted by browsers. 

Here you can see my `domains.yml` for my new experimenting domain `drissamri.me`:

```javascript
{
  "email": "test@drissamri.me",
  "staging": false,
  "domains": [
    {
      "domain": "drissamri.me",
      "hosts": [
        ".",
        "www"
     ]
    }
  ]
}
```
You are now ready to run the script: `python setup-app.py`. This can take a few minutes. If no errors occured, the certificates are generated and available in the script's generated `letsencrypt` application on Cloud Foundry. You can run the following commands to copy the files over to your local machine, don't forget to fill in your domain inside the commands. In my case it would be `drissamri.me`

```shell
cf ssh letsencrypt -c 'cat ~/app/conf/live/<<YOURDOMAIN>>/cert.pem' > cert.pem
cf ssh letsencrypt -c 'cat ~/app/conf/live/<<YOURDOMAIN>>//chain.pem' > chain.pem
cf ssh letsencrypt -c 'cat ~/app/conf/live/<<YOURDOMAIN>>/fullchain.pem' > fullchain.pem
cf ssh letsencrypt -c 'cat ~/app/conf/live/<<YOURDOMAIN>>/privkey.pem' > privkey.pem
```

As far as I know, there is no `Command Line Interface` command to upload these certificates to your Cloud Foundry instance. I suggest you head over to your providers website, and upload them manually through the website. If you are using Bluemix, please use the method provided below. This script's generated `letsencrypt` application does not shutdown automatically, make sure you run `cf stop letsencrypt` to stop the application or `cf delete letsencrypt` to delete the application. Deleting means you can no longer download the certificates and will have to generate them again if you didn't copy them and need them for another webserver.

## Automated Let's Encrypt certificates on Bluemix

There are a few extra prerequisites for this automated step.  
Install the [Cloud Foundry Command Line interface](http://docs.cloudfoundry.org/cf-cli/install-go-cli.html).  
Install the [Bluemix Command Line interface](http://clis.ng.bluemix.net/ui/home.html).  
Download the [bluemix-letsencrypt](https://github.com/ibmjstart/bluemix-letsencrypt) script
Make sure you have [Python](https://www.python.org/downloads/) installed on your computer.  
Make sure you have [pip](https://pip.pypa.io/en/stable/installing/) installed on your computer.  

This method is very similar to the one for Cloud Foundry, only in this step you amlso need to have the `Bluemix CLI` installed and use the `bluemix-letsencrypt` specific script instead of the `cloudfoundry-letsencrypt` script. 

When you have the necessary prerequisites on your compouter, go to the downloaded `bluemix-letsencrypt` folder. Rename the `domains.yml.example` file to `domains.yml`. Add your `email`, `domain(s)` and `host(s)` for which you want to generate and install certificates. You can set the `staging` property to `true`, when trying out the script, so you don't hit the Let's Encrypt rate limits. When set to true, certificates that are generated will be signed by a fake/test certificate and are not trusted by browsers. 

Here you can see my `domains.yml` for my new experimenting domain `drissamri.me`:

```javascript
{
  "email": "test@drissamri.me",
  "staging": false,
  "domains": [
    {
      "domain": "drissamri.me",
      "hosts": [
        ".",
        "www"
     ]
    }
  ]
}
```

You are now ready to run the script: `python setup-app.py`. This can take a few minutes, but if everything went OK, the output should be something like this: 
```shell
requested state: started
instances: 1/1
usage: 64M x 1 instances
urls: drissamri.me/.well-known/acme-challenge/, www.drissamri.me/.well-known/acme-challenge/
last uploaded: Wed Feb 22 00:22:32 UTC 2017
stack: cflinuxfs2
buildpack: python_buildpack

     state     since                    cpu    memory     disk      details
#0   running   2017-02-22 01:24:37 AM   0.0%   0 of 64M   0 of 1G
Parsing log files.
Waiting for certs...
Certs not ready yet, retrying in 5 seconds.
Making GET request to https://drissamri.me
("bad handshake: Error([('SSL routines', 'SSL23_GET_SERVER_HELLO', 'tlsv1 unrecognized name')],)",)
Running: cf ssh letsencrypt -c 'cat ~/app/conf/live/drissamri.me/cert.pem'
Running: cf ssh letsencrypt -c 'cat ~/app/conf/live/drissamri.me/chain.pem'
Running: cf ssh letsencrypt -c 'cat ~/app/conf/live/drissamri.me/fullchain.pem'
Running: cf ssh letsencrypt -c 'cat ~/app/conf/live/drissamri.me/privkey.pem'
Stopping app letsencrypt in org yourorganization / space dev as name@email.com...
OK
Attempting certificate upload...
Uploading certificate to domain 'drissamri.me'...
OK
Certificate was uploaded to domain 'drissamri.me'.

Making GET request to https://drissamri.me
Warning: Please note that your SSL certificate, its corresponding PRIVATE KEY, and its intermediate certificates have been downloaded to the current working directory. If you need to remove them, use `rm *.pem`
Upload Succeeded
```

The benefit of using this method over the previous Cloud Foundry method is that your certificates are automatically uploaded to Bluemix for your existing domain. This means you don't have to manually download them from your instances and upload them. Your website, in my case `drissamri.me`, should now be available over https. If you want to copy the certificates, you just need to follow the instructions in the output. This is done using by ssh'ing into your application using `cf ssh letsencrypt` and copying the files located in `~/app/conf/live/YOURDOMAIN/`.

## Continue your HTTPS journey
1. [Getting started with HTTPS: Overview]({{site.url}}{% link _posts/2017-02-22-getting-started-https.md %})
2. [Getting started with HTTPS: Java keystore and keytool essentials]({{site.url}}{% link _posts/2017-02-22-java-keystore-keytool-essentials.md %})
3. **[Getting started with HTTPS: Let's Encrypt on Cloud Foundry and Bluemix]()**
4. [Getting started with HTTPS: Trusting Let's Encrypt certificates in Java]({{site.url}}{% link _posts/2017-02-22-trusting-lets-encrypt-java.md %})
