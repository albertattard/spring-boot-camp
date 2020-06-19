---
layout: default
title: Dockerize
parent: Demo
grand_parent: Primer
nav_order: 8
permalink: docs/primer/demo/dockerize/
---

# Dockerize
{: .no_toc }

Spring Boot provides a [Gradle task `bootBuildImage`](https://docs.spring.io/spring-boot/docs/current/gradle-plugin/reference/html/#build-image), that we can use to build docker images and take full advantage of the docker layers.  To appreciate the benefits of this approach, we need to dive deeper in the docker layers.

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## What is a docker layer?

Our application can be dockerized using the following `dockerfile`.

```dockerfile
FROM adoptopenjdk/openjdk14:jre-14.0.1_7-alpine
WORKDIR /opt/app
ADD ./build/libs/*.jar ./application.jar
CMD ["java", "-jar", "application.jar"]
```

The above `dockerfile` has four docker commands (lines), each of which is translated as one layer (also referred to as _intermediate image_).  Each docker command creates a new layer by building on the previous one as shown in the following image.

![Docker Layers]({{ '/assets/images/Docker-Layers.png' | absolute_url }})

Let's build the above `dockerfile`.

Make sure to build the application first as the `dockerfile` will use the FatJAR file produced by the Gradle `build` task.

```bash
$ ./gradlew clean build
```

Build the docker image

```bash
$ docker build . -t contact-us:local
```

Each layer is represented with a _layer id_.  For example, the layer id for step 1 is `4b6ab0f52b1b`.

```bash
Sending build context to Docker daemon  24.16MB
Step 1/4 : FROM adoptopenjdk/openjdk14:jre-14.0.1_7-alpine
 ---> 4b6ab0f52b1b
Step 2/4 : WORKDIR /opt/app
 ---> Using cache
 ---> e4b34b3167d1
Step 3/4 : ADD ./build/libs/*.jar ./application.jar
 ---> 15457ae20ca5
Step 4/4 : CMD ["java", "-jar", "application.jar"]
 ---> Running in 5568e3296cfa
Removing intermediate container 5568e3296cfa
 ---> 36c3897b8921
Successfully built 36c3897b8921
Successfully tagged contact-us:local
```

If we modify our application and rebuild the docker image, the first two layers are reused and only the third and the fourth layers are recomputed as captured by the following image.

![Docker Repetitive Builds-Layers]({{ '/assets/images/Docker-Repetitive-Builds-Layers.png' | absolute_url }})

The `application.jar` FatJAR file, copied in step 3, contains our code together with all the dependencies we used.  When we change the code, like when we add new features, we do not necessary change the dependencies.  One small change in the code will cause a new, relatively big, layer to be created.  If, on the other hand, we split our FatJAR into several layers, that would help us reuse some of the previous layers as shown in the following images.

![Docker Repetitive Efficient Builds-Layers]({{ '/assets/images/Docker-Repetitive-Efficient-Builds-Layers.png' | absolute_url }})

In the above **fictitious** example, the first three layers are reused and the fourth and fifth layers, are recomputed.  This is a relatively slim layer and thus a small change in the code, produces new smaller layers.

This approach makes efficient use of the docker layering and caching system.  While we can achieve all this manually, Spring Boot provides a Gradle task for this, names `bootBuildImage`.  Spring Boot leverages [Buildpacks](https://buildpacks.io/) to create an efficient docker image that does not consume unnecessary space as shown above.

[Dive](https://github.com/wagoodman/dive) is a very good command line tool that helps you analyse a given docker image.

![Dive Demo](https://raw.githubusercontent.com/wagoodman/dive/master/.data/demo.gif)

Dive is a great tool for exploring a docker image, layer contents, and discovering ways to shrink the size of your Docker/OCI image.

## Dockersize application using Gradle `bootBuildImage` task (_recommended approach_)

Spring Boot provides the Gradle `bootBuildImage` task and we can create our docker image using this task.

{% include custom/note.html details="The <code>bootBuildImage</code> Gradle task does not make use of a <code>dockerfile</code>, thus one is not required" %}

1. Build the image using the Gradle `bootBuildImage`

   ```bash
   $ ./gradlew bootBuildImage --imageName=contact-us:local
   ```

   The first time we build the image may take up to two minutes, but subsequent runs are much faster.

   ```bash
   ...
       [creator]     *** Images (5f0fe880d725):
       [creator]           docker.io/library/contact-us:local

   Successfully built image 'docker.io/library/contact-us:local'


   BUILD SUCCESSFUL in 50s
   4 actionable tasks: 2 executed, 2 up-to-date
   ```

1. Run the image

   ```bash
   $ docker run -p 8080:8080 -it contact-us:local
   ```

   The docker container should start

   ```bash
      _____            _             _     _    _
     / ____|          | |           | |   | |  | |
    | |     ___  _ __ | |_ __ _  ___| |_  | |  | |___
    | |    / _ \| '_ \| __/ _` |/ __| __| | |  | / __|
    | |___| (_) | | | | || (_| | (__| |_  | |__| \__ \
     \_____\___/|_| |_|\__\__,_|\___|\__|  \____/|___/

   2077-04-27 12:34:54.895  INFO 1 --- [           main] demo.boot.ContactUsApplication           : Starting ContactUsApplication on 6773d5f400b7 with PID 1 (/opt/app/application.jar started by root in /opt/app)
   2077-04-27 12:34:54.903  INFO 1 --- [           main] demo.boot.ContactUsApplication           : No active profile set, falling back to default profiles: default
   2077-04-27 12:34:56.653  INFO 1 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 8080 (http)
   2077-04-27 12:34:56.690  INFO 1 --- [           main] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
   2077-04-27 12:34:56.690  INFO 1 --- [           main] org.apache.catalina.core.StandardEngine  : Starting Servlet engine: [Apache Tomcat/9.0.35]
   2077-04-27 12:34:56.861  INFO 1 --- [           main] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
   2077-04-27 12:34:56.861  INFO 1 --- [           main] o.s.web.context.ContextLoader            : Root WebApplicationContext: initialization completed in 1854 ms
   2077-04-27 12:34:57.247  INFO 1 --- [           main] o.s.s.concurrent.ThreadPoolTaskExecutor  : Initializing ExecutorService 'applicationTaskExecutor'
   2077-04-27 12:34:58.198  INFO 1 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
   2077-04-27 12:34:58.219  INFO 1 --- [           main] demo.boot.ContactUsApplication           : Started ContactUsApplication in 4.238 seconds (JVM running for 5.127)
   ```

1. Access the application

   ```bash
   $ curl http://localhost:8080/offices
   ```

   You should get the contact details of the Cologne office

   ```json
   [{"office":"ThoughtWorks Cologne","address":"Lichtstr. 43i, 50825 Cologne, Germany","phone":"+49 221 64 30 70 63","email":"contact-de@thoughtworks.com"}]
   ```

That's it!!

## Dockersize application using `dockerfile` (_not recommended approach_)

{% include custom/note.html details="The approach described in the <a href='#dockersize-application-using-gradle-bootbuildimage-task-recommended-approach'>previous part</a> makes better use of the docker layer and caching system, as described <a href='#what-is-a-docker-layer'>described before</a>" %}

1. Our application requires Java 14.  We can use the [OpenJDK 14 docker image](https://hub.docker.com/r/adoptopenjdk/openjdk14).

1. Create file `Dockerfile`

   ```dockerfile
   FROM adoptopenjdk/openjdk14:jdk-14.0.1_7-alpine-slim AS builder
   WORKDIR /opt/app
   COPY ./build.gradle .
   COPY ./gradle ./gradle
   COPY ./gradlew .
   COPY ./settings.gradle .
   COPY ./src ./src
   RUN ./gradlew build

   FROM adoptopenjdk/openjdk14:jre-14.0.1_7-alpine
   WORKDIR /opt/app
   COPY --from=builder /opt/app/build/libs/contact-us.jar ./application.jar
   CMD ["java", "-jar", "application.jar"]
   ```

   The above is an example of a multi-stage docker file to build the application before creating the second docker image that will run the application.  **Do not use a multi-stage docker file to build the application if a pipeline (such as [Jenkins](https://www.jenkins.io/) or [GOCD](https://www.gocd.org/)) is used to build the project**.  The pipeline will orchestrate the build process with better visibility and can use the artefacts produced by the previous stage to create the docker image.

1. Build the docker image

   ```bash
   $ docker build . -t contact-us:local
   ```

   This will take a minute or two to build as it initialises Gradle every time it runs.

   ```bash
   ...
   Removing intermediate container c8fb55e46747
    ---> 4082031517ad
   Successfully built 4082031517ad
   Successfully tagged contact-us:local
   ```

1. Run the docker image

   ```bash
   $ docker run -p 8080:8080 -it contact-us:local
   ```

   The docker container should start

   ```bash
      _____            _             _     _    _
     / ____|          | |           | |   | |  | |
    | |     ___  _ __ | |_ __ _  ___| |_  | |  | |___
    | |    / _ \| '_ \| __/ _` |/ __| __| | |  | / __|
    | |___| (_) | | | | || (_| | (__| |_  | |__| \__ \
     \_____\___/|_| |_|\__\__,_|\___|\__|  \____/|___/

   2077-04-27 12:34:54.895  INFO 1 --- [           main] demo.boot.ContactUsApplication           : Starting ContactUsApplication on 6773d5f400b7 with PID 1 (/opt/app/application.jar started by root in /opt/app)
   2077-04-27 12:34:54.903  INFO 1 --- [           main] demo.boot.ContactUsApplication           : No active profile set, falling back to default profiles: default
   2077-04-27 12:34:56.653  INFO 1 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 8080 (http)
   2077-04-27 12:34:56.690  INFO 1 --- [           main] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
   2077-04-27 12:34:56.690  INFO 1 --- [           main] org.apache.catalina.core.StandardEngine  : Starting Servlet engine: [Apache Tomcat/9.0.35]
   2077-04-27 12:34:56.861  INFO 1 --- [           main] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
   2077-04-27 12:34:56.861  INFO 1 --- [           main] o.s.web.context.ContextLoader            : Root WebApplicationContext: initialization completed in 1854 ms
   2077-04-27 12:34:57.247  INFO 1 --- [           main] o.s.s.concurrent.ThreadPoolTaskExecutor  : Initializing ExecutorService 'applicationTaskExecutor'
   2077-04-27 12:34:58.198  INFO 1 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
   2077-04-27 12:34:58.219  INFO 1 --- [           main] demo.boot.ContactUsApplication           : Started ContactUsApplication in 4.238 seconds (JVM running for 5.127)
   ```

1. Access the application

   ```bash
   $ curl http://localhost:8080/offices
   ```

   You should get the contact details of the Cologne office

   ```json
   [{"office":"ThoughtWorks Cologne","address":"Lichtstr. 43i, 50825 Cologne, Germany","phone":"+49 221 64 30 70 63","email":"contact-de@thoughtworks.com"}]
   ```

## Tasks status

The application can now be deployed as a docker image

- [X] Health endpoint
- [X] OpenAPI
- [X] Return one office contact details
- [X] Dockerize application
- [ ] Return all offices contact details
