---
layout: default
title: Info
parent: Spring Boot Actuator
nav_order: 3
permalink: docs/actuator/info/
---

# Info
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Application information

[Spring boot actuator]( https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-features.html#production-ready-application-info) allows us to provide information about our application through [`InfoContributor`](https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/actuate/info/InfoContributor.html).


Spring boot collects the information from all `InfoContributor` and returns one JSON object as a reply to the `/info` endpoint.

```bash
$ curl "http://localhost:8080/info" | jq .
```

We have no information available yet and thus we get an empty JSON object.

```json
{}
```

We can add information about our application by simply adding information to the `src/main/resources/application.yaml` properties file as shown next.


```yaml
info:
  app:
    version: 1.0.0
    name: Contact Us Demo
    camp: Spring Boot Camp
    link: https://albertattard.github.io/spring-boot-camp/
```

Anything under the `info` property is treated as application property.  The following fragment shows both the `info` and `management` properties.

```yaml
management:
  endpoints:
    web:
      base-path:
  endpoint:
    health:
      show-details: always

info:
  app:
    version: 1.0.0
    name: Contact Us Demo
    camp: Spring Boot Camp
    link: https://albertattard.github.io/spring-boot-camp/
```

{% include custom/note.html details="We did not modify the <code>management</code> property." %}

Hit the `/info` endpoint

```bash
$ curl "http://localhost:8080/info" | jq .
```

Hitting the `/info` endpoint will now return the application information.

```json
{
  "app": {
    "name": "Contact Us Demo",
    "version": "1.0.0",
    "camp": "Spring Boot Camp",
    "link": "https://albertattard.github.io/spring-boot-camp/"
  }
}
```

While it is not required, it is recommended to group the custom application properties under the `app` property.  This helps identifying what information belongs to what.

## Git information

One way that we can be certain that the latest and greatest version of our application running is to include the Git commit information and expose this to an endpoint.  Then we can we can have a third-party application that checks the Git repository, build pipeline and our Kubernetes pods and verify that the latest version was built and deployed.

We can easily achieve this using the `/info` endpoint.

The [`GitInfoContributor`](https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/actuate/info/GitInfoContributor.html) is one of the contributors provided by Spring boot.  This simply looks for the properties file `git.properties` on the classpath.

Following is an example of such file.

```properties
git.branch=main
git.build.host=somewhere.far.far.away
git.build.user.email=albert.attard@thoughtworks.com
git.build.user.name=Albert Attard
git.build.version=unspecified
git.closest.tag.commit.count=
git.closest.tag.name=
git.commit.id=ffffffffffffffffffffffffffffffffffffffff
git.commit.id.abbrev=ffffff
git.commit.id.describe=
git.commit.message.full=The commit message\nThis is the long description
git.commit.message.short=The commit message
git.commit.time=2077-04-27T12\:34\:56+0200
git.commit.user.email=albert.attard@thoughtworks.com
git.commit.user.name=Albert Attard
git.dirty=true
git.remote.origin.url=https\://github.com/albertattard/contact-us-demo.git
git.tags=
git.total.commit.count=1
```

This file can be managed by a Gradle plugin, such as [`com.gorylenko.gradle-git-properties`](https://plugins.gradle.org/plugin/com.gorylenko.gradle-git-properties).  This plugin will read the Git information and will produce the `git.properties` properties file automatically.

```groovy
plugins {
  id 'com.gorylenko.gradle-git-properties' version '2.2.2'
}
```

The `build` Gradle task depends (indirectly) on the new `generateGitProperties` task, which is responsible from creating the `git.properties` properties file, as shown next.

```bash
$ ./gradlew build taskTree
...
:build
+--- :assemble
|    +--- :bootJar
|    |    \--- :classes
|    |         +--- :compileJava
|    |         +--- :generateGitProperties
|    |         \--- :processResources
...
```

Every time we build our application the `git.properties` is generated.  The same happens when our application is built from a pipeline, such as [Jenkins](https://www.jenkins.io/) or [GoCD](https://www.gocd.org/).  The pipeline will checkout the latest version from the Git repository and build it before packaging it into a container.

Accessing the `/info` endpoint

```bash
$ curl "http://localhost:8080/info" | jq .
```

will now contain both our application information and the Git related information as shown next.

```json
{
  "app": {
    "name": "Contact Us Demo",
    "version": "1.0.0",
    "camp": "Spring Boot Camp",
    "link": "https://albertattard.github.io/spring-boot-camp/"
  },
  "git": {
    "branch": "main",
    "commit": {
      "id": "ffffff",
      "time": "2077-04-27T12:34:56Z"
    }
  }
}
```
