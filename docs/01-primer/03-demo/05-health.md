---
layout: default
title: Health
parent: Demo
grand_parent: Primer
nav_order: 5
permalink: docs/primer/demo/health/
---

# Health
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Health endpoint (actuator)

1. Create a test

   Update the file `src/test/java/demo/boot/ContactUsApplicationTests.java` from

   ```java
   package demo.boot;

   import org.junit.jupiter.api.Test;
   import org.springframework.boot.test.context.SpringBootTest;

   @SpringBootTest
   class ContactUsApplicationTests {

     @Test
     void contextLoads() {
     }
   }
   ```

   to

   ```java
   package demo.boot;

   import org.junit.jupiter.api.DisplayName;
   import org.junit.jupiter.api.Test;
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.boot.test.context.SpringBootTest;
   import org.springframework.boot.test.web.client.TestRestTemplate;
   import org.springframework.http.HttpStatus;

   import static org.assertj.core.api.Assertions.assertThat;
   import static org.springframework.boot.test.context.SpringBootTest.WebEnvironment;

   @DisplayName( "Contact Us application" )
   @SpringBootTest( webEnvironment = WebEnvironment.RANDOM_PORT )
   public class ContactUsApplicationTests {

     @Autowired
     private TestRestTemplate restTemplate;

     @Test
     @DisplayName( "should return 200 when the health endpoint is accessed" )
     public void shouldReturn200HealthEndpoint() {
       assertThat( restTemplate.getForEntity( "/health", String.class ) )
         .matches( r -> r.getStatusCode() == HttpStatus.OK );
     }
   }
   ```

   Run the test.

   ```bash
   $ ./gradlew clean test
   ```

   The test will fail as we have not yet added the health endpoint.

   ```bash
   ...
   Contact Us application > should return 200 when the health endpoint is accessed FAILED
       java.lang.AssertionError at ContactUsApplicationTests.java:24
   ...

   1 test completed, 1 failed

   > Task :test FAILED
   ```

1. Add the [actuator dependency](https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-features.html)

   ```groovy
   dependencies {
     implementation 'org.springframework.boot:spring-boot-starter-actuator'
   }
   ```

1. Change the health endpoint path mapping

   By default, [the health endpoint is exposed at `/actuator/health`](https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-features.html#production-ready-endpoints).  We can change the path through the application properties.

   Update the file `src/main/resources/application.yaml`

   ```yaml
   management:
     endpoints:
       web:
         base-path:
   ```

   All we need to do is to change the `base-path` to blank.  We can set the health path too, as shown next, but this is not required as the health endpoint is already set to `/health`.

   ```yaml
   management:
     endpoints:
       web:
         base-path:
         path-mapping:
           health: /health
   ```

   Following is the equivalent in properties format.

   ```properties
   management.endpoints.web.base-path=
   ```

   Or, the following if both paths need to be set.

   ```properties
   management.endpoints.web.base-path=
   management.endpoints.web.path-mapping.health=/health
   ```

1. Run the test again

   ```bash
   $ ./gradlew clean test
   ```

   The test should pass now.

    ```bash
    ...
    Contact Us application > should return 200 when the health endpoint is accessed PASSED
    ...

    BUILD SUCCESSFUL in 6s
    5 actionable tasks: 5 executed
    ```

1. Start the application

   ```bash
   $ ./gradlew bootRun
   ```

1. Access the health endpoint: [http://localhost:8080/health](http://localhost:8080/health)

   ```bash
   $ curl -v http://localhost:8080/health
   ```

   This should return

   ```bash
   ➜  ~ curl -v http://localhost:8080/health
   *   Trying ::1...
   * TCP_NODELAY set
   * Connected to localhost (::1) port 8080 (#0)
   > GET /health HTTP/1.1
   > Host: localhost:8080
   > User-Agent: curl/7.64.1
   > Accept: */*
   >
   < HTTP/1.1 200
   < Content-Type: application/vnd.spring-boot.actuator.v3+json
   < Transfer-Encoding: chunked
   < Date: Teu, 27 April 2077 12:34:56 GMT
   <
   * Connection #0 to host localhost left intact
   * Closing connection 0
   {"status":"UP"}
   ```

1. Stop the application

   Use `[control] + [c]` to stop the application

## View application endpoint mapping (IntelliJ)

1. Run the application from IntelliJ

   ![Run Application from IntelliJ]({{ '/assets/images/IntelliJ-Run-Application.png' | absolute_url }})

1. Explore the Application Endpoints

   ![Explore the Application Endpoints]({{ '/assets/images/IntelliJ-Application-Endpoints.png' | absolute_url }})

   IntelliJ may report a warning such as the following.

   ```bash
   Failed to check application ready state JMX agent not loaded...
   ```

   Add the following to the _VM options_

   ```bash
   -Dcom.sun.management.jmxremote.port=9999
   -Dcom.sun.management.jmxremote.authenticate=false
   -Dcom.sun.management.jmxremote.ssl=false
   ```

   ![IntelliJ-Fix-JMX-Agent-Not-Loaded.png]({{ '/assets/images/IntelliJ-Fix-JMX-Agent-Not-Loaded.png' | absolute_url }})

   Please refer to the following [post](https://youtrack.jetbrains.com/issue/IDEA-204797) for more information about this warning.

1. Stop the application

   Use `[control] + [c]` to stop the application

## Tasks status

The health endpoint is implemented and tested

- [X] Health endpoint
- [ ] OpenAPI
- [ ] Return one office contact details
- [ ] Dockerize application
- [ ] Return all offices contact details
