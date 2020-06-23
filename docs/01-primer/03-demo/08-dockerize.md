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

Spring Boot provides an efficient approach to dockerize an application that takes full advantage of the docker layers.

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

Each layer is represented with a _layer id_.  For example, the layer id for step 1 is `d873c8fea8bd`.

```bash
Sending build context to Docker daemon  24.22MB
Step 1/4 : FROM adoptopenjdk/openjdk14:jre-14.0.1_7-alpine
jre-14.0.1_7-alpine: Pulling from adoptopenjdk/openjdk14
df20fa9351a1: Pull complete
519cb7a36d12: Pull complete
8e7ea6d57839: Pull complete
Digest: sha256:adb1a9135f33d5a82f2be01e8da3bf7b6300b884162f1036a419f885baafeb25
Status: Downloaded newer image for adoptopenjdk/openjdk14:jre-14.0.1_7-alpine
 ---> d873c8fea8bd
Step 2/4 : WORKDIR /opt/app
 ---> Running in 17301002974d
Removing intermediate container 17301002974d
 ---> 24d382537622
Step 3/4 : COPY ./build/libs/*.jar ./application.jar
 ---> 6331681c6d0e
Step 4/4 : CMD ["java", "-jar", "application.jar"]
 ---> Running in 31eaea7bfd34
Removing intermediate container 31eaea7bfd34
 ---> bc85de35701a
Successfully built bc85de35701a
Successfully tagged contact-us:local
```

These layers are saved and can be viewed using the `history` command, shown next

```bash
$ docker history contact-us:local
```

The history command will list all layers for the given image.

```bash

IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
bc85de35701a        20 seconds ago      /bin/sh -c #(nop)  CMD ["java" "-jar" "appli…   0B
6331681c6d0e        20 seconds ago      /bin/sh -c #(nop) COPY file:9a8d1169bd3e092b…   23.3MB
24d382537622        20 seconds ago      /bin/sh -c #(nop) WORKDIR /opt/app              0B
d873c8fea8bd        2 weeks ago         /bin/sh -c #(nop)  ENV JAVA_HOME=/opt/java/o…   0B
<missing>           2 weeks ago         /bin/sh -c set -eux;     apk add --no-cache …   168MB
<missing>           2 weeks ago         /bin/sh -c #(nop)  ENV JAVA_VERSION=jdk-14.0…   0B
<missing>           2 weeks ago         /bin/sh -c apk add --no-cache --virtual .bui…   14MB
<missing>           2 weeks ago         /bin/sh -c #(nop)  ENV LANG=en_US.UTF-8 LANG…   0B
<missing>           3 weeks ago         /bin/sh -c #(nop)  CMD ["/bin/sh"]              0B
<missing>           3 weeks ago         /bin/sh -c #(nop) ADD file:c92c248239f8c7b9b…   5.57MB
```

Note that the layers (also referred to as _intermediate images_) id shown in the history match those shown in during the build process, in reverse order.  The last layer build is shown at the top of the history (`bc85de35701a`).  The layers are reused as much as possible.  Building the image without changing anything will reuse the exiting layers, a shown next.

```bash
$ docker build . -t contact-us:local
Sending build context to Docker daemon  24.22MB
Step 1/4 : FROM adoptopenjdk/openjdk14:jre-14.0.1_7-alpine
 ---> d873c8fea8bd
Step 2/4 : WORKDIR /opt/app
 ---> Using cache
 ---> 24d382537622
Step 3/4 : COPY ./build/libs/*.jar ./application.jar
 ---> Using cache
 ---> 6331681c6d0e
Step 4/4 : CMD ["java", "-jar", "application.jar"]
 ---> Using cache
 ---> bc85de35701a
Successfully built bc85de35701a
Successfully tagged contact-us:local
```

If we rebuild our application (even without changing the code) and then build the docker image, the first two layers are reused and the third and the fourth layers are recomputed as captured by the following image.

![Docker Repetitive Builds-Layers]({{ '/assets/images/Docker-Repetitive-Builds-Layers.png' | absolute_url }})

We can also see this in practice.

1. Rebuild the FatJAR

   {% include custom/note.html details="We simply rebuild the FatJAR without changing the code." %}

   ```bash
   $ ./gradlew clean bootJar
   ```

