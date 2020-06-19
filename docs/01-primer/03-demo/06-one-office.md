---
layout: default
title: Return one office
parent: Demo
grand_parent: Primer
nav_order: 6
permalink: docs/primer/demo/one-office/
---

# Return one office
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Rest Controller

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
   │   └── wrapper
   │       ├── gradle-wrapper.jar
   │       └── gradle-wrapper.properties
   ├── gradlew
   ├── gradlew.bat
   ├── settings.gradle
   └── src
      ├── main
      │   ├── java
      │   │   └── demo
      │   │       └── boot
      │   │           └── ContactUsApplication.java
      │   └── resources
      │       ├── application.yaml
      │       └── banner.txt
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
      "office": "ThoughtWorks Cologne",
      "address": "Lichtstr. 43i, 50825 Cologne, Germany",
      "phone": "+49 221 64 30 70 63",
      "email": "contact-de@thoughtworks.com"
     }
   ]
   ```

   The response does not need to be formatted as shown above.

   Update the file `src/test/java/demo/boot/ContactUsApplicationTests.java` and add the new test.

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
         new Office( "ThoughtWorks Cologne",
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

   Create file: `src/main/java/demo/boot/OfficeController.java`

   There are two options.

   1. Use the [`@RestController`](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/web/bind/annotation/RestController.html), which was introduced in [Spring 4.0](https://docs.spring.io/spring/docs/4.1.5.RELEASE/spring-framework-reference/html/new-in-4.0.html#_general_web_improvements) (*preferred option*)

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
              "ThoughtWorks Cologne",
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
              "ThoughtWorks Cologne",
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
   [{"office":"ThoughtWorks Cologne","address":"Lichtstr. 43i, 50825 Cologne, Germany","phone":"+49 221 64 30 70 63","email":"contact-de@thoughtworks.com"}]
   ```

   Access the endpoint from a browser: [http://localhost:8080/offices](http://localhost:8080/offices)

   ![One Office Result]({{ '/assets/images/One-Office-Result.png' | absolute_url }})

## Tasks status

The office endpoint is implemented and tested with one contact

- [X] Health endpoint
- [ ] OpenAPI
- [X] Return one office contact details
- [ ] Dockerize application
- [ ] Return all offices contact details
