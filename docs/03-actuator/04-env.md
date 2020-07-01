---
layout: default
title: Env
parent: Spring Boot Actuator
nav_order: 4
permalink: docs/actuator/env/
---

# Environment
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Environment information

When debugging a problem with an application running in production, know its environment will help you determining how the application is configured and whether it is properly configured.

Spring boot provides an endpoint, `/env` that returns all environment information that the application has access too.

```bash
$ curl "http://localhost:8080/env" | jq .
```

Hitting this endpoint will return a `404` as shown next.

```json
{
  "timestamp": "2077-04-27T12:34:56.789+00:00",
  "status": 404,
  "error": "Not Found",
  "message": "",
  "path": "/env"
}
```

By default, Spring boot exposes only the health and the info endpoints.  The env endpoint is not exposed.  We can include the env endpoint to the list of endpoints exposed by the actuator by editing the `src/main/resources/application.yaml` properties file, as shown next.

```yaml
management:
  endpoints:
    web:
      base-path:
      exposure:
        include: env, health, info
```

The following properties fragment shows both `management` and `info` complete configuration.

```yaml
management:
  endpoints:
    web:
      base-path:
      exposure:
        include: env, health, info
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

In the above example we expose three, from the many, actuator endpoints.

```bash
$ curl "http://localhost:8080/env" | jq .
```

With the env actuator endpoint enables, we will receive a long JSON object containing the environment our application has access too.

{% include custom/note.html details="The following response was truncated for brevity." %}

```json
{
  "activeProfiles": [],
  "propertySources": [ /* ... */ ]
}
```

The response will contain sensitive information such as database password.  This information is masked for security reasons, using a [`Sanitizer`](https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/actuate/endpoint/Sanitizer.html), as shown next.

```json
"DATABASE_PASSWORD": {
  "value": "******",
  "origin": "System Environment Property \"DATABASE_PASSWORD\""
}
```

The response also provides the origin of the configuration as shown in the following fragment.

```yaml
{
  "name": "applicationConfig: [classpath:/application.yaml]",
  "properties": {
    "spring.datasource.url": {
      "value": "jdbc:postgresql://localhost:5432/contact-us",
      "origin": "class path resource [application.yaml]:3:10"
    },
    "spring.datasource.driver-class-name": {
      "value": "org.postgresql.Driver",
      "origin": "class path resource [application.yaml]:4:24"
    },
    "spring.datasource.username": {
      "value": "tw-data",
      "origin": "class path resource [application.yaml]:5:15"
    }
  }
}
```

This is quite important as it helps use identify the property file that needs to be updated.  The properties shown above are retrieved from the `application.yaml` (located at `src/main/resources/application.yaml`)

```yaml
spring:
  datasource:
    url: ${DATABASE_URL}
    driver-class-name: ${DATABASE_DRIVER}
    username: ${DATABASE_USERNAME}
    password: ${DATABASE_PASSWORD}
```
