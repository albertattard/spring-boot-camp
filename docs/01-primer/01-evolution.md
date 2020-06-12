---
layout: default
title: Evolution
parent: Primer
nav_order: 1
permalink: docs/primer/evolution/
---

# Evolution
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Application servers

A web application receives HTTP requests and then respond to these requests.  Consider the following HTTP request to Google ([http://www.google.com](http://www.google.com)).

```bash
$ curl -s -o /dev/null -v http://www.google.com
```

Some webserver at Google is receiving this request and then reply to it as shown below.

```bash
*   Trying 2a00:1450:4001:801::2004...
* TCP_NODELAY set
* Connected to www.google.com (2a00:1450:4001:801::2004) port 80 (#0)
> GET / HTTP/1.1
> Host: www.google.com
> User-Agent: curl/7.64.1
> Accept: */*
>
< HTTP/1.1 200 OK
< Date: Fri, 12 Jun 2020 15:39:07 GMT
< Expires: -1
< Cache-Control: private, max-age=0
< Content-Type: text/html; charset=ISO-8859-1
< P3P: CP="This is not a P3P policy! See g.co/p3phelp for more info."
< Server: gws
< X-XSS-Protection: 0
< X-Frame-Options: SAMEORIGIN
< Set-Cookie: 1P_JAR=2020-06-12-15; expires=Sun, 12-Jul-2020 15:39:07 GMT; path=/; domain=.google.com; Secure
< Set-Cookie: NID=204=fSrl3ApUcVrTNsB09Jqpw3HOkDndjb-e7O6LsWgrlakSuuOwDn9_-n7-9fPsc_WMV0HtrBsYFTmeUVn7SiuGHldHEfwC7VJaJYtG0d9cHsvc93jAskuwfOfkEsOUIQoRZOZX5S1hI6Qyz2IwDicRMdrOj-n6ktp4HG1zBIwqXb0; expires=Sat, 12-Dec-2020 15:39:07 GMT; path=/; domain=.google.com; HttpOnly
< Accept-Ranges: none
< Vary: Accept-Encoding
< Transfer-Encoding: chunked
<
{ [8932 bytes data]
* Connection #0 to host www.google.com left intact
* Closing connection 0
```

The actual response was sent to `/dev/null` for brevity.  The following image captures the above.

![HTTP Request]({{ '/assets/images/Curl-Request-Google.png' | absolute_url }})

The actual HTTP request handling can be written from the ground up using the [Java Streams and Sockets](https://docs.oracle.com/javase/tutorial/networking/sockets/clientServer.html).  Alternatively, we can take advantage of existing infrastructure.

One approach, which was very dominant in the past, is to create the application and follow the [JSR 369: Java Servlet 4.0 Specification](https://jcp.org/en/jsr/detail?id=369)) (or the [latest](https://javaee.github.io/servlet-spec/)).  This approach would take care of most boilerplate code and allows you to focus on the application instead.  The application is packaged as a [WAR](https://en.wikipedia.org/wiki/WAR_(file_format)) or JAR file and then it is deployed to an application or web server as shown next.

![Deploy to Application Server]({{ '/assets/images/Deploy-to-Application-Server.png' | absolute_url }})

There are several application or web servers available.  The following table shows a list of some popular application and web servers.

| Name                                                                     | Type               | Vendor            |
| ------------------------------------------------------------------------ | ------------------ | ----------------- |
| [Apache Tomcat](https://tomcat.apache.org/)                              | Webserver          | Apache            |
| [Netty](https://netty.io/)                                               | Webserver          | The Netty project |
| [Jetty](https://www.eclipse.org/jetty/)                                  | Webserver          | Eclipse           |
| [GlassFish](https://projects.eclipse.org/proposals/eclipse-glassfish)    | Application Server | Eclipse           |
| [WebSphere](https://www.ibm.com/cloud/websphere-application-server)      | Application Server | IBM               |
| [WebLogic](https://www.oracle.com/middleware/technologies/weblogic.html) | Application Server | Oracle            |

Our application will be running together with other applications on the same application or web server.  Containers were not popular in the late 1990s, and this was the best approach available.

This approach has several challenges.  The application shares the same Java process with other applications.  If one application misbehaves, the other applications on the same Java process may suffer.  Another issue that the above approach has is that each application needs to work with the provided infrastructure.  Say that our application needs to move to a newer version of Java.  In the above approach all applications deployed to the same application or web server will have to be upgraded to run on the new version.

## Server as part of the application

A new approach, that took us by surprises, was introduce were the application or web server becomes part of the application as shown next.

![Application Server Part of Application]({{ '/assets/images/Application-Server-Part-of-Application.png' | absolute_url }})

In the new approach, the application or web server is part of our application and we can start the whole application using the `java -jar` command, as shown next.

```bash
$ java -jar application.jar
```

Our FatJAR contains all our code, the code of our dependencies and the application or web server.  Following are some options that we can use. 

| Name                                                  | Vendor                 |
| ----------------------------------------------------- | ---------------------- |
| [Spring Boot](https://spring.io/projects/spring-boot) | Spring                 |
| [Dropwizard](https://www.dropwizard.io/)              | JetBrains              |
| [Micronaut](https://micronaut.io/)                    | Object Computing, Inc. |
| [Quarkus](https://quarkus.io/)                        | Red Hat/IBM            |
| [Helidon](https://helidon.io/)                        | Oracle                 |

Since its introduction, Spring boot dominated the market.  According to [Google trends](https://trends.google.com/trends/explore?q=Spring%20Boot,Dropwizard,Micronaut,Quarkus,Helidon), the competitors do not come close.

![Spring Boot - Popularity]({{ '/assets/images/Spring-Boot-Popularity.png' | absolute_url }})

Popularity is quite important as this means, that there is lots of interest in the technology.

## Containers

This approach was improved further with the use of containers, such as [Docker](https://www.docker.com/), as shown next.

![Containers]({{ '/assets/images/Containers.png' | absolute_url }})

The application is now deployed as a container that includes

1. The OS that suites our application
1. The Java version that our needs
1. The application or web server
1. The third-party dependencies that our application needs
1. Our application

## Which approach to should I pick?

**It depends!!**

While we can never have a one-size-fits-all answer, containers + Fat JARs that contain the application server seems to be the way to go.  The adoption of Spring Boot is witness to this.

Spring is very productive as it takes away much boilerplate code that is otherwise required.  Being very popular, makes it very accessible too.  There are many videos and tutorial available and endless libraries that works well with it.

Take for example [Axon Framework](https://axoniq.io/), a [CQRS framework](https://martinfowler.com/bliki/CQRS.html).  This framework works effortlessly with the Spring echo system but takes some time to set it up without using Spring.  Like this, many other libraries or frameworks leverage Spring to simplify their adoption.

Spring boot take this to the next level as it simplifies the integration of third-party libraries with little or no effort at all.  You can create an application without adding one line of configuration.

There are other alternatives such as [Micronaut]( https://micronaut.io/) which provide even better performance without sacrificing performance.  Yet, these alternatives do not enjoy the support Spring does and you may find yourself alone.
