---
layout: default
title: Java Persistence API (JPA)
parent: Spring Data
nav_order: 3
permalink: docs/data/jpa/
---

# Java Persistence API (JPA)
{: .no_toc }

The application makes use of a CSV file to read the list of offices.  In this page we will convert the data source from CSV to database and will take advantage of Spring Boot to minimize the amount of boilerplate code required.

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## What is JPA?

The Java Persistence API (JPA) provides a set of annotations that can be used to map Java objects to database tables, as showning the following image.

![JPA-Read-From-Table]({{ '/assets/images/JPA-Read-From-Table.png' | absolute_url }})

JPA was initially released as a part of the EJB 3.0 specification ([JSR 220](https://jcp.org/en/jsr/detail?id=220)), but then moved to a separate specification ([JSR 317](https://jcp.org/en/jsr/detail?id=317)).

JPA is nothing but specification.  There are different implementations, [Hibernate](https://hibernate.org/) being the most popular, according to [Google trends](https://trends.google.com/trends/explore?q=Hibernate,EclipseLink,OpenJPA,DataNucleus).

![Google-Trends-Hibernate-And-The-Rest.png]({{ '/assets/images/Google-Trends-Hibernate-And-The-Rest.png' | absolute_url }})

It is worth noting that [EclipseLink](https://www.eclipse.org/eclipselink/) is the reference implementation of JPA.

## JPA Entity

Entities are classes that map to a database table.

1. Create the entity class mapping the `offices` table

   Create file: `src/main/java/demo/boot/OfficeEntity.java`

   ```java
   package demo.boot;

   import lombok.Data;

   import javax.persistence.Entity;
   import javax.persistence.Id;
   import javax.persistence.Table;

   @Data
   @Entity
   @Table( name = "offices" )
   public class OfficeEntity {

     @Id
     private String office;
     private String address;
     private String country;
     private String phone;
     private String email;
     private String webpage;
   }
   ```

Classes that need to be mapped with a table, need to be annotated with the [`@Entity`](https://docs.oracle.com/javaee/7/api/javax/persistence/Entity.html) annotation.  Note that the above class only defines the properties and the properties name matches the table's column names.  JPA uses [convention over configuration](https://en.wikipedia.org/wiki/Convention_over_configuration) to minimise the amount of configuration required.

Given that the table name (`offices`) is different from the class name (`OfficeEntity`), we need to specify the table name through the [`@Table`](https://docs.oracle.com/javaee/7/api/javax/persistence/Table.html) annotation, as shown next.

```java
@Table( name = "offices" )
```

JPA requires a primary key.  This is annotated with the [`@Id`](https://docs.oracle.com/javaee/7/api/javax/persistence/Id.html) annotation, as shown next.

```java
@Id
private String office;
```

The properties name can have a different name from the table column name, in which case we need to use the [`@Column`](https://docs.oracle.com/javaee/7/api/javax/persistence/column.html) annotation and provide the table column name.

The `OfficeEntity` class uses [Lombok](https://projectlombok.org/) to generate the usual methods through the [`@Data`](https://projectlombok.org/api/lombok/Data.html) annotation.  This reduces the amount of code that we have to write.

## Repository

Spring Data introduced [repositories](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#reference) to simplify the interaction with the database.

1. Create the JPA repository

   Create file: `src/main/java/demo/boot/OfficesRepository.java`

   ```java
   package demo.boot;

   import org.springframework.data.jpa.repository.JpaRepository;
   import org.springframework.stereotype.Repository;

   @Repository
   public interface OfficesRepository extends JpaRepository<OfficeEntity, String> {
   }
   ```

That's it!!

{% include custom/note.html details="Note that the <code>OfficesRepository</code> is an interface and not a class.  Spring Data will implement this interface for us and it provides several useful methods too, that allows us to read from and write to the <code>offices</code> table, through the <code>OfficeEntity</code> entity." %}

In the following examples, we will use the [`findAll()`](https://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/repository/CrudRepository.html#findAll--) method, which will return all rows in the `offices` table as instances of the `OfficeEntity` class.

## Use the JPA repository

Read all rows in the `offices` table using the JPA repository.

1. Create new service that will be backed by the JPA repository

   Create file: `src/main/java/demo/boot/JpaContactUsService.java`

   ```java
   package demo.boot;

   import lombok.AllArgsConstructor;

   import java.util.List;

   @AllArgsConstructor
   public class JpaContactUsService implements ContactUsService {

     private final OfficesRepository repository;

     @Override
     public List<Office> list() {
       throw new UnsupportedOperationException();
     }
   }
   ```

   The `JpaContactUsService` require an instance of `OfficesRepository` which will be provided by Spring.  Note that the above class, also takes advantage from Lombok's [`@AllArgsConstructor`](https://projectlombok.org/api/lombok/AllArgsConstructor.html) annotation.

1. Create the test

   Create file: `src/test/java/demo/boot/JpaContactUsServiceTest.java`

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

   @DisplayName( "JPA contact us service" )
   public class JpaContactUsServiceTest {

     @Test
     @DisplayName( "should return all offices returned by the repository" )
     public void shouldReturnOffices() {
       final List<OfficeEntity> entities = List.of(
         new OfficeEntity( "a1", "a2", "a3", "a4", "a5", "a6" ),
         new OfficeEntity( "b1", "b2", "b3", "b4", "b5", "b6" )
       );

       final OfficesRepository repository = mock( OfficesRepository.class );
       when( repository.findAll() ).thenReturn( entities );

       final ContactUsService service = new JpaContactUsService( repository );
       final List<Office> offices = service.list();

       final List<Office> expected = List.of(
         new Office( "a1", "a2", "a4", "a5" ),
         new Office( "b1", "b2", "b4", "b5" )
       );

       assertEquals( expected, offices );

       verify( repository, times( 1 ) ).findAll();
     }
   }
   ```

   The above test does not interact with the real repository.  Instead, it uses a mocked version which ensures that our service interacts with the repository as expected, without the need of a database.

   Run the tests.

   ```bash
   $ ./gradlew clean build

   ...
   BUILD FAILED in 9s
   6 actionable tasks: 6 executed
   ```

   The test will fail for two reasons.

   1. The `list()` method throws an exception
   1. The `list()` method is not interacting with the mocked repository as expected.

1. Implement the service

   Update file: `src/main/java/demo/boot/JpaContactUsService.java`

   ```java
   package demo.boot;

   import lombok.AllArgsConstructor;

   import java.util.List;
   import java.util.function.Function;
   import java.util.stream.Collectors;

   @AllArgsConstructor
   public class JpaContactUsService implements ContactUsService {

     private final OfficesRepository repository;

     @Override
     public List<Office> list() {
       return repository
         .findAll()
         .stream()
         .map( mapToOffice() )
         .collect( Collectors.toList() );
     }

     private Function<OfficeEntity, Office> mapToOffice() {
       return entity -> new Office(
         entity.getOffice(),
         entity.getAddress(),
         entity.getPhone(),
         entity.getEmail()
       );
     }
   }
   ```

   Our service retrieves all the records from the `offices` tables using the `findAll()` method.  We did not write the code for the `findAll()` method.  This was provided by Spring Boot, together with many other methods as shown next.

   ![Repository-Default-Methods.png]({{ '/assets/images/Repository-Default-Methods.png' | absolute_url }})

   Run the tests

   ```bash
   $ ./gradlew clean build

   ...
   BUILD SUCCESSFUL in 9s
   6 actionable tasks: 6 executed
   ```

   All tests should pass.

## Use the new service

Note that now we will have two implementations for the same service as shown in the following image.

![Two Implementations.png]({{ '/assets/images/Two-Implementations.png' | absolute_url }})

We need to tell Spring which one is the preferred service that needs to be picked.  Spring cannot pick one by chance.

1. Mark the service

   Update file: `src/main/java/demo/boot/JpaContactUsService.java`

   ```java
   package demo.boot;

   import lombok.AllArgsConstructor;
   import org.springframework.context.annotation.Primary;
   import org.springframework.stereotype.Service;

   import java.util.List;
   import java.util.function.Function;
   import java.util.stream.Collectors;

   @Service
   @Primary
   @AllArgsConstructor
   public class JpaContactUsService implements ContactUsService { /* ... */ }
   ```

   Note that we have to use the [`@Primary`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/annotation/Primary.html) annotation to mark our JPA service as the main service.

   Without the `@Primary` annotation Spring will not know which of the two services to use and will not start.

## View the generated SQL code

Sometimes is it very useful to see what SQL code was generated and used to run a query.  We can enable this through configuration.

1. Show SQL used

   Update file: `src/main/resources/application.yaml`

   ```yaml
   spring:

     jpa:
       show-sql: true
       properties:
         hibernate:
           format_sql: true
       hibernate:
         ddl-auto: validate
   ```

   Following is the complete example of the `application.yaml` file.

   ```yaml
   spring:
     datasource:
       url: jdbc:h2:mem:contact-us;DATABASE_TO_UPPER=false;
       driver-class-name: org.h2.Driver
       username: tw-data
       password: SomeRandomPassword

     jpa:
       show-sql: true
       properties:
         hibernate:
           format_sql: true
       hibernate:
         ddl-auto: validate

   management:
     endpoints:
       web:
         base-path:
         path-mapping:
           health: /health
   ```

   When running the tests, or the application, you will see the generated SQL in the logs, similar to the following.

   ```bash
   Hibernate:
       select
           officeenti0_.office as office1_0_,
           officeenti0_.address as address2_0_,
           officeenti0_.country as country3_0_,
           officeenti0_.email as email4_0_,
           officeenti0_.phone as phone5_0_,
           officeenti0_.webpage as webpage6_0_
       from
           offices officeenti0_
   ```

## H2 console

[H2 console](http://www.h2database.com/html/quickstart.html#h2_console) is a web SQL tool that can connect to the in-memory database configured above.  Note that we cannot connect to the in-memory database using any other application as the database resides in the memory of the Contact Us application.  No other application can read this memory area.

1. Enable H2 console

   Update file: `src/main/resources/application.yaml`

   ```yaml
   spring:

     h2:
       console:
         enabled: true
         path: /h2
   ```

   Following is the complete example of the `application.yaml` file.

   ```yaml
   spring:
     datasource:
       url: jdbc:h2:mem:contact-us;DATABASE_TO_UPPER=false;
       driver-class-name: org.h2.Driver
       username: tw-data
       password: SomeRandomPassword

     jpa:
       show-sql: true
       properties:
         hibernate:
           format_sql: true
       hibernate:
         ddl-auto: validate

     h2:
       console:
         enabled: true
         path: /h2

   management:
     endpoints:
       web:
         base-path:
         path-mapping:
           health: /health
   ```

1. Access the H2 Console ([http://localhost:8080/h2](http://localhost:8080/h2))

   You need to use the same credentials as defined in the `application.yaml` file.

   | Name     | Value                    |
   | -------- | ------------------------ |
   | JDBC URL | `jdbc:h2:mem:contact-us` |
   | Username | `tw-data`                |
   | Password | `SomeRandomPassword`     |

   Note that there is no need to provide the connection flags to the `JDBC URL` field.

   ![H2 Console]({{ '/assets/images/H2-Console.png' | absolute_url }})

## Remove the CSV related code

Our application will only use the database and will not rely on the CSV anymore.  Therefore this code can be safely deleted.

1. Delete the three CSV related files

   ```bash
   $ rm src/main/resources/offices.csv
   $ rm src/main/java/demo/boot/CsvContactUsService.java
   $ rm src/test/java/demo/boot/CsvContactUsServiceTest.java
   ```

1. Remove the [`commons-csv` dependency](https://mvnrepository.com/artifact/org.apache.commons/commons-csv) from the dependencies as this is not required anymore.

   Update the file `build.gradle`, by removing the `org.apache.commons:commons-csv:1.8` dependency.

   ```groovy
     /* CSV */
     implementation 'org.apache.commons:commons-csv:1.8'
   ```

   The following fragment shows the remaining dependencies.

   ```groovy
   dependencies {
     /* Lombok */
     compileOnly 'org.projectlombok:lombok'
     annotationProcessor 'org.projectlombok:lombok'

     /* Spring */
     implementation 'org.springframework.boot:spring-boot-starter-web'
     implementation 'org.springframework.boot:spring-boot-starter-actuator'
     testImplementation('org.springframework.boot:spring-boot-starter-test') {
       exclude group: 'org.junit.vintage', module: 'junit-vintage-engine'
     }

     /* Data */
     implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
     runtimeOnly 'org.flywaydb:flyway-core'
     runtimeOnly 'com.h2database:h2'

     /* OpenApi/Swagger */
     implementation 'org.springdoc:springdoc-openapi-ui:1.3.9'
   }
   ```

1. Build the project

   ```bash
   $ ./gradlew clean build

   ...
   BUILD FAILED in 9s
   6 actionable tasks: 6 executed
   ```

   All tests should pass
