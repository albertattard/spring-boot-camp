---
layout: default
title: Demo
parent: Primer
nav_order: 3
permalink: docs/primer/demo/
---

# Contact us
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Technology stack

1. [Java 14](https://openjdk.java.net/projects/jdk/14/)
1. [Gradle](https://gradle.org/) (_single project_)
1. [Docker](https://www.docker.com/)
1. [Spring](https://spring.io/)
   1. [Spring framework](https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html) (_dependency injection_)
   1. [Spring Boot](https://spring.io/projects/spring-boot)
   1. [Spring Web](https://docs.spring.io/spring/docs/current/spring-framework-reference/web.html) (not reactive) and [REST](https://en.wikipedia.org/wiki/Representational_state_transfer)
   1. [Spring Actuator](https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-features.html)
1. [OpenApi](https://www.openapis.org/) (_Swagger_)
1. [Lombok](https://projectlombok.org/) ([Lombok Plugin](https://github.com/mplushnikov/lombok-intellij-plugin))
1. [PIT](https://pitest.org/)
1. [Mockito](https://site.mockito.org/)

**The technology stack is shown first as this helps the reader understand what technologies are used in this demo.  Never pick the technology before understanding the problem being solved first!!**

## Scenario

Our company, ThoughWorks, is thinking in building an API to provide information about its offices and different ways a potential customer can reach out.  This API will not have any frontend UI but will simply provide a set of endpoints.

1. Expose a health endpoint

   This endpoint will be used by [Kubernetes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/) to verify that our application is up and running.  It takes the form of `GET` request to the `/health`, as shown in th following example.

   ```bash
   $ curl -v "http://localhost:8080/health"
   ```

   Our application needs to return [an HTTP `200` response](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/200), as shown next.

   ```bash
   ...
   < HTTP/1.1 200
   ...
   ```

   The content returned by the response does not matter much.

1. Return the list of offices

   The second endpoint that we need to build is a `GET` request to the e`/offices`, as shown in th following example.

   ```bash
   $ curl "http://localhost:8080/offices"
   ```

   This should return a [JSON](https://www.json.org/json-en.html) array of objects as shown next.

   ``` json
   [
     {
       "office": "string",
       "address": "string",
       "phone": "string",
       "email": "string"
     }
   ]
   ```

   Following is an example of an office contact information.

   ```json
   [
     {
       "office": "ThoughtWorks Köln",
       "address": "Lichtstr. 43i, 50825 Cologne, Germany",
       "phone": "+49 221 64 30 70 63",
       "email": "contact-de@thoughtworks.com"
     }
   ]
   ```

   The application needs to be deployed as a docker image.

### Tasks

We need to carry out the following tasks

- [ ] Health endpoint
- [ ] OpenAPI
- [ ] Return one office contact details
- [ ] Return all offices contact details
- [ ] Dockerize application

## Create project

1. Access [Spring initializr https://start.spring.io/](https://start.spring.io/)

1. Configure the application

   ![Spring initializr]({{ 'assets/images/Spring-Initializr.png' | absolute_url }})

   | Option      | Selection      |
   | ----------- | -------------- |
   | Project     | Gradle Project |
   | Language    | Java           |
   | Spring Boot | 2.3.1          |

   **Project Metadata**

   | Option       | Selection       |
   | ------------ | --------------- |
   | Group        | demo            |
   | Artifact     | contact-us      |
   | Name         | contact-us      |
   | Description  | Contact Us Demo |
   | Package name | demo.games      |
   | Packaging    | jar             |
   | Java         | 14              |

   **Dependencies**

   | Dependencies |
   | ------------ |
   | Lombok       |
   | Spring Web   |

1. (_Optional_) Explore the project

   Click _EXPLORE_ to view the project

   ![Spring initializr]({{ 'assets/images/Spring-Initializr.png' | absolute_url }})

1. Download (or generate) the project

   Click _DOWNLOAD_ (or _GENERATE_) to download the zip file

   ![Spring initializr Explore]({{ 'assets/images/Spring-Initializr-Explore.png' | absolute_url }})

The application can also downloaded from [contact-us.zip]({{ 'assets/startes/contact-us.zip' | absolute_url }})

## Configure the project

1. Extract the downloaded zip file

   ```bash
   $ unzip contact-us.zip

   Archive:  contact-us.zip
      creating: contact-us/
     inflating: contact-us/settings.gradle
      creating: contact-us/gradle/
      creating: contact-us/gradle/wrapper/
     inflating: contact-us/gradle/wrapper/gradle-wrapper.properties
     inflating: contact-us/gradle/wrapper/gradle-wrapper.jar
     inflating: contact-us/gradlew
     inflating: contact-us/gradlew.bat
     inflating: contact-us/build.gradle
      creating: contact-us/src/
      creating: contact-us/src/main/
      creating: contact-us/src/main/java/
      creating: contact-us/src/main/java/demo/
      creating: contact-us/src/main/java/demo/boot/
     inflating: contact-us/src/main/java/demo/boot/ContactUsApplication.java
      creating: contact-us/src/main/resources/
     inflating: contact-us/src/main/resources/application.properties
      creating: contact-us/src/main/resources/templates/
      creating: contact-us/src/main/resources/static/
      creating: contact-us/src/test/
      creating: contact-us/src/test/java/
      creating: contact-us/src/test/java/demo/
      creating: contact-us/src/test/java/demo/boot/
     inflating: contact-us/src/test/java/demo/boot/ContactUsApplicationTests.java
     inflating: contact-us/HELP.md
     inflating: contact-us/.gitignore
   ```

   The directory structure of the project

   ```bash
   $ tree contact-us
   contact-us
   ├── HELP.md
   ├── build.gradle
   ├── gradle
   │   └── wrapper
   │       ├── gradle-wrapper.jar
   │       └── gradle-wrapper.properties
   ├── gradlew
   ├── gradlew.bat
   ├── settings.gradle
   └── src
      ├── main
      │   ├── java
      │   │   └── demo
      │   │       └── boot
      │   │           └── ContactUsApplication.java
      │   └── resources
      │       ├── application.properties
      │       ├── static
      │       └── templates
      └── test
          └── java
              └── demo
                  └── boot
                      └── ContactUsApplicationTests.java

   14 directories, 10 files
   ```

1. Navigate in the project's directory

   ```bash
   $ cd contact-us
   ```

   All commands are executed from within the project directory.

1. Delete the unnecessary files and folders

   ```bash
   $ rm -rf src/main/resources/templates/
   $ rm -rf src/main/resources/static/
   $ rm HELP.md
   ```

1. (_Optional_) Change the empty file's `src/main/resources/application.properties` extension to `yaml` (or `yml`), `src/main/resources/application.yaml`

   ```bash
   $ mv src/main/resources/application.properties src/main/resources/application.yaml
   ```

   This is a matter of preference as the application can be configured either using `.properties` or `.yaml`.

   Note that the examples shown in this demo make use of `yaml`.

1. Open the project in the IDE

   ```bash
   $ idea .
   ```

1. Configure Gradle

   Update file: `build.gradle`

   ```groovy
   plugins {
     id 'java'

     id 'org.springframework.boot' version '2.3.0.RELEASE'
     id 'io.spring.dependency-management' version '1.0.9.RELEASE'
   }

   java {
     sourceCompatibility = JavaVersion.VERSION_14
     targetCompatibility = JavaVersion.VERSION_14
   }

   repositories {
     mavenCentral()
     jcenter()
   }

   configurations {
     compileOnly {
      extendsFrom annotationProcessor
     }
   }

   dependencies {
     /* Lombok */
     compileOnly 'org.projectlombok:lombok'
     annotationProcessor 'org.projectlombok:lombok'

     /* Spring */
     implementation 'org.springframework.boot:spring-boot-starter-web'
     testImplementation('org.springframework.boot:spring-boot-starter-test') {
      exclude group: 'org.junit.vintage', module: 'junit-vintage-engine'
     }
   }

   test {
     useJUnitPlatform()
     testLogging {
      events = ['FAILED', 'PASSED', 'SKIPPED', 'STANDARD_OUT']
     }
   }
   ```

1. Build the application

   ```bash
   $ ./gradlew clean build

   ...
   BUILD SUCCESSFUL in 4s
   6 actionable tasks: 5 executed, 1 up-to-date
   ```

   Should build without errors

## Run the application

1. Run the application using the [Spring boot Gradle task `bootRun`](https://docs.spring.io/spring-boot/docs/current/gradle-plugin/api/org/springframework/boot/gradle/tasks/run/BootRun.html)

   ```bash
   $ ./gradlew bootRun
   ```

   This should start the application

   ```bash
   > Task :bootRun

     .   ____          _            __ _ _
    /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
   ( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
    \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
     '  |____| .__|_| |_|_| |_\__, | / / / /
    =========|_|==============|___/=/_/_/_/
    :: Spring Boot ::        (v2.3.0.RELEASE)

   2077-04-27 12:34:55.768  INFO 5554 --- [           main] demo.games.GameApplication               : Starting GameApplication on Alberts-MBP.fritz.box with PID 5554 (build/classes/java/main started by albertattard in .)
   2077-04-27 12:34:55.771  INFO 5554 --- [           main] demo.games.GameApplication               : No active profile set, falling back to default profiles: default
   2077-04-27 12:34:56.545  INFO 5554 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 8080 (http)
   2077-04-27 12:34:56.556  INFO 5554 --- [           main] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
   2077-04-27 12:34:56.556  INFO 5554 --- [           main] org.apache.catalina.core.StandardEngine  : Starting Servlet engine: [Apache Tomcat/9.0.35]
   2077-04-27 12:34:56.629  INFO 5554 --- [           main] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
   2077-04-27 12:34:56.629  INFO 5554 --- [           main] o.s.web.context.ContextLoader            : Root WebApplicationContext: initialization completed in 819 ms
   2077-04-27 12:34:56.752  INFO 5554 --- [           main] o.s.s.concurrent.ThreadPoolTaskExecutor  : Initializing ExecutorService 'applicationTaskExecutor'
   2077-04-27 12:34:56.890  INFO 5554 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
   2077-04-27 12:34:56.898  INFO 5554 --- [           main] demo.games.GameApplication               : Started GameApplication in 1.508 seconds (JVM running for 1.838)
   <=========----> 75% EXECUTING [1m 23s]
   > :bootRun
   ```

1. Access the application: [http://localhost:8080/](http://localhost:8080/)

   ![Whitelabel Error Page]({{ 'assets/images/Whitelabel-Error-Page.png' | absolute_url }})

   The application does not expose any endpoints and we have no custom error handling.

1. Stop the application

   Use `[control] + [c]` to stop the application

## (_Optional_) Change the ASCII art (the `banner.txt` file)

1. Create a banner ([Ascii Art](http://patorjk.com/software/taag/#p=display&f=Big&t=Contact%20Us))

   Create file: `src/main/resources/banner.txt`

   ```bash
     _____            _             _     _    _
    / ____|          | |           | |   | |  | |
   | |     ___  _ __ | |_ __ _  ___| |_  | |  | |___
   | |    / _ \| '_ \| __/ _` |/ __| __| | |  | / __|
   | |___| (_) | | | | || (_| | (__| |_  | |__| \__ \
    \_____\___/|_| |_|\__\__,_|\___|\__|  \____/|___/
   ```

   Any text will do here.

1. Running the application will now show the new banner

   ```bash
   $  ./gradlew bootRun

   > Task :bootRun
      _____            _             _     _    _
     / ____|          | |           | |   | |  | |
    | |     ___  _ __ | |_ __ _  ___| |_  | |  | |___
    | |    / _ \| '_ \| __/ _` |/ __| __| | |  | / __|
    | |___| (_) | | | | || (_| | (__| |_  | |__| \__ \
     \_____\___/|_| |_|\__\__,_|\___|\__|  \____/|___/
    ...
   ```

   Use `[control] + [c]` to stop the application

## Health endpoint (actuator)

1. Create a test

   Update the file `src/test/java/demo/games/ContactUsApplicationTests.java` from

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
         path-mapping:
           health: /health
   ```

   Following is the equivalent in properties format.

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
   $  ./gradlew bootRun
   ```

1. Access the health endpoint: [http://localhost:8080/health](http://localhost:8080/health)

   ```bash
   $ curl http://localhost:8080/health
   ```

   This should return

   ```json
   {"status":"UP"}
   ```

1. Stop the application

   Use `[control] + [c]` to stop the application

### View application endpoint mapping (IntelliJ)

1. Run the application from IntelliJ

   ![Run Application from IntelliJ]({{ 'assets/images/Run-Application-IntelliJ.png' | absolute_url }})

1. Explore the Application Endpoints

   ![Explore the Application Endpoints]({{ 'assets/images/IntelliJ-Application-Endpoints.png' | absolute_url }})

1. Stop the application

### Tasks status

The health endpoint is implemented and tested

- [X] Health endpoint
- [ ] OpenAPI
- [ ] Return one office contact details
- [ ] Return all offices contact details
- [ ] Dockerize application

## Return one office address

1. Project structure

    Clean the project to remove any artefacts produced by the build task before.

   ```bash
   $ ./gradlew clean
   ```

   The project structure should look like the following (easier to compare without the *build* folder)

   ```bash
   $ tree .
   .
   ├── build.gradle
   ├── gradle
   │   └── wrapper
   │       ├── gradle-wrapper.jar
   │       └── gradle-wrapper.properties
   ├── gradlew
   ├── gradlew.bat
   ├── settings.gradle
   └── src
      ├── main
      │   ├── java
      │   │   └── demo
      │   │       └── boot
      │   │           └── ContactUsApplication.java
      │   └── resources
      │       ├── application.yaml
      │       └── banner.txt
      └── test
          └── java
              └── demo
                   └── boot
                       └── ContactUsApplicationTests.java

    12 directories, 10 files
   ```

1. Update the application tests

   Our application should accept a `GET` request to `/offices` and respond with the following JSON.

   ```json
   [
     {
      "office": "ThoughtWorks Köln",
      "address": "Lichtstr. 43i, 50825 Cologne, Germany",
      "phone": "+49 221 64 30 70 63",
      "email": "contact-de@thoughtworks.com"
     }
   ]
   ```

   The response does not need to be formatted as shown above.

   Update the file `src/test/java/demo/games/ContactUsApplicationTests.java` and add the new test.

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
     public void shouldReturn200HealthEndpoint() { /* ... */ }

     @Test
     @DisplayName( "should return the offices" )
     public void shouldReturnTheOffices() {
       final Office[] offices = {
         new Office( "ThoughtWorks Köln",
           "Lichtstr. 43i, 50825 Cologne, Germany",
           "+49 221 64 30 70 63",
           "contact-de@thoughtworks.com" )
       };

       assertThat( restTemplate.getForObject( "/offices", Office[].class ) )
         .isEqualTo( offices );
     }
   }
   ```

   The test will not compile yet as we are missing the `Office` class.

   Create file: `src/main/java/demo/boot/Office.java`

   ```java
   package demo.boot;

   import lombok.AllArgsConstructor;
   import lombok.Data;
   import lombok.NoArgsConstructor;

   @Data
   @NoArgsConstructor
   @AllArgsConstructor
   public class Office {

     private String office;
     private String address;
     private String phone;
     private String email;
   }
   ```

   The above example makes use of [Lombok](https://projectlombok.org/) to reduce the boilerplate code.  Make sure that the [Lombok IntelliJ plugin](https://plugins.jetbrains.com/plugin/6317-lombok) is also installed, as otherwise IntelliJ will not work as expected.

   Running the test, will fail as we have not yet implemented the endpoint.

   ```bash
   $ ./gradlew clean build

   ...
   Contact Us application > should return the offices FAILED
     org.springframework.web.client.RestClientException at ContactUsApplicationTests.java:29
      Caused by: org.springframework.http.converter.HttpMessageNotReadableException at ContactUsApplicationTests.java:29
        Caused by: com.fasterxml.jackson.databind.exc.MismatchedInputException at ContactUsApplicationTests.java:29
   ...
   ```

1. Create the office controller

   Create file: `src/main/java/demo/games/OfficeController.java`

   There are two options.

   1. Use the [`@RestController`](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/web/bind/annotation/RestController.html), which as introduced in [Spring 4.0](https://docs.spring.io/spring/docs/4.1.5.RELEASE/spring-framework-reference/html/new-in-4.0.html#_general_web_improvements) (*preferred option*)

      ```java
      package demo.boot;

      import org.springframework.web.bind.annotation.GetMapping;
      import org.springframework.web.bind.annotation.RestController;

      import java.util.List;

      @RestController
      public class OfficeController {

        @GetMapping( "/offices" )
        public List<Office> offices() {
          return List.of(
            new Office(
              "ThoughtWorks Köln",
              "Lichtstr. 43i, 50825 Cologne, Germany",
              "+49 221 64 30 70 63",
              "contact-de@thoughtworks.com"
            )
          );
        }
      }
      ```

   1. Use the [`@Controller`](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/stereotype/Controller.html) together with [`@ResponseBody`](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/web/bind/annotation/ResponseBody.html) annotations

      ```java
      package demo.boot;

      import org.springframework.stereotype.Controller;
      import org.springframework.web.bind.annotation.GetMapping;
      import org.springframework.web.bind.annotation.ResponseBody;

      import java.util.List;

      @Controller
      public class OfficeController {

        @ResponseBody
        @GetMapping( "/offices" )
        public List<Office> offices() {
          return List.of(
            new Office(
              "ThoughtWorks Köln",
              "Lichtstr. 43i, 50825 Cologne, Germany",
              "+49 221 64 30 70 63",
              "contact-de@thoughtworks.com"
            )
          );
        }
      }
      ```

   Both options will yield the same the results, with the first approach uses fewer annotations.

   Run the tests.

   ```bash
   $ ./gradlew clean build

   ...
   Contact Us application > should return the offices PASSED

   Contact Us application > should return 200 when the health endpoint is accessed PASSED
   ...

   BUILD SUCCESSFUL in 6s
   6 actionable tasks: 6 executed
   ```

   The test should now pass.

1. Access the new endpoint

   Start the application

   ```bash
   $ ./gradlew bootRun
   ```

   Access the new endpoint from the command line

   ```bash
   $ curl http://localhost:8080/offices
   ```

   This should return the office

   ```json
   [{"office":"ThoughtWorks Köln","address":"Lichtstr. 43i, 50825 Cologne, Germany","phone":"+49 221 64 30 70 63","email":"contact-de@thoughtworks.com"}]
   ```

   Access the endpoint from a browser: [http://localhost:8080/offices](http://localhost:8080/offices)

   ![One Office Result]({{ 'assets/images/One-Office-Result.png' | absolute_url }})

### Tasks status

The office endpoint is implemented and tested with one contact

- [X] Health endpoint
- [ ] OpenAPI
- [X] Return one office contact details
- [ ] Return all offices contact details
- [ ] Dockerize application

## OpenApi (swagger)

1. Add that [OpenApi dependency](https://github.com/springdoc/springdoc-openapi) dependency

   ```groovy
   dependencies {
    /* Spring/OpenApi */
    implementation 'org.springdoc:springdoc-openapi-ui:1.3.9'
   }
   ```

1. Start the application (_if not already started_)

   ```bash
   $ ./gradlew bootRun
   ```

1. Access the OpenApi from browser: [http://localhost:8080/swagger-ui/index.html?configUrl=/v3/api-docs/swagger-config](http://localhost:8080/swagger-ui/index.html?configUrl=/v3/api-docs/swagger-config)

   ![OpenApi Offices Endpoint]({{ 'assets/images/OpenApi-Offices-Endpoint.png' | absolute_url }})

   You can try the API too.

   ![OpenApi Demo]({{ 'assets/gifs/OpenApi-Demo.gif' | absolute_url }})

### Tasks status

The application endpoints are exposed through OpenAPI

- [X] Health endpoint
- [X] OpenAPI
- [X] Return one office contact details
- [ ] Return all offices contact details
- [ ] Dockerize application

## Dockerize the application

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

   The above is an example of a multi-stage docker file that builds the application before creating the second docker image that will run the application.  **Do not use a multi-stage docker file if a pipeline (such as [Jenkins](https://www.jenkins.io/) or [GOCD](https://www.gocd.org/)) is used to build the project**.  The pipeline will orchestrate the build process with better visibility and can use the artefacts produced by the previous stage to create the docker image.

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

   You should get the contact details of the Köln office

   ```json
   [{"office":"ThoughtWorks Köln","address":"Lichtstr. 43i, 50825 Cologne, Germany","phone":"+49 221 64 30 70 63","email":"contact-de@thoughtworks.com"}]
   ```

## Add more TW offices details

Pending!!
