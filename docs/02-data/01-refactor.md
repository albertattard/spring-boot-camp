---
layout: default
title: Introduce Service Interface
parent: Spring Data
nav_order: 1
permalink: docs/data/refactor/
---

# Introduce Service Interface
{: .no_toc }

We need to started reading the data from the database instead of a CSV file.  To ensure a smooth refactoring we will introduce an interface to isolate the contract (_return a list of offices_) from the implementation (_read from CSV_ and _read from database_).

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Refactoring

1. Rename `ContactUsService` to `CsvContactUsService` and the test from `ContactUsServiceTest` to `CsvContactUsServiceTest`

1. Create a new interface `ContactUsService`

   ```java
   package demo.boot;

   import java.util.List;

   public interface ContactUsService {
     List<Office> list();
   }
   ```

   Do not mark this interface as [`@FunctionalInterface`](https://docs.oracle.com/en/java/javase/14/docs/api/java.base/java/lang/FunctionalInterface.html) as we will add more methods to it.

1. Implement the new interface

   Update file `src/main/java/demo/boot/CsvContactUsService.java`

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
   public class CsvContactUsService implements ContactUsService {

     @Override
     public List<Office> list() { /* ... */ }

     private Office parseOffice( final CSVRecord record ) { /* ... */ }

     private BufferedReader readCsv() { /* ... */ }
   }
   ```

   Do not forget to add the [`@Override`](https://docs.oracle.com/en/java/javase/14/docs/api/java.base/java/lang/Override.html) to the `list()` method.

1. Use the interface (`ContactUsService`) instead of the implementation (`CsvContactUsService`)

   1. Refactor the `OfficeController` class

      Update file `src/main/java/demo/boot/OfficeController.java`

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
        public List<Office> offices() { /* ... */ }
      }
      ```

   1. Refactor the `OfficeControllerTest` test class

      Update file `src/test/java/demo/boot/OfficeControllerTest.java`

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
        public void shouldReturnTheOffices() throws Exception { /* ... */ }
      }
      ```

1. The project structure

   ```bash
   $ tree .
   .
   ...
   └── src
       ├── main
       │   ├── java
       │   │   └── demo
       │   │       └── boot
       │   │           ├── ContactUsApplication.java
       │   │           ├── ContactUsService.java
       │   │           ├── CsvContactUsService.java
       │   │           ├── Office.java
       │   │           └── OfficeController.java
       │   └── resources
       │       ├── application.yaml
       │       ├── banner.txt
       │       └── offices.csv
       └── test
           └── java
               └── demo
                   └── boot
                       ├── ContactUsApplicationTests.java
                       ├── CsvContactUsServiceTest.java
                       └── OfficeControllerTest.java

   12 directories, 19 files
   ```

1. Build the project

   ```bash
   ./gradlew clean build

   ...
   BUILD SUCCESSFUL in 7s
   6 actionable tasks: 6 executed
   ```

   The project should build and run successfully.
