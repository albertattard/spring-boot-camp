---
layout: default
title: Custom Queries
parent: Spring Data
nav_order: 99
permalink: docs/data/custom-queries/
---

# Custom Query
{: .no_toc }

We are able to return all office using the provided [`findAll()`](https://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/repository/CrudRepository.html#findAll--) method.  In this section we will introduce custom queries and take advantage of [Spring Repositories](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#jpa.repositories).

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Scenario

Customers using our API came back with feedback.  They would like to filter offices by country.  For example, given the country, `"germany"`, our repository should return all offices that belong to this country, case-insensitive.

## Filter by a property (JPA query methods)

[Spring Data JPA](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/) provides [query method](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#jpa.query-methods) that we can take advantage from.

We can acheive filtering by simply adding a new method to our repository as shown next.

```java
List<OfficeEntity> findAllByCountryIgnoreCase( final String country );
```

That's it!!

No need to add code, as Spring Data JPA will take care of the rest.  The method name is not random and Spring is using the method name to determine what needs to be done.  Following is a breakdown of the method name (`findAllByCountryIgnoreCase`)

1. `findAll` (or just `find`): indicates that this method will query the table.  We can prefix the method name with: `read…By`, `query…By`, and `get…By` instead.
1. `By`: indicates that a set of filters will follow
1. `Country`: the property name (not necessarily the column name) by which we will filter
1. `IgnoreCase`: indicate that the filter should be case-insensitive

Note that while we are not adding actual code, the method name has logic bound to it and thus it needs to be tested like any other code.

1. Create a repository (integration) test

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

      These two annotations prepares our application for tests.  Usually we can do with just the [`@DataJpaTest`](https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/test/autoconfigure/orm/jpa/DataJpaTest.html) (_super_) annotation, but given that we do not want to swap the actual database with an embedded one (such as [H2](https://www.h2database.com/)), we need to fine tune our test setup.  Without the [`@AutoConfigureTestDatabase`](https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/test/autoconfigure/jdbc/AutoConfigureTestDatabase.html) annotation will test will fail to run due to the following exception.

      ```bash
      Caused by: java.lang.IllegalStateException: Failed to replace DataSource with an embedded database for tests. If you want an embedded database please put a supported one on the classpath or tune the replace attribute of @AutoConfigureTestDatabase.
          at org.springframework.util.Assert.state(Assert.java:73)
          at org.springframework.boot.test.autoconfigure.jdbc.TestDatabaseAutoConfiguration$EmbeddedDataSourceFactory.getEmbeddedDatabase(TestDatabaseAutoConfiguration.java:179)
          at org.springframework.boot.test.autoconfigure.jdbc.TestDatabaseAutoConfiguration$EmbeddedDataSourceFactoryBean.afterPropertiesSet(TestDatabaseAutoConfiguration.java:145)
          at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.invokeInitMethods(AbstractAutowireCapableBeanFactory.java:1855)
          at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.initializeBean(AbstractAutowireCapableBeanFactory.java:1792)
          ... 115 more
      ```

      Using the `@AutoConfigureTestDatabase` annotation we can customise the test setup and use the original PostgreSQL database.

      One of the great advantages of `@DataJpaTest` is that it includes [`@Transactional`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/transaction/annotation/Transactional.html) annotation.  When used within tests, the `@Transactional` rollbacks at the end of each test, and thus any changes made will be ignored.  For example if we delete all from a table, these will be only deleted for the tests only and these changes will be rolled back after each test.

   1. Entity Manager

      ```java
        @PersistenceContext
        private EntityManager entityManager;
      ```

      An [`EntityManager`](https://docs.oracle.com/javaee/7/api/javax/persistence/EntityManager.html) is responsible for the lifecycle of entities.  Our application is using an instance of `EntityManager` to read data from the `offices` table.  It is used within this test to be able to empty the table and add new offices to prepare the test data for our tests.

   1. Setup the data

      ```java
        @BeforeEach
        public void before() {
          entityManager.createQuery( "DELETE FROM OfficeEntity" ).executeUpdate();
          entityManager.persist( COLOGNE );
          entityManager.persist( MANCHESTER );
        }
      ```

      Empty the table first and adds two offices.  This makes sure that the table will only have two offices for the duration of the tests.  Note that these changes are rolled back at the end of the test.

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

      This test makes sure that our method name, `findAllByCountryIgnoreCase`, works as expected.

   Unfortunately, this is a case where the number of lines of test code is far too many when compare to the code being tested.  In this example we have a `1:60` relation.  With that said, this ensures that our code is behaving as it is expected.

## More

```java
package demo.boot;

import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

import java.util.List;

@Repository
public interface OfficesRepository extends JpaRepository<OfficeEntity, String> {

  List<OfficeEntity> findAllByCountryIgnoreCase( String country );
}
```


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

```java
package demo.boot;

import java.util.List;

public interface ContactUsService {

  List<Office> list();

  List<Office> listInCountry( String country );
}
```

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

```bash
$ ./gradlew clean test

...
BUILD FAILED in 9s
5 actionable tasks: 5 executed
```

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

```bash
$ ./gradlew clean test

...
BUILD SUCCESSFUL in 5s
5 actionable tasks: 5 executed
```

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

  private List<Office> mapToOffices( List<OfficeEntity> entities ) {
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
