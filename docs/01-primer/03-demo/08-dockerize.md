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
Sending build context to Docker daemon  24.25MB
Step 1/4 : FROM adoptopenjdk/openjdk14:jre-14.0.1_7-alpine
 ---> 4b6ab0f52b1b
Step 2/4 : WORKDIR /opt/app
 ---> Using cache
 ---> 625114fe4e40
Step 3/4 : ADD ./build/libs/*.jar ./application.jar
 ---> c9f365565171
Step 4/4 : CMD ["java", "-jar", "application.jar"]
 ---> Running in 32a880759172
Removing intermediate container 32a880759172
 ---> 68914dbaa5f9
Successfully built 68914dbaa5f9
Successfully tagged contact-us:local
```

If we modify our application and rebuild the docker image, the first two layers are reused and only the third and the fourth layers are recomputed as captured by the following image.

![Docker Repetitive Builds-Layers]({{ '/assets/images/Docker-Repetitive-Builds-Layers.png' | absolute_url }})

The `application.jar` FatJAR, copied in step 3, contains our code together with all the dependencies we used (_our code + dependencies_).  When we change the application code, like when we add new features, we do not necessary change the dependencies.  One small change in the application's code will cause a new, relatively big, layer to be created.

We can analyse the docker image created using [Dive](https://github.com/wagoodman/dive). Dive is a very good command line tool that helps you analyse a given docker image.

```bash
$ dive contact-us:local
```

This will show the layers of our docker image.

![Dive-Contact-Us-FatJAR.png]({{ '/assets/images/Dive-Contact-Us-FatJAR.png' | absolute_url }})

Our very small application comprises 7 small files, roughly 20KB in size.  Every time we modify the code, we create a docker layer of more than 20MB.  This is quite inefficient from space point of view.  If we deploy 20 times per day, we will generate 2GB of waste in one week (_20MB × 20 deployments in one day × 5 day_).

Alternatively, we can split our FatJAR into several layers, so that parts that do not change that often are separated from those part that change frequently.  That would help us reuse some of the previous layers as shown in the following images.

![Docker Repetitive Efficient Builds-Layers]({{ '/assets/images/Docker-Repetitive-Efficient-Builds-Layers.png' | absolute_url }})

In the above **fictitious** example, the first three layers are reused and the fourth and fifth layers, are recomputed.  Note that once a layer is modified, all the subsequent layers need to be recomputed.  The fourth layer, where our application code is, is a relatively slim layer and thus a small change in the application code, produces a new small layer.

This approach makes efficient use of the docker layering and caching system.

##  Dockersize application using layered JAR (_recommended approach_)

We can take advantage of [layered JAR](https://docs.spring.io/spring-boot/docs/current/gradle-plugin/reference/html/#packaging-layered-jars) to create an efficient docker image that makes best us of space.

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

   {% include custom/note.html details="Note that the <code>bootJar</code> Gradle task does not depend on the tests tasks and thus may build a JAR that fails the tests" %}

   ```bash
   $ ./gradlew clean check

   ...
   BUILD SUCCESSFUL in 8s
   5 actionable tasks: 5 executed
   ```

1. Build the application

   {% include custom/note.html details="Make sure to build the JAR file after the layered JAR configuration described in the previous step" %}

   ```bash
   $ ./gradlew clean bootJar
   ```

   The following steps depend on the JAR file created by the `bootJar` task.

1. Create multistage `dockerfile` that takes advantage of the layered JAR

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

1. Analyse the layered JAR docker image

   ```bash
   $ dive contact-us:local
   ```

    The layer containing the application code is 12KB in size, which is relatively small when compared to the dependencies layer, which is 23MB.

   ![Dive - Contact Us Layered]({{ '/assets/images/Dive-Contact-Us-Layered-JAR.png' | absolute_url }})

   This makes efficient use of docker's layering as following changes will reuse the previous layers and add another 20KB.  20 deployments per day will consume 1.2MB per week instead of 2GB.

1. (_Optionally_) Run the docker image

   ```bash
   $ docker run -it -p 8080:8080 contact-us:local
   ```

1. (_Optionally_) Access the application

   ```bash
   $ curl "http://localhost:8080/offices"
   ```

   You should get the contact details of the Cologne office

   ```json
   [{"office":"ThoughtWorks Cologne","address":"Lichtstr. 43i, 50825 Cologne, Germany","phone":"+49 221 64 30 70 63","email":"contact-de@thoughtworks.com"}]
   ```

## Dockersize application using Gradle `bootBuildImage` task (_less recommended approach_)

Spring Boot provides the Gradle `bootBuildImage` task and we can create our docker image using this task.  This task makes use of [Buildpacks](https://buildpacks.io/) to create layered image.

{% include custom/note.html details="The <code>bootBuildImage</code> Gradle task does not make use of a <code>dockerfile</code>, thus one is not required" %}

1. Run the tests

   {% include custom/note.html details="Note that the <code>bootBuildImage</code> Gradle task does not depend on the tests tasks and thus may build a JAR that fails the tests" %}

   ```bash
   $ ./gradlew clean check

   ...
   BUILD SUCCESSFUL in 8s
   5 actionable tasks: 5 executed
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


   BUILD SUCCESSFUL in 1m 50s
   4 actionable tasks: 2 executed, 2 up-to-date
   ```

1. Analyse the created docker image

   ```bash
   $ dive contact-us:local
   ```

   The following image shows the layers of the image `contact-us:local`

   ![Dive - Contact Us using Buildpacks]({{ '/assets/images/Dive-Contact-Us-Buildpacks.png' | absolute_url }})

   Note that this approach is not as efficient as one would have hoped so.  The application layer still contains both our code and the application dependencies.

## Tasks status

The application can now be deployed as a docker image

- [X] Health endpoint
- [X] OpenAPI
- [X] Return one office contact details
- [X] Dockerize application
- [ ] Return all offices contact details
