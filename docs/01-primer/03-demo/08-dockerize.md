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

Spring Boot provides a [Gradle task `bootBuildImage`](https://docs.spring.io/spring-boot/docs/current/gradle-plugin/reference/html/#build-image), that we can use to build docker images and take full advantage of the docker layers.

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
COPY ./build/libs/*.jar ./application.jar
CMD ["java", "-jar", "application.jar"]
```

The above `dockerfile` has four docker commands (lines), each of which is translated as one layer (also referred to as _intermediate image_).  Each docker command creates a new layer by building on the previous one as shown in the following image.

![Docker Layers]({{ '/assets/images/Docker-Layers.png' | absolute_url }})

Let's build the above `dockerfile`.

Make sure to build the application first as the `dockerfile` will use the FatJAR file produced by the Gradle `bootJar` task.

```bash
$ ./gradlew clean bootJar
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
Step 3/4 : COPY ./build/libs/*.jar ./application.jar
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

The `application.jar` FatJAR, copied in step 3, contains our code together with all the dependencies we used (_our code + dependencies_).  When we change the application code, like when we add new features, we do not necessary change the dependencies.  One small change in the application's code will cause a new, relatively big, layer to be created.

Alternatively, we can split our FatJAR into several layers, so that parts that do not change that often are separated from those part that change frequently.  That would help us reuse some of the previous layers as shown in the following images.

![Docker Repetitive Efficient Builds-Layers]({{ '/assets/images/Docker-Repetitive-Efficient-Builds-Layers.png' | absolute_url }})

In the above **fictitious** example, the first three layers are reused and the fourth and fifth layers, are recomputed.  Note that once a layer is modified, all the subsequent layers need to be recomputed.  The fourth later, where our application code is, is a relatively slim layer and thus a small change in the application code, produces a new small layer.

This approach makes efficient use of the docker layering and caching system.  While we can achieve all this manually, Spring Boot provides a Gradle task for this, names `bootBuildImage`.  Spring Boot leverages [Buildpacks](https://buildpacks.io/) to create an efficient docker image that does not consume unnecessary space as shown above.

[Dive](https://github.com/wagoodman/dive) is a very good command line tool that helps you analyse a given docker image.

![Dive Demo](https://raw.githubusercontent.com/wagoodman/dive/master/.data/demo.gif)

Dive is a great tool for exploring a docker image, layer contents, and discovering ways to shrink the size of your Docker/OCI image.

## Dockersize application using Gradle `bootBuildImage` task (_recommended approach_)

Spring Boot provides the Gradle `bootBuildImage` task and we can create our docker image using this task.

{% include custom/note.html details="The <code>bootBuildImage</code> Gradle task does not make use of a <code>dockerfile</code>, thus one is not required" %}

1. Run all tests

   The `bootBuildImage` does not run the tests, as this only depends on `bootJar`, which in turn depends on the `classes` task as shown next.

   ```bash
   $ ./gradlew bootBuildImage taskTree

   ...
   :bootBuildImage
   \--- :bootJar
        \--- :classes
             +--- :compileJava
             \--- :processResources
   ```

   Instead, we can use the `check` task as the `check` task depends on the tests.  Run the tests.

   ```bash
   $ ./gradlew clean check

   ...
   BUILD SUCCESSFUL in 14s
   7 actionable tasks: 7 executed
   ```

1. Build the image using the Gradle `bootBuildImage` task

   ```bash
   $ ./gradlew bootBuildImage --imageName=contact-us:local
   ```

   The first time we build the image may take a couple of minutes, but subsequent runs are much faster.

   ```bash
   ...
       [creator]     *** Images (5f0fe880d725):
       [creator]           docker.io/library/contact-us:local

   Successfully built image 'docker.io/library/contact-us:local'


   BUILD SUCCESSFUL in 50s
   4 actionable tasks: 2 executed, 2 up-to-date
   ```

1. (_Optionally_) Run the image

   ```bash
   $ docker run -it --network host contact-us:local
   ```

   {% include custom/note.html details="Note that this time we are not publishing ports as we are using the host network (<code>--network host</code>)" %}

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

1. (_Optionally_) Access the application

   ```bash
   $ curl "http://localhost:8080/offices"
   ```

   You should get the contact details of the Cologne office

   ```json
   [{"office":"ThoughtWorks Cologne","address":"Lichtstr. 43i, 50825 Cologne, Germany","phone":"+49 221 64 30 70 63","email":"contact-de@thoughtworks.com"}]
   ```

1. (_Optionally_) Analyse the created docker image

   ```bash
   $ dive contact-us:local
   ```

   The following image shows the layers of the image `contact-us:local`

   ![Dive - Contact Us using Buildpacks]({{ '/assets/images/Dive-Contact-Us-Buildpacks.png' | absolute_url }})

   Our application's code is added to the end.  This enables reuse of the upper layers when our application changes.

   The layer containing the application code is 20KB in size, which is relatively small.  This makes efficient use of docker's layering as following changes will reuse the previous layers and add another 20KB.

##  Dockersize application using layered JAR (_alternative approach_)

We can take advantage of [layered JAR](https://docs.spring.io/spring-boot/docs/current/gradle-plugin/reference/html/#packaging-layered-jars) to create an efficient docker image that makes best us of space.  This achieves similar results as those shown in the [previous part](#dockersize-application-using-gradle-bootbuildimage-task-recommended-approach), using layered JAR approach instead of Buildpacks.

1. Create a layered JAR

   Update file: `build.gradle`

   ```groovy
   bootJar {
     layered()
   }
   ```

   The following four layers are defined, by default

   1. `dependencies`: containing the dependency, whose version **does not** contain `SNAPSHOT`
   1. `spring-boot-loader`: containing the jar loader classes
   1. `snapshot-dependencies`: containing the snapshot dependency, whose version contains `SNAPSHOT`.
   1. `application`: containing the application classes and resources

   It is important that the layers that are less likely to change are first and those that are most likely to change are last as this will make best use of caching.

   When creating a layered JAR, the [`spring-boot-jarmode-layertools`](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-jarmode-layertools) JAR file is added as a dependency.  The _layertools_ JAR enables the application to run in a special mode, which allows the bootstrap code to run something entirely different from the application.

   ```bash
   $ java -Djarmode=layertools -jar build/libs/contact-us.jar
   ```

   The _layertools_ has three options as shown next

   ```bash
   Usage:
     java -Djarmode=layertools -jar contact-us.jar

   Available commands:
     list     List layers from the jar that can be extracted
     extract  Extracts layers from the jar for image creation
     help     Help about any command
   ```

   To list the layers that our layered JAR has, use the `list` option, as shown next.

   ```bash
   $ java -Djarmode=layertools -jar build/libs/contact-us.jar list
   dependencies
   spring-boot-loader
   snapshot-dependencies
   application
   ```

   The layered JAR will have these four layers.  We can extract the layered JAR using the `extract` option.


1. Run the tests

   ```bash
   $ ./gradlew clean check

   ...
   BUILD SUCCESSFUL in 14s
   7 actionable tasks: 7 executed
   ```

   {% include custom/note.html details="Note that the <code>bootJar</code> Gradle task does not depend on the tests tasks and thus may build a JAR that fails the tests" %}

1. Build the application

   {% include custom/note.html details="Make sure to build the JAR file after the layered JAR configuration described in the previous step" %}

   ```bash
   $ ./gradlew clean bootJar
   ```

   The following steps depend on the JAR file created by the `bootJar` task.

1. Create multistage `dockerfile`

   Create file: `dockerfile`

   ```dockerfile
   FROM adoptopenjdk/openjdk14:jre-14.0.1_7-alpine as builder
   WORKDIR /opt/app
   COPY ./build/libs/*.jar application.jar
   RUN java -Djarmode=layertools -jar application.jar extract

   FROM adoptopenjdk/openjdk14:jre-14.0.1_7-alpine
   WORKDIR /opt/app
   COPY --from=builder /opt/app/dependencies ./
   COPY --from=builder /opt/app/spring-boot-loader ./
   COPY --from=builder /opt/app/snapshot-dependencies ./
   COPY --from=builder /opt/app/application ./
   ENTRYPOINT ["java", "org.springframework.boot.loader.JarLauncher"]
   ```

   The multistage `dockerfile` shown above is using the _layertools_ to extract the layered JAR produced in the previous step, into its four layers and then copying each layer.

   ```dockerfile
   COPY --from=builder /opt/app/dependencies ./
   COPY --from=builder /opt/app/spring-boot-loader ./
   COPY --from=builder /opt/app/snapshot-dependencies ./
   COPY --from=builder /opt/app/application ./
   ```

   Spring Boot can be started using the [`JarLauncher`](https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/loader/JarLauncher.html), instead of the traditional `java -jar application.jar`

   ```dockerfile
   ENTRYPOINT ["java", "org.springframework.boot.loader.JarLauncher"]
   ```

1. Build the docker image

   ```bash
   $ docker build . -t contact-us:local
   ```

1. (_Optionally_) Run the docker image

   ```bash
   $ docker run -it --network host contact-us:local
   ```

   {% include custom/note.html details="Note that this time we are not publishing ports as we are using the host network (<code>--network host</code>)" %}

1. (_Optionally_) Access the application

   ```bash
   $ curl "http://localhost:8080/offices"
   ```

   You should get the contact details of the Cologne office

   ```json
   [{"office":"ThoughtWorks Cologne","address":"Lichtstr. 43i, 50825 Cologne, Germany","phone":"+49 221 64 30 70 63","email":"contact-de@thoughtworks.com"}]
   ```

1. (_Optionally_) Analyse the layered JAR docker image

   ```bash
   $ dive contact-us:local
   ```

   This docker image has less layers when compared with the one generated by the Gradle `bootBuildImage` task.

   ![Dive - Contact Us Layered]({{ '/assets/images/Dive-Contact-Us-Layered-JAR.png' | absolute_url }})

    The layer containing the application code is 20KB in size, which is relatively small.  This makes efficient use of docker's layering as following changes will reuse the previous layers and add another 20KB.

## Dockersize application using FatJAR (_less recommended approach_)

{% include custom/note.html details="The approaches described before, using <a href='#dockersize-application-using-gradle-bootbuildimage-task-recommended-approach'>Buildpacks</a> and using <a href='#dockersize-application-using-layered-jar-alternative-approach'>layered JAR</a>, make better use of the docker layering and caching system, as described <a href='#what-is-a-docker-layer'>described before</a> and should be preferred over this approach." %}

The following example is only shown for completeness and you should only use this approach if the previous two approaches are not possible.

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

1. (_Optionally_) Run the image

   ```bash
   $ docker run -it --network host contact-us:local
   ```

   {% include custom/note.html details="Note that this time we are not publishing ports as we are using the host network (<code>--network host</code>)" %}

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

1. (_Optionally_) Access the application

   ```bash
   $ curl "http://localhost:8080/offices"
   ```

   You should get the contact details of the Cologne office

   ```json
   [{"office":"ThoughtWorks Cologne","address":"Lichtstr. 43i, 50825 Cologne, Germany","phone":"+49 221 64 30 70 63","email":"contact-de@thoughtworks.com"}]
   ```

1. (_Optionally_) Analyse the FatJAR docker image

   ```bash
   $ dive contact-us:local
   ```

   The application's layer is 44MB in size, which is quite large when compared with the 20KB when using the other approaches.

   ![Dive - Contact Us FatJAR]({{ '/assets/images/Dive-Contact-Us-FatJAR.png' | absolute_url }})

## Tasks status

The application can now be deployed as a docker image

- [X] Health endpoint
- [X] OpenAPI
- [X] Return one office contact details
- [X] Dockerize application
- [ ] Return all offices contact details
