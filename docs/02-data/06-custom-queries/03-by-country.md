---
layout: default
title: Filter by a country
parent: Custom Queries
grand_parent: Spring Data
nav_order: 3
permalink: docs/data/custom-queries/find-by-country/
---

# Filter by a country
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Filter by a country (JPA query methods)

[Spring Data JPA](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/) provides [query method](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#jpa.query-methods) that we can take advantage from.

We can achieve filtering by simply adding a new method to our repository as shown next.

```java
List<OfficeEntity> findAllByCountryIgnoreCase( final String country );
```

That's it!!

No need to add code, as Spring Data JPA will take care of the rest.  The method name is not random, and Spring Data JPA is using the method name to determine what needs to be done.  Following is a breakdown of the method name (`findAllByCountryIgnoreCase`)

1. `findAll` (or just `find`): indicates that this method will query the table.  We can prefix the method name with: `read…By`, `query…By`, and `get…By` instead.
1. `By`: indicates that a set of filters will follow
1. `Country`: the property name (not necessarily the column name) by which we will filter
1. `IgnoreCase`: indicate that the filter should be case-insensitive

Note that while we are not adding any actual code, the method name has logic bound to it and thus it needs to be tested like any other code.

1. Create a repository (**integration**) test

   Note that our new test will interact with a PostgreSQL database, thus it is an integration test.

   Create file `src/test-integration/java/demo/boot/OfficesRepositoryTest.java`

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
     @DisplayName( "should return all offices for the given country (case insensitive) " )
     public void shouldReturnAllInCountry() {
       final List<OfficeEntity> offices = repository.findAllByCountryIgnoreCase( "germany" );

       assertThat( offices.size() ).isEqualTo( 1 );
       assertThat( offices ).contains( COLOGNE );
       assertThat( offices ).doesNotContain( MANCHESTER );
     }
   }
   ```

   This test is a bit elaborate, so let's break it down.

   1. Test annotations

      ```java
      @DataJpaTest
      @AutoConfigureTestDatabase( replace = AutoConfigureTestDatabase.Replace.NONE )
      ```

      These two annotations prepare our application for the repository test.  Usually we can do with just the [`@DataJpaTest`](https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/test/autoconfigure/orm/jpa/DataJpaTest.html) annotation, but given that we do not want to swap the actual database with an embedded one (such as [H2](https://www.h2database.com/)), we need to fine tune our test setup.  Without the [`@AutoConfigureTestDatabase`](https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/test/autoconfigure/jdbc/AutoConfigureTestDatabase.html) annotation, this test will fail to run due to the following exception.

      ```bash
      Caused by: java.lang.IllegalStateException: Failed to replace DataSource with an embedded database for tests. If you want an embedded database please put a supported one on the classpath or tune the replace attribute of @AutoConfigureTestDatabase.
          at org.springframework.util.Assert.state(Assert.java:73)
          at org.springframework.boot.test.autoconfigure.jdbc.TestDatabaseAutoConfiguration$EmbeddedDataSourceFactory.getEmbeddedDatabase(TestDatabaseAutoConfiguration.java:179)
          at org.springframework.boot.test.autoconfigure.jdbc.TestDatabaseAutoConfiguration$EmbeddedDataSourceFactoryBean.afterPropertiesSet(TestDatabaseAutoConfiguration.java:145)
          at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.invokeInitMethods(AbstractAutowireCapableBeanFactory.java:1855)
          at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.initializeBean(AbstractAutowireCapableBeanFactory.java:1792)
          ... 115 more
      ```

      Using the `@AutoConfigureTestDatabase` annotation we can customise the test setup and use the original PostgreSQL database, by setting the [`replace`](https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/test/autoconfigure/jdbc/AutoConfigureTestDatabase.html#replace--) parameter to [`AutoConfigureTestDatabase.Replace.NONE`](https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/test/autoconfigure/jdbc/AutoConfigureTestDatabase.Replace.html#NONE).

      One of the great advantages of `@DataJpaTest` annotation is that it includes [`@Transactional`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/transaction/annotation/Transactional.html) annotation.  When used within tests, the `@Transactional` annotation rollbacks the database transaction at the end of each test, and thus any changes made will be ignored.  For example, if we delete all data from a table, these will be only deleted for the duration of the test and these changes are rolled back after each test.  This is quite convenient as the database is left in the same state as it was before the test is started.

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
        @DisplayName( "should return all offices for the given country (case insensitive) " )
        public void shouldReturnAllInCountry() {
          final List<OfficeEntity> offices = repository.findAllByCountryIgnoreCase( "germany" );

          assertThat( offices.size() ).isEqualTo( 1 );
          assertThat( offices ).contains( COLOGNE );
          assertThat( offices ).doesNotContain( MANCHESTER );
        }
      ```

      This test makes sure that our method name, `findAllByCountryIgnoreCase`, works as expected, returns all offices in the given country.

   Unfortunately, this is a case where the number of lines of test code is far too many when compare to the code being tested.  In this example we have a `1:60` line-relation.  With that said, this ensures that our code is behaving as it is expected.

   Note that the above test cannot yet run as we still need to add the new method to our repository.

