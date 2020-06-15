---
layout: default
title: Spring
parent: Primer
nav_order: 2
permalink: docs/primer/spring/
---

# Spring
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## What is spring?

[Spring](https://spring.io/) is the most popular Java framework to date.  Released in [October 2002]
(https://en.wikipedia.org/wiki/Spring_Framework), Spring gained popularity due to its simplicity.  Alternative technologies, such as [Java EE](https://en.wikipedia.org/wiki/Java_Platform,_Enterprise_Edition), were quite complicated and required lots of boilerplate code.  Furthermore, these alternative technologies were hard to test as they relied on the application server.  It was not easy to test the application without spinning up the application server itself.

The Spring [comprise several projects](https://spring.io/projects), such as [Spring Data](https://spring.io/projects/spring-data) or [Spring Security](https://spring.io/projects/spring-security) to name just two.  The [Spring Framework](https://spring.io/projects/spring-framework) is just one of the projects.

## How can we use spring?

Spring comprise several projects and each project can be used independently.  For example, we can use Spring Data independent from the other projects, including the Spring Framework.

Spring projects can be added to the project like any other dependency.  For example, the following example adds the [Spring Data JDBC](https://spring.io/projects/spring-data-jdbc) to the project's dependencies.

```groovy
dependencies {
  implementation 'org.springframework.data:spring-data-jdbc:2.0.1.RELEASE'
}
```

This will allow us to use [`JdbcTemplate`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/jdbc/core/JdbcTemplate.html) to interact with the database in a safe manner.

## What is the difference between spring and spring boot?

Say that we need to create an application that
1. Interacts with a database
1. Provides a set of secure endpoints

We can take advantage of the respective Spring projects, such as Spring Framework, Spring Data and Spring Security.  While these projects take away most of the boilerplate code, they still need to be configured.

[Spring Boot](https://spring.io/projects/spring-boot) is one of the newly added projects to the Spring family.  The goal of Spring Boot is to simplify the usage and configuration of other boot compatible libraries.  It is important to understand that Spring Boot does not replace Spring Framework, for example.

Spring Boot, using respective starter projects such as `spring-boot-starter-web`, configures the respective project.  Spring Boot is highly opinionated so that we can start working with little effort.

By importing the right set of *starter* projects, we will setup an application, as shown next.

```groovy
dependencies {
  /* Spring Boot*/
  implementation 'org.springframework.boot:spring-boot-starter-web'
  implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
  implementation 'org.springframework.boot:spring-boot-starter-security'

  runtimeOnly 'org.postgresql:postgresql'

  /* Testing */
  testImplementation('org.springframework.boot:spring-boot-starter-test') {
    exclude group: 'org.junit.vintage', module: 'junit-vintage-engine'
  }
  testImplementation 'org.springframework.security:spring-security-test'
}
```

The above adds the required projects to our application.  All we need to do is to provide the database connection properties, and Spring Boot takes care of the rest.

```yaml
spring:
  datasource:
    url: ${DATABASE_URL}
    driver-class-name: ${DATABASE_DRIVER}
    username: ${DATABASE_USERNAME}
    password: ${DATABASE_PASSWORD}
```

The properties can be retrieved from the environment variables too, as shown in the above example.

## How does spring boot works?
