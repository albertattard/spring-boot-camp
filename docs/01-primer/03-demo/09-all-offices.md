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
       return new Office( record.get( "Office" ),
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

1. Use the service from the controller

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
         .andExpect( jsonPath( "$.[0].office", is( cologne.getOffice() ) ) )
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

## Tasks status

The application now returns all offices found in the CSV file

- [X] Health endpoint
- [X] OpenAPI
- [X] Return one office contact details
- [X] Dockerize application
- [X] Return all offices contact details
