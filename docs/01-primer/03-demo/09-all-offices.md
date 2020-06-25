---
layout: default
title: Return all office
parent: Demo
grand_parent: Primer
nav_order: 9
permalink: docs/primer/demo/all-offices/
---

# Return all office
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Service layer

1. Introduce a service layer

   ![Controller-Service-CSV-Flow.png]({{ '/assets/images/Controller-Service-CSV-Flow.png' | absolute_url }})

   The service layer will us to test the controller without having to worry about the actual data.  Also, we can then replace the CSV data file with a database without having to change the controller or its tests.

1. Save the offices details in a csv file.

   Download the file [offices.csv]({{ '/assets/demo/01-primer/offices.csv' | absolute_url }}) and save it to file: `src/main/resources/offices.csv`

1. Add the [commons-csv dependency](https://mvnrepository.com/artifact/org.apache.commons/commons-csv)

   ```groovy
   dependencies {
     /* CSV */
     implementation 'org.apache.commons:commons-csv:1.8'
   }
   ```

1. Add test

   Create the test file: `src/test/java/demo/boot/ContactUsServiceTest.java`

   ```java
   package demo.boot;

   import org.junit.jupiter.api.DisplayName;
   import org.junit.jupiter.api.Test;

   import java.util.List;

   import static org.junit.jupiter.api.Assertions.assertEquals;
   import static org.junit.jupiter.api.Assertions.assertTrue;

   @DisplayName( "Contact us service" )
   public class ContactUsServiceTest {

     @Test
     @DisplayName( "should parse the offices from CSV file" )
     public void shouldParseCsv() {
       final ContactUsService service = new ContactUsService();
       final List<Office> offices = service.list();

       final Office cologne = new Office( "ThoughtWorks Cologne", "Lichtstr. 43i, 50825 Cologne, Germany", "+49 221 64 30 70 63", "contact-de@thoughtworks.com" );
       final Office london = new Office( "ThoughtWorks London", "76 Wardour Street, London W1F 0UR, UK", "+44 (0)20 3437 0990", null );

       assertEquals( 6, offices.size() );
       assertEquals( cologne, offices.get( 0 ) );
       assertEquals( london, offices.get( 4 ) );
     }
   }
   ```

   Add the service skeleton (_just enough to make the test compiles_).

   Create file `src/main/java/demo/boot/ContactUsService.java`

   ```java
   package demo.boot;

   import java.util.List;

   public class ContactUsService {

     public List<Office> list() {
       throw new UnsupportedOperationException();
     }
   }
   ```

   Run the test.

   ```bash
   $ ./gradlew clean test

   ...
   Contact us service > should parse the offices from CSV file FAILED
   ...
   ```

   The test should fail.

1. Implement the service

   Update file `src/main/java/demo/boot/ContactUsService.java`

   ```java
   package demo.boot;

   import org.apache.commons.csv.CSVFormat;
   import org.apache.commons.csv.CSVRecord;

   import java.io.BufferedReader;
   import java.io.IOException;
   import java.io.InputStreamReader;
   import java.nio.charset.StandardCharsets;
   import java.util.List;
   import java.util.stream.Collectors;

   public class ContactUsService {

     public List<Office> list() {
       try ( final BufferedReader reader = readCsv() ) {
         return CSVFormat.DEFAULT
           .withFirstRecordAsHeader()
           .withNullString( "" )
           .parse( reader )
           .getRecords()
           .stream()
           .map( this::parseOffice )
           .collect( Collectors.toList() );
       } catch ( IOException e ) {
         throw new RuntimeException( "Failed to read and parse offices", e );
       }
     }

     private Office parseOffice( final CSVRecord record ) {
       return new Office( record.get( "Name" ),
         record.get( "Address" ),
         record.get( "Phone" ),
         record.get( "Email" ) );
     }

     private BufferedReader readCsv() {
       return new BufferedReader(
         new InputStreamReader(
           getClass().getResourceAsStream( "/offices.csv" ),
           StandardCharsets.UTF_8 )
       );
     }
   }
   ```

## Use service

1. Create a test to assert the relation between the service (`ContactUsService`) and the controller (`OfficeController`).

   {% include custom/note.html details="The following example takes advantage of <a href='https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-testing'>Spring Boot Test</a>.  <a href='#can-we-test-the-controller-using-other-means'>Later on</a> we will see alternative options to test the controller." %}

   Create test file `src/test/java/demo/boot/OfficeControllerTest.java`

   ```java
   package demo.boot;

   import org.junit.jupiter.api.DisplayName;
   import org.junit.jupiter.api.Test;
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
   import org.springframework.boot.test.mock.mockito.MockBean;
   import org.springframework.test.web.servlet.MockMvc;

   import java.util.List;

   import static org.hamcrest.Matchers.hasSize;
   import static org.hamcrest.Matchers.is;
   import static org.mockito.Mockito.times;
   import static org.mockito.Mockito.verify;
   import static org.mockito.Mockito.when;
   import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
   import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.jsonPath;
   import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

   @DisplayName( "Office controller" )
   @WebMvcTest( OfficeController.class )
   public class OfficeControllerTest {

     @Autowired
     private MockMvc mockMvc;

     @MockBean
     private ContactUsService service;

     @Test
     @DisplayName( "should return the list of offices returned by the service" )
     public void shouldReturnTheOffices() throws Exception {
       final Office cologne =
         new Office( "ThoughtWorks Cologne",
           "Lichtstr. 43i, 50825 Cologne, Germany",
           "+49 221 64 30 70 63",
           "contact-de@thoughtworks.com" );
       when( service.list() ).thenReturn( List.of( cologne ) );

       mockMvc.perform( get( "/offices" ) )
         .andExpect( status().isOk() )
         .andExpect( jsonPath( "$" ).isArray() )
         .andExpect( jsonPath( "$", hasSize( 1 ) ) )
         .andExpect( jsonPath( "$.[0].name", is( cologne.getName() ) ) )
         .andExpect( jsonPath( "$.[0].address", is( cologne.getAddress() ) ) )
         .andExpect( jsonPath( "$.[0].phone", is( cologne.getPhone() ) ) )
         .andExpect( jsonPath( "$.[0].email", is( cologne.getEmail() ) ) )
       ;

       verify( service, times( 1 ) ).list();
     }
   }
   ```

   The test expects the same office that the current controller returns.  Furthermore, the tests expect an interaction with the service too, something that it is not happening yet.  Run the tests.

   ```bash
   $ ./gradlew clean test
   ```

   The tests should fail as the controller is not interacting with the service yet.

   ```bash
   Office controller > should return the list of offices returned by the service FAILED
       org.mockito.exceptions.verification.WantedButNotInvoked at OfficeControllerTest.java:51
   ```

1. Annotate the service with the [`@Service` annotation](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/stereotype/Service.html)

   Update the file `src/main/java/demo/boot/ContactUsService.java`

   ```java
   package demo.boot;

   import org.apache.commons.csv.CSVFormat;
   import org.apache.commons.csv.CSVRecord;
   import org.springframework.stereotype.Service;

   import java.io.BufferedReader;
   import java.io.IOException;
   import java.io.InputStreamReader;
   import java.nio.charset.StandardCharsets;
   import java.util.List;
   import java.util.stream.Collectors;

   @Service
   public class ContactUsService { /* ... */ }
   ```

1. Use the service from the controller

   Update the file `src/main/java/demo/boot/OfficeController.java`

   1. Use Lombok (_preferred approach_)

      ```java
      package demo.boot;

      import lombok.AllArgsConstructor;
      import org.springframework.web.bind.annotation.GetMapping;
      import org.springframework.web.bind.annotation.RestController;

      import java.util.List;

      @RestController
      @AllArgsConstructor
      public class OfficeController {

        private final ContactUsService service;

        @GetMapping( "/offices" )
        public List<Office> offices() {
          return service.list();
        }
      }
      ```

   1. Create the constructor manually

      ```java
      package demo.boot;

      import org.springframework.web.bind.annotation.GetMapping;
      import org.springframework.web.bind.annotation.RestController;

      import java.util.List;

      @RestController
      public class OfficeController {

        private final ContactUsService service;

        public OfficeController( final ContactUsService service ) {
          this.service = service;
        }

        @GetMapping( "/offices" )
        public List<Office> offices() {
          return service.list();
        }
      }
      ```

   Both approaches yield the same thing.  In the first example, Lombok created the constructor for us, while in the second example we create the constructor.

   Spring will create one instance of the service and pass it to the controller.

   Note that when the class has only one constructor, no other annotations are required.  Alternatively, the [`@Autowired`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/beans/factory/annotation/Autowired.html) can be used.

1. Update the test `ContactUsApplicationTests`, as this is expecting only one office

   Update file `src/test/java/demo/boot/ContactUsApplicationTests.java`

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
       final Office cologne =
         new Office( "ThoughtWorks Cologne",
           "Lichtstr. 43i, 50825 Cologne, Germany",
           "+49 221 64 30 70 63",
           "contact-de@thoughtworks.com" );

       assertThat( restTemplate.getForObject( "/offices", Office[].class ) )
         .contains( cologne );
     }
   }
   ```

   Instead of checking for all offices, the test just makes sure that the Cologne office is returned with the correct information.

1. Run the tests

   ```bash
   $ ./gradlew clean test
   ```

   All four tests should pass

   ```bash
   ...
   Contact Us application > should return the offices PASSED

   Contact Us application > should return 200 when the health endpoint is accessed PASSED

   Office controller > should return the list of offices returned by the service PASSED

   Contact us service > should parse the offices from CSV file PASSED
   ...
   ```

### Do we need to reset the mocks between tests?

There is no need to reset the beans annotated with the [`@MockBean`](https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/test/mock/mockito/MockBean.html) annotation.  According to the blog post written by [Phillip Webb](https://spring.io/team/pwebb), titled [Testing improvements in Spring Boot 1.4](https://spring.io/blog/2016/04/15/testing-improvements-in-spring-boot-1-4#mocking-and-spying):

"_Mocks will be automatically reset across tests_"

Therefore, there is no need to reset the mocks between tests as Spring takes care of that for you.

### Can we test the controller using other means?

In our [previous example](#use-service), we tested the controller using Spring Boot Test, as shown next.

```java
package demo.boot;

