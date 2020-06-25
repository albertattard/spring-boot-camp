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

   import lombok.AllArgsConstructor;
   import lombok.Data;
   import lombok.NoArgsConstructor;

   import javax.persistence.Entity;
   import javax.persistence.Id;
   import javax.persistence.Table;

   @Data
   @Entity
   @Table( name = "offices" )
   @AllArgsConstructor
   @NoArgsConstructor
   public class OfficeEntity {

     @Id
     private String name;
     private String address;
     private String country;
     private String phone;
     private String email;
     private String webpage;
   }
   ```

Classes that need to be mapped with a table, need to be annotated with the [`@Entity`](https://docs.oracle.com/javaee/7/api/javax/persistence/Entity.html) annotation.  The above class only defines the properties and the properties name matches the table's column names.  JPA uses [convention over configuration](https://en.wikipedia.org/wiki/Convention_over_configuration) to minimise the amount of configuration required.

Given that the table name (`offices`) is different from the class name (`OfficeEntity`), we need to specify the table name through the [`@Table`](https://docs.oracle.com/javaee/7/api/javax/persistence/Table.html) annotation, as shown next.

```java
@Table( name = "offices" )
```

JPA requires a primary key.  This is annotated with the [`@Id`](https://docs.oracle.com/javaee/7/api/javax/persistence/Id.html) annotation, as shown next.

```java
@Id
private String name;
```

The properties name can have a different name from the table column name, in which case we need to use the [`@Column`](https://docs.oracle.com/javaee/7/api/javax/persistence/column.html) annotation and provide the table column name.

The `OfficeEntity` class uses [Lombok](https://projectlombok.org/) to generate the usual methods through the [`@Data`](https://projectlombok.org/api/lombok/Data.html) annotation.  This reduces the amount of code that we have to write.  The [`@AllArgsConstructor`](https://projectlombok.org/api/lombok/AllArgsConstructor.html) and [`@NoArgsConstructor`](https://projectlombok.org/api/lombok/NoArgsConstructor.html) annotations are added to simplify the creation of the `OfficeEntity` class.

{% include custom/note.html details="JPA required the entities to have the default constructor." %}

### (_Optional_) How can we test the JPA entities?

JPA entities are rarely tested individually as these are usually tested together with the [repository](#jpa-repository) as part of the feature.  My preferred approach is to test the entities together with the repository as these two are tightly coupled.

{% include custom/note.html details="Do not implement this test if you are going to make use of <a href='#jpa-repository'>JPA repositories</a>, as we will be repeating the test later on.  This test is only shown here for completeness." %}

We can write a simple test to ensure that the entity is properly configured.

1. Add a test class that selects entities from the database

   Create file: `src/test/java/demo/boot/OfficeEntityTest.java`

   ```java
   package demo.boot;

   import org.junit.jupiter.api.BeforeEach;
   import org.junit.jupiter.api.DisplayName;
   import org.junit.jupiter.api.Test;
   import org.springframework.boot.test.autoconfigure.jdbc.AutoConfigureTestDatabase;
   import org.springframework.boot.test.autoconfigure.orm.jpa.DataJpaTest;

   import javax.persistence.EntityManager;
   import javax.persistence.PersistenceContext;
   import java.util.List;

   import static org.assertj.core.api.Assertions.assertThat;

   @DataJpaTest
   @DisplayName( "Office entity" )
   @AutoConfigureTestDatabase( replace = AutoConfigureTestDatabase.Replace.NONE )
   public class OfficeEntityTest {

     @PersistenceContext
     private EntityManager entityManager;

     private static final OfficeEntity COLOGNE = new OfficeEntity(
       "ThoughtWorks Cologne",
       "Lichtstr. 43i, 50825 Cologne, Germany",
       "Germany",
       "+49 221 64 30 70 63",
       "contact-de@thoughtworks.com",
       "https://www.thoughtworks.com/locations/cologne"
     );

     private static final OfficeEntity MANCHESTER = new OfficeEntity(
       "ThoughtWorks 'ThoughtWorks Manchester'",
       "4th Floor Federation House, 2 Federation St., Manchester M4 4BF, UK",
       "UK",
       "+44 (0)161 923 6810",
       null,
       "https://www.thoughtworks.com/locations/manchester"
     );

     @BeforeEach
     public void before() {
       entityManager.createQuery( "DELETE FROM OfficeEntity" ).executeUpdate();
       entityManager.persist( COLOGNE );
       entityManager.persist( MANCHESTER );
     }

     @Test
     @DisplayName( "should select all offices" )
     public void shouldSelectAll() {
       final List<OfficeEntity> entities = entityManager
         .createQuery( "SELECT c FROM OfficeEntity c", OfficeEntity.class )
         .getResultList();

       assertThat( entities.size() ).isEqualTo( 2 );
       assertThat( entities ).contains( COLOGNE );
       assertThat( entities ).contains( MANCHESTER );
     }

     @Test
     @DisplayName( "should select the Cologne office" )
     public void shouldSelectCologne() {
       final OfficeEntity entity = entityManager
         .createQuery( "SELECT c FROM OfficeEntity c WHERE c.name LIKE :name", OfficeEntity.class )
         .setParameter( "name", COLOGNE.getName() )
         .getSingleResult();

       assertThat( entity ).isEqualTo( COLOGNE );
     }
   }
   ```

   The above test class has two test cases, one that selects all offices and the other one that selects one office.

   1. Take advantage from [Spring Boot Test](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-testing)

       ```java
       @DataJdbcTest
       @AutoConfigureTestDatabase( replace = AutoConfigureTestDatabase.Replace.NONE )
       ```

      These two annotations prepare our application as required by the test.  Usually we can do with just the [`@DataJpaTest`](https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/test/autoconfigure/orm/jpa/DataJpaTest.html) annotation, but given that we do not want to swap the actual database with an embedded one, we need to fine tune our test setup using the [`@AutoConfigureTestDatabase`](https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/test/autoconfigure/jdbc/AutoConfigureTestDatabase.html) annotation.

      Using the `@AutoConfigureTestDatabase` annotation we can customise the test setup and use the production database, by setting the [`replace`](https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/test/autoconfigure/jdbc/AutoConfigureTestDatabase.html#replace--) parameter to [`AutoConfigureTestDatabase.Replace.NONE`](https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/test/autoconfigure/jdbc/AutoConfigureTestDatabase.Replace.html#NONE).

      One of the great advantages of `@DataJpaTest` annotation is that it includes [`@Transactional`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/transaction/annotation/Transactional.html) annotation.  When used within tests, the `@Transactional` annotation rollbacks the database transaction at the end of each test, and thus any changes made will be ignored.  For example, if we delete all data from a table, these will be only deleted for the duration of the test and these changes are rolled back after each test.  This is quite convenient as the database is left in the same state as it was before the test is started.

      {% include custom/note.html details="Our test depends on the database being ready.  Spring Boot will first run the flyway migration and then will run our tests." %}

   1. Entity Manager

      ```java
        @PersistenceContext
        private EntityManager entityManager;
      ```

      An [`EntityManager`](https://docs.oracle.com/javaee/7/api/javax/persistence/EntityManager.html) is responsible for the lifecycle of entities.  Our application is using an instance of `EntityManager` under the hood, to read data from the `offices` table.  The repositories interact with an instance of the `EntityManager` which is conveniently setup and configured by [Spring Boot](https://spring.io/projects/spring-boot).  The `EntityManager` is used within this test to be able to empty the table and add new offices to prepare the test data for our tests.

      We can interact with the database directly using SQL, but that may produce unexpected results.  The `EntityManager` may cache data and when the database is modified outside of the `EntityManager`, the repository may be dealing stale data.

   1. Setup the data

      ```java
        @BeforeEach
        public void before() {
          entityManager.createQuery( "DELETE FROM OfficeEntity" ).executeUpdate();
          entityManager.persist( COLOGNE );
          entityManager.persist( MANCHESTER );
        }
      ```

      Empty the table first and adds two offices.  This makes sure that the table will only have two offices for the duration of the tests.

      {% include custom/note.html details="These changes are rolled back at the end of the test." %}

   1. Query the repository

      Select both offices

      ```java
        @Test
        @DisplayName( "should select all offices" )
        public void shouldSelectAll() {
          final List<OfficeEntity> entities = entityManager
            .createQuery( "SELECT c FROM OfficeEntity c", OfficeEntity.class )
            .getResultList();

          assertThat( entities.size() ).isEqualTo( 2 );
          assertThat( entities ).contains( COLOGNE );
          assertThat( entities ).contains( MANCHESTER );
        }
      ```

      Select one office

      ```java
        @Test
        @DisplayName( "should select the Cologne office" )
        public void shouldSelectCologne() {
          final OfficeEntity entity = entityManager
            .createQuery( "SELECT c FROM OfficeEntity c WHERE c.name LIKE :name", OfficeEntity.class )
            .setParameter( "name", COLOGNE.getName() )
            .getSingleResult();

          assertThat( entity ).isEqualTo( COLOGNE );
        }
      ```

      These tests interact with the database using JPA and verify that our entity is properly annotated.

1. Run the test

   ```bash
   $ ./gradlew clean test

   ...

   BUILD SUCCESSFUL in 14s
   6 actionable tasks: 6 executed
   ```

   All tests should pass.

### Should we take advantage of Hibernate validation to test our entity?

Hibernate can be configured to validate our entities before these are used, through the `src/main/resources/application.yaml` properties file as shown next.

```yaml
spring:

  jpa:
    hibernate:
      ddl-auto: validate
