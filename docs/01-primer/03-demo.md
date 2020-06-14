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
1. [OpenApi](https://www.openapis.org/) (_Swagger_)
1. [Lombok](https://projectlombok.org/) ([Lombok Plugin](https://github.com/mplushnikov/lombok-intellij-plugin))
1. [PIT](https://pitest.org/)
1. [Mockito](https://site.mockito.org/)

## Scenario

Pending!!

## Create project

1. Access [Spring initializr https://start.spring.io/](https://start.spring.io/)

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

    Click _EXPLORE_ to view the project

    ![Spring initializr Explore]({{ 'assets/images/Spring-Initializr-Explore.png' | absolute_url }})

    Click _DOWNLOAD_ to download the zip file

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

      /* Spring/OpenApi */
      implementation 'org.springdoc:springdoc-openapi-ui:1.3.9'
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

1. Run the application using the Spring boot Gradle task `bootRun`

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

1. Stop the application

    Use `[control] + [c]` to stop the application

## (Optional) Change the ASCII art (the `banner.txt` file)

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

    Our application should accept a `GET` request to `/offices` and respond with the following [JSON](https://www.json.org/json-en.html).

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

    import static org.assertj.core.api.Assertions.assertThat;
    import static org.springframework.boot.test.context.SpringBootTest.WebEnvironment;

    @DisplayName( "Contact Us application" )
    @SpringBootTest( webEnvironment = WebEnvironment.RANDOM_PORT )
    public class ContactUsApplicationTests {

      @Autowired
      private TestRestTemplate restTemplate;

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

    The above example makes use of [Lombok](https://projectlombok.org/) to reduce the boilerplate code.

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

    Both options will yield the same the results, with the second approach uses fewer annotations.

    Run the tests.

    ```bash
    $ ./gradlew clean build

    ...
    Contact Us application > should return the offices PASSED
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

    Access the endpoint: [http://localhost:8080/offices](http://localhost:8080/offices)

    ![One Office Result]({{ 'assets/images/One-Office-Result.png' | absolute_url }})

## OpenApi (swagger)

1. Verify that [OpenApi dependency](https://github.com/springdoc/springdoc-openapi) is part of the dependencies

    ```groovy
    dependencies {
      /* Spring/OpenApi */
      implementation 'org.springdoc:springdoc-openapi-ui:1.3.9'
    }
    ```

    The above dependency should have been in there already.

1. Start the application (_if not already started_)

    ```bash
    $ ./gradlew bootRun
    ```

1. Access the OpenApi from browser: [http://localhost:8080/swagger-ui/index.html?configUrl=/v3/api-docs/swagger-config](http://localhost:8080/swagger-ui/index.html?configUrl=/v3/api-docs/swagger-config)

    ![OpenApi Demo]({{ 'assets/gifs/OpenApi-Demo.gif' | absolute_url }})