import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.boot.test.mock.mockito.MockBean;
import org.springframework.test.web.servlet.MockMvc;

import java.util.List;

import static org.hamcrest.Matchers.hasSize;
import static org.hamcrest.Matchers.is;
import static org.junit.jupiter.api.Assertions.assertEquals;
import static org.mockito.Mockito.mock;
import static org.mockito.Mockito.reset;
import static org.mockito.Mockito.times;
import static org.mockito.Mockito.verify;
import static org.mockito.Mockito.when;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.jsonPath;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

@DisplayName( "Office controller" )
@WebMvcTest( OfficeController.class )
public class OfficeControllerTest {

  @Autowired
  private MockMvc mockMvc;

  @MockBean
  private ContactUsService service;

  @Test
  @DisplayName( "should return the list of offices returned by the service" )
  public void shouldReturnTheOffices() throws Exception { /* ... */ }
}
```

The annotation [`@WebMvcTest`](https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/test/autoconfigure/web/servlet/WebMvcTest.html) starts our Spring Boot web application without starting the webserver.  The controller under test, `OfficeController`, will be wired and provided the mocked version of the `ContactUsService`.  You can see this in the output produced during tests, where the banner is displayed as shown next.

```bash
...
  _____            _             _     _    _
 / ____|          | |           | |   | |  | |