```

This is quite good as it ensures that our entities are properly annotated.  With that said, Hibernate validations are **not** standalone tests and these are triggered when our application starts.  Any test that starts the application will also trigger these validations and the test may fail for a different reason from which it was designed for.

My preferred approach is to test the entities together with the repository as these two are tightly coupled.

## JPA Repository

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

{% include custom/note.html details="The <code>OfficesRepository</code> is an interface and not a class.  Spring Data will implement this interface for us and it provides several useful methods too, that allows us to read from and write to the <code>offices</code> table, through the <code>OfficeEntity</code> entity." %}

In the following examples, we will use the [`findAll()`](https://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/repository/CrudRepository.html#findAll--) method, which will return all rows in the `offices` table as instances of the `OfficeEntity` class.

### Should we test the JPA repository?

**Yes**.

While it is tempted not to test the JPA repository, `OfficesRepository`, given that this interface has no code, we need to make sure that our entity and repository are working as expected together.

1. Create JPA repository test

   Create file: `src/test/java/demo/boot/OfficesRepositoryTest.java`

   ```java
   package demo.boot;

   import org.junit.jupiter.api.BeforeEach;
   import org.junit.jupiter.api.DisplayName;
   import org.junit.jupiter.api.Test;
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.boot.test.autoconfigure.jdbc.AutoConfigureTestDatabase;
   import org.springframework.boot.test.autoconfigure.orm.jpa.DataJpaTest;

   import javax.persistence.EntityManager;
   import javax.persistence.PersistenceContext;
   import java.util.List;

   import static org.assertj.core.api.Assertions.assertThat;

   @DataJpaTest
   @DisplayName( "Offices repository" )
   @AutoConfigureTestDatabase( replace = AutoConfigureTestDatabase.Replace.NONE )
   public class OfficesRepositoryTest {

     @PersistenceContext
     private EntityManager entityManager;

     @Autowired
     private OfficesRepository repository;

     private static final OfficeEntity COLOGNE = new OfficeEntity(
       "ThoughtWorks Cologne",
       "Lichtstr. 43i, 50825 Cologne, Germany",
       "Germany",
       "+49 221 64 30 70 63",
       "contact-de@thoughtworks.com",
       "https://www.thoughtworks.com/locations/cologne"
     );

     private static final OfficeEntity MANCHESTER = new OfficeEntity(
       "ThoughtWorks 'ThoughtWorks Manchester'",
       "4th Floor Federation House, 2 Federation St., Manchester M4 4BF, UK",
       "UK",
       "+44 (0)161 923 6810",
       null,
       "https://www.thoughtworks.com/locations/manchester"
     );

     @BeforeEach
     public void before() {
       entityManager.createQuery( "DELETE FROM OfficeEntity" ).executeUpdate();
       entityManager.persist( COLOGNE );
       entityManager.persist( MANCHESTER );
     }

     @Test
     @DisplayName( "should return all offices in the table" )
     public void shouldReturnAll() {
       final List<OfficeEntity> offices = repository.findAll();

       assertThat( offices.size() ).isEqualTo( 2 );
       assertThat( offices ).contains( COLOGNE );
       assertThat( offices ).contains( MANCHESTER );
     }
   }
   ```

   {% include custom/note.html details="The test shown above is very similar to the <code>OfficeEntityTest</code> example shown <a href='#optional-how-can-we-test-the-jpa-entities'>before</a>.<br/>Delete the <code>OfficeEntityTest</code> if you have this test as we have two tests now that are almost doing the same thing.  Have multiple tests covering the same thing make the tests very rigid and makes our code harder to change.  A small refactor may break many tests." %}

   When used together with JPA repositories, entities are automatically tested when testing the JPA repository.  These two objects are tightly coupled by their nature.

   1. Take advantage from [Spring Boot Test](https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-testing)

       ```java
       @DataJdbcTest
       @AutoConfigureTestDatabase( replace = AutoConfigureTestDatabase.Replace.NONE )
       ```

      These two annotations prepare our application as required by the test.  Usually we can do with just the [`@DataJpaTest`](https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/test/autoconfigure/orm/jpa/DataJpaTest.html) annotation, but given that we do not want to swap the actual database with an embedded one, we need to fine tune our test setup using the [`@AutoConfigureTestDatabase`](https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/test/autoconfigure/jdbc/AutoConfigureTestDatabase.html) annotation.

      Using the `@AutoConfigureTestDatabase` annotation we can customise the test setup and use the production database, by setting the [`replace`](https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/test/autoconfigure/jdbc/AutoConfigureTestDatabase.html#replace--) parameter to [`AutoConfigureTestDatabase.Replace.NONE`](https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/test/autoconfigure/jdbc/AutoConfigureTestDatabase.Replace.html#NONE).

      A great advantage of `@DataJpaTest` annotation is that it includes [`@Transactional`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/transaction/annotation/Transactional.html) annotation.  When used within tests, the `@Transactional` annotation rollbacks the database transaction at the end of each test, and thus any changes made will be ignored.  For example, if we delete all data from a table, these will be only deleted for the duration of the test and these changes are rolled back after each test.  This is quite convenient as the database is left in the same state as it was before the test is started.

      {% include custom/note.html details="Our test depends on the database being ready.  Spring Boot will first run the flyway migration and then will run our tests." %}

   1. Entity Manager

      ```java
        @PersistenceContext
        private EntityManager entityManager;
      ```

      An [`EntityManager`](https://docs.oracle.com/javaee/7/api/javax/persistence/EntityManager.html) is responsible for the lifecycle of entities.  Our application is using an instance of `EntityManager` under the hood, to read data from the `offices` table.  The repositories interact with an instance of the `EntityManager` which is conveniently setup and configured by [Spring Boot](https://spring.io/projects/spring-boot).  The `EntityManager` is used within this test to be able to empty the table and add new offices to prepare the test data for our tests.

      We can interact with the database directly using SQL, but that may produce unexpected results.  The `EntityManager` may cache data and when the database is modified outside of the `EntityManager`, the repository may be dealing stale data.

   1. Setup the data

      ```java
        @BeforeEach
        public void before() {
          entityManager.createQuery( "DELETE FROM OfficeEntity" ).executeUpdate();
          entityManager.persist( COLOGNE );
          entityManager.persist( MANCHESTER );
        }
      ```

      Empty the table first and adds two offices.  This makes sure that the table will only have two offices for the duration of the tests.

      {% include custom/note.html details="These changes are rolled back at the end of the test." %}

   1. Query the repository

      ```java
        @Test
        @DisplayName( "should return all offices in the table" )
        public void shouldReturnAll() {
          final List<OfficeEntity> offices = repository.findAll();

          assertThat( offices.size() ).isEqualTo( 2 );
          assertThat( offices ).contains( COLOGNE );
          assertThat( offices ).contains( MANCHESTER );
        }
      ```

      This tests interact with the database using JPA repository and verify that our entity is properly annotated.

1. Run the test

   ```bash
   $ ./gradlew clean test

   ...

   BUILD SUCCESSFUL in 14s
   6 actionable tasks: 6 executed
   ```

   All tests should pass.

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

   The `JpaContactUsService` require an instance of `OfficesRepository` which will be provided by Spring.

   {% include custom/note.html details="The above class also takes advantage from Lombok's <a href='https://projectlombok.org/api/lombok/AllArgsConstructor.html'><code>@AllArgsConstructor</code></a> annotation." %}

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
         entity.getName(),
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

We now have two implementations for the same service as shown in the following image.

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

   {% include custom/note.html details="We have to use the <a href='https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/annotation/Primary.html'><code>@Primary</code></a> annotation to mark our JPA service as the main service.  Without the <code>@Primary</code> annotation Spring will not know which of the two services to use and will not start." %}

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

   Following is the complete example of the `application.yaml` properties file.

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
           officeenti0_.name as name1_0_,
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

   Following is the complete example of the `application.yaml` properties file.

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

   You need to use the same credentials as defined in the `application.yaml` properties file.

   | Name     | Value                    |
   | -------- | ------------------------ |
   | JDBC URL | `jdbc:h2:mem:contact-us` |
   | Username | `tw-data`                |
   | Password | `SomeRandomPassword`     |

   Note that there is no need to provide the connection flags to the `JDBC URL` field.

   ```sql
   SELECT * FROM "offices";
   ```

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

   {% include custom/note.html details="The above example is missing the <code>testImplementation 'org.flywaydb:flyway-core'</code> entry, which needs to be included if Flyway is tested separately, using the <code>FlywayMigrationTest</code> test class discussed <a href='/spring-boot-camp/docs/data/database/#optional-testing-flyway-migration-scripts'>before</a>." %}

1. Remove the `@Primary` annotation as we only have one service

   Update file: `src/main/java/demo/boot/JpaContactUsService.java`

   ```java
   package demo.boot;

   import lombok.AllArgsConstructor;
   import org.springframework.orm.ObjectOptimisticLockingFailureException;
   import org.springframework.retry.annotation.Retryable;
   import org.springframework.stereotype.Service;
   import org.springframework.transaction.annotation.Transactional;

   import java.util.List;
   import java.util.Optional;
   import java.util.function.Function;
   import java.util.stream.Collectors;

   @Service
   @AllArgsConstructor
   public class JpaContactUsService implements ContactUsService { /* ... */ }
   ```

   **Given that we have one implementation, should we remove the interface?**

   We can remove the interface `ContactUsService` and just use the implementation directly instead.

1. Build the project

   ```bash
   $ ./gradlew clean build

   ...
   BUILD FAILED in 9s
   6 actionable tasks: 6 executed
   ```

   All tests should pass
