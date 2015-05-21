---
layout: post
title: Getting started with Spring Security, Spring Session and Redis
author: Driss Amri
date: 2015-05-21
tags:
 - Java
 - Spring
 - Spring Session
 - Spring Security
 - Spring Boot
 - Redis
---
You want to build a scalable cloud ready application? Great! Let's have a look how to build a server  Spring Boot REST endpoint and secure it with stateless Spring Security. We want to authenticate every request with basic authentication OR with a token when the user has sent its credentials once. When a user authenticates with basic authentication we store a session ID in Redis with Spring Session and return it to the user to in the following requests.

_If you haven't heard of Spring Boot before, be sure to read up on it on: [What are Microservices](http://127.0.0.1:4000/blog/architecture/what-are-microservices/) or [Build your own Linkshortener API](http://127.0.0.1:4000/blog/rest/building-your-own-linkshortener-api/)._


## Standard HTTP Session in Java (JEE)
There are a quite a few shortcommings in Application Servers HTTP session implementations that we want to avoid:

 * Have you ever tried to enable [HTTP Sessions replication and clustering](http://tomcat.apache.org/tomcat-8.0-doc/cluster-howto.html) on Tomcat? It's not an easy thing to do.
 * Sessions are tightly coupled with the HTTP protocol, but there are other scenario's where you'd like to access the session like when using JMS. Or keeping sessions alive when using WebSockets.
 * How do you go about when you want to allow users to have multiple sessions, one for each tab (like in Gmail)?

## Get started with Spring Session
With [Spring Session](http://docs.spring.io/spring-session/docs/current/reference/html5/) all these issues are solved without any effort. This makes it easy to scale your cloud application since it doesn't need to store sessions on disk anymore. As long as your Application Server uses a  [HttpSession](http://docs.spring.io/spring-session/docs/current/reference/html5/#httpsession), you can get started in a couple of easy steps. Let's get started with a simple Spring Boot application with Spring Security for securing a RESTful service and store sessions in Redis:

_Application.java_
{% highlight java %}
@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
{% endhighlight %}

_pom.xml_
{% highlight xml %}
<dependencies>
  <dependency>
    <groupId>org.springframework.session</groupId>
    <artifactId>spring-session-data-redis</artifactId>
    <version>1.0.1.RELEASE</version>
  </dependency>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
  </dependency>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
  </dependency>
</dependencies>
{% endhighlight %}

This would give us a working web application, that is secured by default because we include the Spring Boot Security Starter dependency. For ease of use we'll start an in-memory database with one user present to use for authentication. Every will authenticate every request with Basic Authentication and also create a `HeaderHttpSessionStrategy` to tell Spring to use a `X-Auth-Token` header for retrieving the session.


_SecurityConfiguration.java_
{% highlight java %}
@Configuration
@EnableWebSecurity
@Order(SecurityProperties.ACCESS_OVERRIDE_ORDER)
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(AuthenticationManagerBuilder builder) throws Exception {
        builder.inMemoryAuthentication()
            .withUser("user").password("password").roles("USER");
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            .authorizeRequests()
            .anyRequest().authenticated()
            .and()
            .requestCache()
            .requestCache(new NullRequestCache())
            .and()
            .httpBasic();
    }

    @Bean
    public HttpSessionStrategy httpSessionStrategy() {
        return new HeaderHttpSessionStrategy();
    }
}
{% endhighlight %}  

## Hello Redis - a NoSQL key-value database
We don't want to to reauthenticate every request by going to the database, this is why we will store a Session ID in Redis. This Session ID will be passed to the client after authenticating the first time, and from then on it can send this as a request header for all subsequent requests which will verify the user is authenticated.

If you look at the Spring Session [documentation](http://docs.spring.io/spring-session/docs/current/reference/html5/#httpsession-redis) you'll see that they always use an `@EmbeddedRedisConfiguration` to startup an embedded Redis instance, it's pretty unclear that this is **NOT** yet available in Spring Session. They include some addtional classes in all their examples to get this to work, we'll have to wait for [Spring Session GitHub ticket](https://github.com/spring-projects/spring-session/issues/121) until this actually will be implemented in Spring itself. For now I have a simple Configuration class that starts an instance without any customization, by using the Embedded redis depdency.

_pom.xml_
{% highlight xml %}
<dependency>
    <groupId>com.github.kstyrc</groupId>
    <artifactId>embedded-redis</artifactId>
    <version>0.6</version>
  </dependency>
{% endhighlight %}
_EmbeddedRedisConfiguration.java_
{% highlight java %}
@Configuration
@EnableRedisHttpSession
public class EmbeddedRedisConfiguration {
    private static RedisServer redisServer;

    @Bean
    public JedisConnectionFactory connectionFactory() throws IOException {
        redisServer = new RedisServer(Protocol.DEFAULT_PORT);
        redisServer.start();
        return new JedisConnectionFactory();
    }

    @PreDestroy
    public void destroy() {
        redisServer.stop();
    }
}
{% endhighlight %}

And to finish it off we'll expose one endpoint to actually verify the outcome of our security configuration.

_UserController.java_
{% highlight java %}
@RestController
public class UserController {

    @RequestMapping("/api/users")
    public String authorized() {
        return "Hello Secured World";
    }
}
{% endhighlight %}

We can verify the behaviour of our application by using curl:  
`curl -v http://localhost:8080/api/users`

{% highlight bash %}
< HTTP/1.1 401 Unauthorized
* Server Apache-Coyote/1.1 is not blacklisted
< Server: Apache-Coyote/1.1
< X-Content-Type-Options: nosniff
< X-XSS-Protection: 1; mode=block
< Cache-Control: no-cache, no-store, max-age=0, must-revalidate
< Pragma: no-cache
< Expires: 0
< X-Frame-Options: DENY
< WWW-Authenticate: Basic realm="Realm"
< Content-Type: application/json;charset=UTF-8
< Transfer-Encoding: chunked
< Date: Thu, 21 May 2015 21:44:26 GMT
{% endhighlight %}

Now let's do that again, but this time added Basic Authentication with our in-memory database user:  
`curl -v http://localhost:8080/api/users -u user:password`

{% highlight bash %}
< HTTP/1.1 200 OK
* Server Apache-Coyote/1.1 is not blacklisted
< Server: Apache-Coyote/1.1
< X-Content-Type-Options: nosniff
< X-XSS-Protection: 1; mode=block
< Cache-Control: no-cache, no-store, max-age=0, must-revalidate
< Pragma: no-cache
< Expires: 0
< X-Frame-Options: DENY
< x-auth-token: ef555ceb-1c77-4fdd-8e42-e04399fe5b95
< Content-Type: text/plain;charset=UTF-8
< Content-Length: 19
< Date: Thu, 21 May 2015 21:47:30 GMT
{% endhighlight %}

We sent our credentials and received a `x-auth-token` in the response, which we can use in the following requests to authenticate:  
`curl -v http://localhost:8080/api/users -H "x-auth-token: ef555ceb-1c77-4fdd-8e42-e04399fe5b95"`

{% highlight bash %}
< HTTP/1.1 200 OK
* Server Apache-Coyote/1.1 is not blacklisted
< Server: Apache-Coyote/1.1
< X-Content-Type-Options: nosniff
< X-XSS-Protection: 1; mode=block
< Cache-Control: no-cache, no-store, max-age=0, must-revalidate
< Pragma: no-cache
< Expires: 0
< X-Frame-Options: DENY
< Content-Type: text/plain;charset=UTF-8
< Content-Length: 19
< Date: Thu, 21 May 2015 21:50:24 GMT
{% endhighlight %}

That's all there is too it. If you have any questions or suggestions do let me know on any of the social links below.