1. Add the `findAllByCountryIgnoreCase()` method

   Update file: `src/main/java/demo/boot/OfficesRepository.java`

   ```java
   package demo.boot;

   import org.springframework.data.jpa.repository.JpaRepository;
   import org.springframework.stereotype.Repository;

   import java.util.List;

   @Repository
   public interface OfficesRepository extends JpaRepository<OfficeEntity, String> {

     List<OfficeEntity> findAllByCountryIgnoreCase( final String country );
   }
   ```

1. Make sure that PostgreSQL is running

   ```bash
   $ docker ps

   ...
   CONTAINER ID        IMAGE                 COMMAND                  CREATED             STATUS                 PORTS                           NAMES
   5a8316ce8655        postgres:11.1         "docker-entrypoint.s…"   4 hours ago         Up 4 hours (healthy)   0.0.0.0:5432->5432/tcp          contact-us
   363fe0c321cb        dpage/pgadmin4:4.22   "/entrypoint.sh"         4 hours ago         Up 4 hours             443/tcp, 0.0.0.0:8000->80/tcp   pgadmin4
   ```

   You can start the services using the following command, if these are not running.

   ```bash
   $ docker-compose up -d
   ```

1. Run the integration tests

   ```bash
   $ ./gradlew clean integrationTest

   ...
   BUILD SUCCESSFUL in 18s
   6 actionable tasks: 6 executed
   ```

   The integration tests should all pass

### Link to service

A new method will be added to the `ContactUsService`, named `listInCountry( String )`, that will interact with the repository to return only the offices for the given country.