| |     ___  _ __ | |_ __ _  ___| |_  | |  | |___
| |    / _ \| '_ \| __/ _` |/ __| __| | |  | / __|
| |___| (_) | | | | || (_| | (__| |_  | |__| \__ \
 \_____\___/|_| |_|\__\__,_|\___|\__|  \____/|___/

2077-04-27 12:34:55.005  INFO 10104 --- [    Test worker] demo.boot.OfficeControllerTest           : Starting OfficeControllerTest on Alberts-MBP.fritz.box with PID 10104 (started by albertattard in /Users/albertattard/Downloads/contact-us)
2077-04-27 12:34:55.008  INFO 10104 --- [    Test worker] demo.boot.OfficeControllerTest           : No active profile set, falling back to default profiles: default
2077-04-27 12:34:56.142  INFO 10104 --- [    Test worker] o.s.s.concurrent.ThreadPoolTaskExecutor  : Initializing ExecutorService 'applicationTaskExecutor'
2077-04-27 12:34:56.616  INFO 10104 --- [    Test worker] o.s.b.t.m.w.SpringBootMockServletContext : Initializing Spring TestDispatcherServlet ''
2077-04-27 12:34:56.616  INFO 10104 --- [    Test worker] o.s.t.web.servlet.TestDispatcherServlet  : Initializing Servlet ''
2077-04-27 12:34:56.627  INFO 10104 --- [    Test worker] o.s.t.web.servlet.TestDispatcherServlet  : Completed initialization in 11 ms
2077-04-27 12:34:56.651  INFO 10104 --- [    Test worker] demo.boot.OfficeControllerTest           : Started OfficeControllerTest in 1.97 seconds (JVM running for 2.943)
...
```

However, there are arguments where these types of tests are slow, when compared to unit tests that do not use Spring.  Spring driven tests have an overhead, as Spring Boot starts and configures all parts that are needed before the test runs.

Alternatively, we can test the controller directly, without using Spring, as shown next.

```java
package demo.boot;

import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;

import java.util.List;

import static org.junit.jupiter.api.Assertions.assertEquals;
import static org.mockito.Mockito.mock;
import static org.mockito.Mockito.times;
import static org.mockito.Mockito.verify;
import static org.mockito.Mockito.when;

@DisplayName( "Office controller" )
public class OfficeControllerTest {

  @Test
  @DisplayName( "should return the list of offices returned by the service" )
  public void shouldReturnTheOffices() {
    final Office cologne =
      new Office( "ThoughtWorks Cologne",
        "Lichtstr. 43i, 50825 Cologne, Germany",
        "+49 221 64 30 70 63",
        "contact-de@thoughtworks.com" );
    final List<Office> offices = List.of( cologne );

    final ContactUsService service = mock( ContactUsService.class );
    when( service.list() ).thenReturn( offices );

    final OfficeController controller = new OfficeController( service );
    final List<Office> list = controller.offices();

    assertEquals( offices, list );

    verify( service, times( 1 ) ).list();
  }
}
```

In the above test we are doing the following

1. Creating the mock, instead of relying on Spring to create it for us

   ```java
       final ContactUsService service = mock( ContactUsService.class );
   ```

   The mock is then configured to behave as we need it to behave, as we did with Spring.

   ```java
       when( service.list() ).thenReturn( offices );
   ```

1. Create the controller and provide it with the mocked service, instead of relying on Spring to create it for us

   ```java
       final OfficeController controller = new OfficeController( service );
   ```

   Call the controller's `office()` method, instead if making a REST request

   ```java
       final List<Office> list = controller.offices();
   ```

   Finally, compare the actual list, instead of the JSON response.

   ```java
       assertEquals( offices, list );
   ```

1. Verify the mock

   ```java
       verify( service, times( 1 ) ).list();
   ```


On average, the Spring Boot Test version takes about 6 seconds to run while the version shown above takes about 2 seconds.  The tests themselves take far less time to run, as shown in the following table.

| Type           | Overhead | Test  |
| -------------- | -------: | ----: |
| Spring         |       6s | 200ms |
| Without Spring |       2s |  50ms |

The speed difference between these tests is not negligible, especially for an application that has lots of endpoints.

While the tests that do not make use of Spring are faster, here we are not comparing like with like.  The test that does not involve Spring will still pass even if we remove the REST related annotations (`@RestController` and `@GetMapping`) from the controller, as shown next.

{% include custom/proceed_with_caution.html details="Do not update the controller as this is shown here for demonstration purpose.  The controller should have all the annotations as otherwise it will not work." %}

```java
package demo.boot;

import lombok.AllArgsConstructor;
// import org.springframework.web.bind.annotation.GetMapping;
// import org.springframework.web.bind.annotation.RestController;

import java.util.List;

// @RestController
@AllArgsConstructor
public class OfficeController {

  private final ContactUsService service;

  // @GetMapping( "/offices" )
  public List<Office> offices() {
    return service.list();
  }
}
```

The Spring version of the test will fail as no controller is found to handle the request and a 404 is returned instead of a 200.

Despite being slower, I prefer the Spring version of the test for the reasons listed below:

1. The Spring version of the test cover both the controller logic and its wiring (annotations), which is part of the same class.
1. If we **only** opt for the non-Spring tests, then we will have parts of the application that are not tested.  Our application will still pass but
1. If we create _integration like_ tests to cover the annotations, we will duplicate the tests.

The REST controller acts as a bridge between the outside world and our application and enables communication via HTTP/REST as shown next.

![REST, MQ and Service]({{ '/assets/images/REST-MQ-Service.png' | absolute_url }})

The REST controller simply converts the HTTP/REST requests to Java objects and then invokes the service, where the application business logic resides.  Our application can later be connected to a message queue and a new bridge will be added to connect our service to the message queue.

The REST controller may use [path variables](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/web/bind/annotation/PathVariable.html), or [request parameters](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/web/bind/annotation/RequestParam.html), each of which needs to b covered by tests.  In some cases, these parameters may be optional and have a default value while other times these parameters are mandatory.  Testing such cases outside Spring is not possible as the logic behind the validation is performed by Spring, through the annotations used.

## Tasks status

The application now returns all offices found in the CSV file

- [X] Health endpoint
- [X] OpenAPI
- [X] Return one office contact details
- [X] Dockerize application
- [X] Return all offices contact details