1. Build the docker image

   ```bash
   $ docker build . -t contact-us:local
   Sending build context to Docker daemon  24.22MB
   Step 1/4 : FROM adoptopenjdk/openjdk14:jre-14.0.1_7-alpine
    ---> d873c8fea8bd
   Step 2/4 : WORKDIR /opt/app
    ---> Using cache
    ---> 24d382537622
   Step 3/4 : COPY ./build/libs/*.jar ./application.jar
    ---> 89cc0cbe9955
   Step 4/4 : CMD ["java", "-jar", "application.jar"]
    ---> Running in 086b0ec3c794
   Removing intermediate container 086b0ec3c794
    ---> e153943ea402
   Successfully built e153943ea402
   Successfully tagged contact-us:local
   ```

   The layer ids for the first two layers are the same as before, `d873c8fea8bd` and `24d382537622` respectively, but the following layers are new.  We can see this in the history command too.

   ```bash
   docker history contact-us:local
   IMAGE               CREATED              CREATED BY                                      SIZE                COMMENT
   e153943ea402        About a minute ago   /bin/sh -c #(nop)  CMD ["java" "-jar" "appli…   0B
   89cc0cbe9955        About a minute ago   /bin/sh -c #(nop) COPY file:0491a5cc99519740…   23.3MB
   24d382537622        9 minutes ago        /bin/sh -c #(nop) WORKDIR /opt/app              0B
   d873c8fea8bd        2 weeks ago          /bin/sh -c #(nop)  ENV JAVA_HOME=/opt/java/o…   0B
   <missing>           2 weeks ago          /bin/sh -c set -eux;     apk add --no-cache …   168MB
   <missing>           2 weeks ago          /bin/sh -c #(nop)  ENV JAVA_VERSION=jdk-14.0…   0B
   <missing>           2 weeks ago          /bin/sh -c apk add --no-cache --virtual .bui…   14MB
   <missing>           2 weeks ago          /bin/sh -c #(nop)  ENV LANG=en_US.UTF-8 LANG…   0B
   <missing>           3 weeks ago          /bin/sh -c #(nop)  CMD ["/bin/sh"]              0B
   <missing>           3 weeks ago          /bin/sh -c #(nop) ADD file:c92c248239f8c7b9b…   5.57MB
   ```

   Only the last two layers where updated.  The first two layers were reused.

The `application.jar` FatJAR, copied in step 3, contains our code together with all the dependencies we used (_our code + dependencies_).  When we change the application code, like when we add new features, we do not necessary change the dependencies.  One small change in the application's code will cause a new, relatively big, layer to be created.

We can also analyse the docker image created using [Dive](https://github.com/wagoodman/dive). Dive is a very good command line tool that helps you analyse a given docker image.

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

## What's the difference between `CMD`, `RUN` and `ENTRYPOINT`?

These three docker commands provide different functionality.

1. [`RUN`](https://docs.docker.com/engine/reference/builder/#run): executes one or more commands in a new layer and creates a new image.  This is ideal for installing any missing software packages.
1. [`CMD`](https://docs.docker.com/engine/reference/builder/#cmd): used to sets the default command to be executed when the docker container starts.  This can be overwritten from command line when docker container runs.
1. [`ENTRYPOINT`](https://docs.docker.com/engine/reference/builder/#entrypoint): created an executable docker container.

The RUN docker command is unique and serves a unique purpose.

The CMD and ENTRYPOINT can be used to run a command once the docker container starts.
Unless you need to interact with the docker image by providing different parameters, prefer the `ENTRYPOINT` over the `CMD` approach.

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

   **The order in which these are copied is very important**.  The layers that tend to modify more frequently, such as our application, should come after the layers that do not tend to update.

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

   This makes efficient use of docker's layering as new changes in the application code will reuse the previous layers and add another 20KB.  20 deployments per day will consume 1.2MB per week instead of 2GB.

1. (_Optional_) Run the docker image

   ```bash
   $ docker run -it -p 8080:8080 contact-us:local
   ```

1. (_Optional_) Access the application

   ```bash
   $ curl "http://localhost:8080/offices"
   ```

   You should get the contact details of the Cologne office

   ```json
   [{"office":"ThoughtWorks Cologne","address":"Lichtstr. 43i, 50825 Cologne, Germany","phone":"+49 221 64 30 70 63","email":"contact-de@thoughtworks.com"}]
   ```

## Dockersize application using Gradle `bootBuildImage` task (_less recommended approach_)

Spring Boot provides [the Gradle `bootBuildImage` task](https://docs.spring.io/spring-boot/docs/current/gradle-plugin/reference/html/#build-image) and we can create our docker image using this task.  This task makes use of [Buildpacks](https://buildpacks.io/) to create layered image.

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