1. Add a new test method

   Update file: `src/test/java/demo/boot/JpaContactUsServiceTest.java`

   ```java
   package demo.boot;

   import org.junit.jupiter.api.DisplayName;
   import org.junit.jupiter.api.Test;

   import java.util.List;

   import static org.junit.jupiter.api.Assertions.assertEquals;
   import static org.mockito.ArgumentMatchers.eq;
   import static org.mockito.Mockito.mock;
   import static org.mockito.Mockito.times;
   import static org.mockito.Mockito.verify;
   import static org.mockito.Mockito.when;

   @DisplayName( "JPA contact us service" )
   public class JpaContactUsServiceTest {

     @Test
     @DisplayName( "should return all offices returned by the repository" )
     public void shouldReturnOffices() { /* ... */ }

     @Test
     @DisplayName( "should return all offices in a given country" )
     public void shouldReturnOfficesInACountry() {
       final List<OfficeEntity> entities = List.of(
         new OfficeEntity( "a1", "a2", "a3", "a4", "a5", "a6" ),
         new OfficeEntity( "b1", "b2", "b3", "b4", "b5", "b6" )
       );

       final String country = "Germany";

       final OfficesRepository repository = mock( OfficesRepository.class );
       when( repository.findAllByCountryIgnoreCase( eq( country ) ) ).thenReturn( entities );

       final ContactUsService service = new JpaContactUsService( repository );
       final List<Office> offices = service.listInCountry( country );

       final List<Office> expected = List.of(
         new Office( "a1", "a2", "a4", "a5" ),
         new Office( "b1", "b2", "b4", "b5" )
       );

       assertEquals( expected, offices );

       verify( repository, times( 1 ) ).findAllByCountryIgnoreCase( country );
     }
   }
   ```

   Add the new method to the `ContactUsService` interface.  We will be adding more methods to this interface and that's why we never declared the interface as a [`@FunctionalInterface`](https://docs.oracle.com/en/java/javase/14/docs/api/java.base/java/lang/FunctionalInterface.html).

   Update file: `src/main/java/demo/boot/ContactUsService.java`

   ```java
   package demo.boot;

   import java.util.List;

   public interface ContactUsService {

     List<Office> list();

     List<Office> listInCountry( final String country );
   }
   ```

   Implement the missing method, just enough to make the test compile.

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
   public class JpaContactUsService implements ContactUsService {

     private final OfficesRepository repository;

     @Override
     public List<Office> list() { /* ... */ }

     @Override
     public List<Office> listInCountry( final String country ) {
       throw new UnsupportedOperationException();
     }

     private Function<OfficeEntity, Office> mapToOffice() { /* ... */ }
   }
   ```

   Run the test

   ```bash
   $ ./gradlew clean test

   ...
   BUILD FAILED in 9s
   5 actionable tasks: 5 executed
   ```

   As expected, the test failed.

1. Implement the missing functionality

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
   public class JpaContactUsService implements ContactUsService {

     private final OfficesRepository repository;

     @Override
     public List<Office> list() { /* ... */ }

     @Override
     public List<Office> listInCountry( final String country ) {
       return repository
         .findAllByCountryIgnoreCase( country )
         .stream()
         .map( mapToOffice() )
         .collect( Collectors.toList() );
     }

     private Function<OfficeEntity, Office> mapToOffice() { /* ... */ }
   }
   ```

   Run the tests

   ```bash
   $ ./gradlew clean test

   ...
   BUILD SUCCESSFUL in 5s
   5 actionable tasks: 5 executed
   ```

   All tests should pass.

1. (_Optional_) Refactor

   We have some code repetition that can be moved to one place.  **The amount of repetition is not that much, and it is perfect to leave things as they are**.  The code is refactored just to show an alternative approach.

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
   public class JpaContactUsService implements ContactUsService {

     private final OfficesRepository repository;

     @Override
     public List<Office> list() {
       return mapToOffices( repository.findAll() );
     }

     @Override
     public List<Office> listInCountry( final String country ) {
       return mapToOffices( repository.findAllByCountryIgnoreCase( country ) );
     }

     private List<Office> mapToOffices( final List<OfficeEntity> entities ) {
       return entities
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

## Can we query the repository based on what the user inputs?

**Yes**.  Spring Data JPA provides more elaborate filtering abilities.  We can make use of [`ExampleMatcher`](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#query-by-example.matchers) to create more elaborate queries based on the needs at hand.

The `ExampleMatcher` can be customised based on the needs at hand.  We can replace the previous example with the following code.

```java
    final OfficeEntity office = new OfficeEntity();
    office.setCountry( "germany" );

    final ExampleMatcher matcher = ExampleMatcher.matching()
      .withIgnoreNullValues()
      .withIgnoreCase();

    final Example<OfficeEntity> example = Example.of( office, matcher );
    final List<OfficeEntity> offices = repository.findAll( example );
```

This is an overkill for our example but shows how we can fine tune the query to the required need.

## Tasks

- [X] Return one office by id
- [X] Filter by country
- [ ] Update individual office details
- [ ] Delete an office
