---
layout: default
title: Update one office
parent: Custom Queries
grand_parent: Spring Data
nav_order: 4
permalink: docs/data/custom-queries/update/
---

# Update one office
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Update one office

An existing office can be updated using a new method named `update()`, that takes an `Office` and updates the existing office if this exists.  The `update()` should return back the updated `Office`.

**What should we return if the office does not exists?**

We can rely on the [`Optional`](https://docs.oracle.com/en/java/javase/14/docs/api/java.base/java/util/Optional.html) to indicate whether the update happened successfully or not.  An [empty optional](https://docs.oracle.com/en/java/javase/14/docs/api/java.base/java/util/Optional.html#empty()) as we did in the `findOneByName()` method.

**Why do we need to return an office back?**

We need to indicate that the update was successful or not.  In the event an office does not exists, we may need to communicate that back to the caller.  We have three options

* Return an `Optional` together with the updated office.  This can be very helpful as we can continue working with the updated version of the office, should needs be.  The `Optional` type works very well with streams and makes it the ideal option.

* We can create an enum, such as `UpdateResult`, and return `UPDATED` when the update succeeds or `NOT_FOUND` when the office is not found.

* Alternatively we can return a `boolean`, where `true` indicates a successful update.  This is the least preferred option.

The [repository]( https://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/repository/CrudRepository.html) provides a [`save()`](https://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/repository/CrudRepository.html#save-S-) which returns the updated version of our entity.  We can take advantage of this and return the updated `Office`.

1. Add the new method to the interface

   Update file: `src/main/java/demo/boot/ContactUsService.java`

   ```java
   package demo.boot;

   import java.util.List;
   import java.util.Optional;

   public interface ContactUsService {

     List<Office> list();

     Optional<Office> findOneByName( final String name );

     List<Office> findAllInCountry( final String country );

     Optional<Office> update( final Office office );
   }
   ```

   Update file: `src/main/java/demo/boot/JpaContactUsService.java`

   ```java
   package demo.boot;

   import lombok.AllArgsConstructor;
   import org.springframework.stereotype.Service;

   import java.util.List;
   import java.util.Optional;
   import java.util.function.Function;
   import java.util.stream.Collectors;

   @Service
   @AllArgsConstructor
   public class JpaContactUsService implements ContactUsService {

     private final OfficesRepository repository;

     @Override
     public List<Office> list() { /* ... */ }

     @Override
     public Optional<Office> findOneByName( final String name ) { /* ... */ }

     @Override
     public List<Office> findAllInCountry( final String country ) { /* ... */ }

     @Override
     public Optional<Office> update( final Office office ) {
       return Optional.empty();
     }

     private List<Office> mapToOffices( final List<OfficeEntity> entities ) { /* ... */ }

     private Function<OfficeEntity, Office> mapToOffice() { /* ... */ }
   }
   ```

   {% include custom/note.html details="The update method simply returns an empty optional, just enough to compile." %}

1. Assert that an empty optional is return when the office does not exist.

   Update file: `src/test/java/demo/boot/JpaContactUsServiceTest.java`

   ```java
   package demo.boot;

   import org.junit.jupiter.api.DisplayName;
   import org.junit.jupiter.api.Test;

   import java.util.List;
   import java.util.Optional;

   import static org.junit.jupiter.api.Assertions.assertEquals;
   import static org.mockito.ArgumentMatchers.eq;
   import static org.mockito.Mockito.mock;
   import static org.mockito.Mockito.times;
   import static org.mockito.Mockito.verify;
   import static org.mockito.Mockito.verifyNoMoreInteractions;
   import static org.mockito.Mockito.when;

   @DisplayName( "JPA contact us service" )
   public class JpaContactUsServiceTest {

     @Test
     @DisplayName( "should return all offices returned by the repository" )
     public void shouldReturnOffices() { /* ... */ }

     @Test
     @DisplayName( "should query for the with a given id and return optional empty when the office is not found" )
     public void shouldReturnOptionEmpty() { /* ... */ }

     @Test
     @DisplayName( "should query for the with a given id and return the office returned by the repository" )
     public void shouldReturnOffice() { /* ... */ }

     @Test
     @DisplayName( "should return all offices in a given country" )
     public void shouldReturnOfficesInACountry() { /* ... */ }

     @Test
     @DisplayName( "should return empty optional if the office being saved does not exist" )
     public void shouldSaveNonExistentOffice() {
       final String name = "a1";
       final Office office = new Office( name, "a2", "a3", "a4" );

       final OfficesRepository repository = mock( OfficesRepository.class );
       when( repository.findById( eq( name ) ) ).thenReturn( Optional.empty() );

       final ContactUsService service = new JpaContactUsService( repository );
       final Optional<Office> saved = service.update( office );

       assertEquals( Optional.empty(), saved );

       verify( repository, times( 1 ) ).findById( name );
       verifyNoMoreInteractions( repository );
     }
   }
   ```

   Run the tests

   ```bash
   $ ./gradlew clean test

   ...

   JPA contact us service > should return empty optional if the office being saved does not exists FAILED
       org.mockito.exceptions.verification.WantedButNotInvoked at JpaContactUsServiceTest.java:118

   ...
   BUILD FAILED in 7s
   5 actionable tasks: 5 executed
   ```

   These should fail as despite the fact that the method returns what we expect (an empty optional), the service did not interact with the repository as expected.

1. Make the test pass

   Update file: `src/main/java/demo/boot/JpaContactUsService.java`

   ```java
   package demo.boot;

   import lombok.AllArgsConstructor;
   import org.springframework.stereotype.Service;

   import java.util.List;
   import java.util.Optional;
   import java.util.function.Function;
   import java.util.stream.Collectors;

   @Service
   @AllArgsConstructor
   public class JpaContactUsService implements ContactUsService {

     private final OfficesRepository repository;

     @Override
     public List<Office> list() { /* ... */ }

     @Override
     public Optional<Office> findOneByName( final String name ) { /* ... */ }

     @Override
     public List<Office> findAllInCountry( final String country ) { /* ... */ }

     @Override
     public Optional<Office> update( final Office office ) {
       return repository
         .findById( office.getName() )
         .map( mapToOffice() );
     }

     private List<Office> mapToOffices( final List<OfficeEntity> entities ) { /* ... */ }

     private Function<OfficeEntity, Office> mapToOffice() { /* ... */ }
   }
   ```

   {% include custom/note.html details="The above implementation is not the complete implementation and only enough for the test to pass." %}

   ```bash
   $ ./gradlew clean test

   ...

   BUILD SUCCESSFUL in 15s
   5 actionable tasks: 5 executed
   ```

1. Assert that the office is updated, and the updated office is returned

   Update file: `src/test/java/demo/boot/JpaContactUsServiceTest.java`

   ```java
   package demo.boot;

   import org.junit.jupiter.api.DisplayName;
   import org.junit.jupiter.api.Test;

   import java.util.List;
   import java.util.Optional;

   import static org.junit.jupiter.api.Assertions.assertEquals;
   import static org.mockito.ArgumentMatchers.eq;
   import static org.mockito.Mockito.mock;
   import static org.mockito.Mockito.times;
   import static org.mockito.Mockito.verify;
   import static org.mockito.Mockito.verifyNoMoreInteractions;
   import static org.mockito.Mockito.when;

   @DisplayName( "JPA contact us service" )
   public class JpaContactUsServiceTest {

     @Test
     @DisplayName( "should return all offices returned by the repository" )
     public void shouldReturnOffices() { /* ... */ }

     @Test
     @DisplayName( "should query for the with a given id and return optional empty when the office is not found" )
     public void shouldReturnOptionEmpty() { /* ... */ }

     @Test
     @DisplayName( "should query for the with a given id and return the office returned by the repository" )
     public void shouldReturnOffice() { /* ... */ }

     @Test
     @DisplayName( "should return all offices in a given country" )
     public void shouldReturnOfficesInACountry() { /* ... */ }

     @Test
     @DisplayName( "should return empty optional if the office being saved does not exist" )
     public void shouldSaveNonExistentOffice() { /* ... */ }

     @Test
     @DisplayName( "should update the office and return the updated version" )
     public void shouldSaveOffice() {
       final String name = "a1";
       final Office office = new Office( name, "a2", "a4", "a5" );
       final OfficeEntity existingEntity = new OfficeEntity( name, "o2", "o3", "o4", "o5", "o6" );
       final OfficeEntity updatedEntity = new OfficeEntity( name, "a2", "o3", "a4", "a5", "o6" );

       final OfficesRepository repository = mock( OfficesRepository.class );
       when( repository.findById( eq( name ) ) ).thenReturn( Optional.of( existingEntity ) );
       when( repository.save( eq( updatedEntity ) ) ).thenReturn( updatedEntity );

       final ContactUsService service = new JpaContactUsService( repository );
       final Optional<Office> saved = service.update( office );

       assertEquals( Optional.of( office ), saved );

       verify( repository, times( 1 ) ).findById( name );
       verify( repository, times( 1 ) ).save( updatedEntity );
       verifyNoMoreInteractions( repository );
     }
   }
   ```

   Run the tests.

   ```bash
   $ ./gradlew clean test

   ...

   BUILD FAILED in 8s
   5 actionable tasks: 5 executed
   ```

   The new test should fail.

1. Add the missing functionality.

   Update file: `src/main/java/demo/boot/JpaContactUsService.java`

   ```java
   package demo.boot;

   import lombok.AllArgsConstructor;
   import org.springframework.stereotype.Service;

   import java.util.List;
   import java.util.Optional;
   import java.util.function.Function;
   import java.util.stream.Collectors;

   @Service
   @AllArgsConstructor
   public class JpaContactUsService implements ContactUsService {

     private final OfficesRepository repository;

     @Override
     public List<Office> list() { /* ... */ }

     @Override
     public Optional<Office> findOneByName( final String name ) { /* ... */ }

     @Override
     public List<Office> findAllInCountry( final String country ) { /* ... */ }

     @Override
     public Optional<Office> update( final Office office ) {
       return repository
         .findById( office.getName() )
         .map( entity -> {
           entity.setAddress( office.getAddress() );
           entity.setPhone( office.getPhone() );
           entity.setEmail( office.getEmail() );
           return entity;
         } )
         .map( repository::save )
         .map( mapToOffice() );
     }

     private List<Office> mapToOffices( final List<OfficeEntity> entities ) { /* ... */ }

     private Function<OfficeEntity, Office> mapToOffice() { /* ... */ }
   }
   ```

   Run the tests.

   ```bash
   $ ./gradlew clean check
   ```

   All tests should pass.

   With the tests all passed, we can refactor the new method and move out the mapping of `Office` to `OfficeEntity` to a new method, as shown next.

   ```java
   package demo.boot;

   import lombok.AllArgsConstructor;
   import org.springframework.stereotype.Service;

   import java.util.List;
   import java.util.Optional;
   import java.util.function.Function;
   import java.util.stream.Collectors;

   @Service
   @AllArgsConstructor
   public class JpaContactUsService implements ContactUsService {

     private final OfficesRepository repository;

     @Override
     public List<Office> list() { /* ... */ }

     @Override
     public Optional<Office> findOneByName( final String name ) { /* ... */ }

     @Override
     public List<Office> findAllInCountry( final String country ) { /* ... */ }

     @Override
     public Optional<Office> update( final Office office ) {
       return repository
         .findById( office.getName() )
         .map( updateEntity( office ) )
         .map( repository::save )
         .map( mapToOffice() );
     }

     private List<Office> mapToOffices( final List<OfficeEntity> entities ) { /* ... */ }

     private Function<OfficeEntity, Office> mapToOffice() { /* ... */ }

     private Function<OfficeEntity, OfficeEntity> updateEntity( final Office office ) {
       return entity -> {
         entity.setAddress( office.getAddress() );
         entity.setPhone( office.getPhone() );
         entity.setEmail( office.getEmail() );
         return entity;
       };
     }
   }
   ```

   Run the tests again.

   ```bash
   $ ./gradlew clean check
   ```

   All tests should pass.

## Are concurrent updates a problem?

**YES**

Let's analyse the office update operation we have implemented so far.

```java
  @Override
  public Optional<Office> update( final Office office ) {
    return repository
      .findById( office.getName() )
      .map( updateEntity( office ) )
      .map( repository::save )
      .map( mapToOffice() );
  }
```

The above office update operation is subject to the _race condition_, or as also know to the _check then act_ concurrency problem.  We _check_ whether the office exists when we invoke the `findById()` method.  Then we _act_ by saving the updates by calling the `save()` method.

```java
    return repository
      .findById( office.getName() )   /* Check */
      .map( updateEntity( office ) )
      .map( repository::save )        /* Act */
      .map( mapToOffice() );
```

Each individual database update is atomic by nature, as the database ensures that.  In our current example, if two or more updates happen at the same time, these are simply applied in the order they are received by the database.  The _check then act_ problem does not affect us when updates happen concurrently, because irrespective of how may updates we make (the _act_ part), the `findById()` method (the _check_ part) will always find the office as this is still in the database.

In the [next section]({{ '/docs/data/custom-queries/delete/' | absolute_url }}), we will introduce the office delete operation, which simply deletes the office from the database.  This will introduce a new complexity as the update should not happen if the office does not exist.  We should not be able to update an office that was delete.  Consider the following scenario, where a delete operation happens while the office is being updated on two different threads (_T1_ and _T2_).

| Time | Update (_T1_)                        | Delete (_T2_)     |
| :--: | ------------------------------------ | ----------------- |
|   1  | `findById()` return office (_check_) |                   |
|   2  |                                      | `delete()` office |
|   3  | `save()` changes (_act_)             |                   |

The above table shows two things happening at the same time, on two separate threads (_T1_ and _T2_).  The sequence shown above will leave the data in an invalid state as the office will be saved back to the database (Time: 3) after this was deleted (Time: 2).  Irrespective of in which order the office update and delete operations happen, the office should be deleted.

* When the update happens before the delete, the update will succeed and then the item is deleted
* When the delete happens before the update, the update will not happen as the item does not exist

## How can we protect ourselves against issues related to concurrency?

Testing for concurrency is not an easy task.  First you need to identify where the problem lies, and then we need to be able to replicate it.  The latter is not an easy task as we need to get the problematic sequence right.

Our example is subject to the _check then act_ problem that may manifest itself once we introduce the delete operation.

{% include custom/note.html details="Such problems may lie hidden for a long time and only surface once in a while." %}

### Replicate problem

We need to write a test that is able to delete the office right after the `findById()` method (the _check_ part) is invoked but before the office is saved back to the database (the _act_ part), as shown next.

```java
    return repository
      .findById( office.getName() )   /* Check */
      .map( updateEntity( office ) )  /* <<< We need to delete the item at this point */
      .map( repository::save )        /* Act */
      .map( mapToOffice() );
```

The `updateEntity()` method returns a [`Function<OfficeEntity, OfficeEntity>`](https://docs.oracle.com/en/java/javase/14/docs/api/java.base/java/util/function/Function.html) that takes an `OfficeEntity` and returns it updated with the given `Office` values.

```java
  private Function<OfficeEntity, OfficeEntity> updateEntity( final Office office ) {
    return entity -> {
      entity.setAddress( office.getAddress() );
      entity.setPhone( office.getPhone() );
      entity.setEmail( office.getEmail() );
      return entity;
    };
  }
```

We can leverage this part of the code to simulate a delete operation.  We cannot override the `updateEntity()` method as this is private, but we can use a custom version of the `Office` that behaves as we need it to. Consider the following code fragment.

```java
    final Office office = new Office( "a", "b", "c", "d" ) {
      @Override
      public String getAddress() {
        /* Delete the office between the findById() and save() */
        deleteOfficeFromAnotherThread();
        return super.getAddress();
      }
    };
```

The above fragment, creates an inner anonymous class of `Office` and overrides the `getAddress()` method.  In the overridden `getAddress()` method, we invoke yet another method (`deleteOfficeFromAnotherThread()`) that deletes the office, then resumes by returning the address.

The `getAddress()` method is invoked from within the `updateEntity()` method, after the `findById()` method is invoked, but before the `save()` method is invoked. The `deleteOfficeFromAnotherThread()` needs to delete the office using a different thread, as shown next.

{% include custom/proceed_with_caution.html details="The following code fragment is supressing the <a href='https://docs.oracle.com/en/java/javase/14/docs/api/java.base/java/lang/InterruptedException.html'><code>InterruptedException</code></a>, as this does not affect our tests.  <strong>Do not suppress or ignore <code>InterruptedException</code> unless you know what you are doing</strong>." %}

```java
  private void deleteOfficeFromAnotherThread() {
    try {
      final Thread delete = new Thread( () -> repository.delete( COLOGNE ), "DELETE" );
      delete.start();
      delete.join();
    } catch ( final InterruptedException e ) { /* Ignore for the test */ }
  }
```

It is important to delete the office using a different thread, as otherwise Hibernate will fail for a different reason and the test will not be simulating a real-life situation.

The following image captures the whole flow.

![Delete office while update]({{ '/assets/images/Office-Delete-While-Update.png' | absolute_url }})

Following is the complete test that replicates this problem.

1. Replicate problem

   Create file: `src/test-integration/java/demo/boot/JpaContactUsServiceDeleteWhileUpdateTest.java`

   {% include custom/note.html details="This is an <strong>integration test</strong> as we need to make use of Spring to solve this problem." %}

   ```java
   package demo.boot;

   import org.junit.jupiter.api.AfterEach;
   import org.junit.jupiter.api.BeforeEach;
   import org.junit.jupiter.api.DisplayName;
   import org.junit.jupiter.api.Test;
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.boot.test.context.SpringBootTest;
   import org.springframework.boot.test.context.SpringBootTest.WebEnvironment;

   import java.util.Optional;

   import static org.assertj.core.api.Assertions.assertThat;

   @DisplayName( "JPA contact us service (delete while updating)" )
   @SpringBootTest( webEnvironment = WebEnvironment.NONE )
   public class JpaContactUsServiceDeleteWhileUpdateTest {

     @Autowired
     private OfficesRepository repository;

     @Autowired
     private ContactUsService service;

     private static final OfficeEntity ENTITY = new OfficeEntity(
       "Test Office",
       "Test Address",
       "Test Country",
       "Test Phone",
       "Test Email",
       "Test Webpage"
     );

     @BeforeEach
     void setUp() {
       repository.save( ENTITY );
     }

     @AfterEach
     void tearDown() {
       repository.delete( ENTITY );
     }

     @Test
     @DisplayName( "should not save the office after this is deleted" )
     public void shouldHandleConcurrentUpdates() {
       final Office office = new Office( ENTITY.getName(), "b", "c", "d" ) {
         @Override
         public String getAddress() {
           /* Delete the office between the findById() and save() */
           deleteOfficeFromAnotherThread();
           return super.getAddress();
         }
       };

       service.update( office );

       /* The office should not be in the database, as this was deleted, and the update should not succeed */
       final Optional<OfficeEntity> entity = repository.findById( ENTITY.getName() );
       assertThat( entity.isEmpty() ).isTrue();
     }

     private void deleteOfficeFromAnotherThread() {
       try {
         final Thread delete = new Thread( () -> repository.delete( ENTITY ), "DELETE" );
         delete.start();
         delete.join();
       } catch ( final InterruptedException e ) { /* Ignore for the test */ }
     }
   }
   ```

   Parts of this test were already discussed before while other parts were not.  Let's break this test down.

   1. This test requires the application to be setup to work without the web part.

      ```java
      @SpringBootTest( webEnvironment = WebEnvironment.NONE )
      ```

   1. In this particular test we cannot take advantage of test transaction rollback and we need to manually delete the office that was added during the test.  Furthermore, to minimise the interference with other tests, we are not using a known office.

      ```java
        private static final OfficeEntity ENTITY = new OfficeEntity(
          "Test Office",
          "Test Address",
          "Test Country",
          "Test Phone",
          "Test Email",
          "Test Webpage"
        );

        @BeforeEach
        public void setUp() {
          repository.save( ENTITY );
        }

        @AfterEach
        public void tearDown() {
          repository.delete( ENTITY );
        }
      ```

   1. The test also verifies that the office is deleted, and it is not in the table at the end of the test.

      ```java
          /* The office should not be in the database, as this was deleted, and the update should not succeed */
          final Optional<OfficeEntity> entity = repository.findById( COLOGNE.getName() );
          assertThat( entity.isEmpty() ).isTrue();
      ```

1. Run the test

   ```bash
   $ ./gradlew clean integrationTest

   ...
   JPA contact us service (delete while updating) > should not save the office after this is deleted FAILED
       org.opentest4j.AssertionFailedError at JpaContactUsServiceDeleteWhileUpdateTest.java:55
   ...

    BUILD FAILED in 13s
    6 actionable tasks: 6 executed
   ```

   The test will fail as we have no protection in place to safeguard against this race condition.

   View the test report.

   ```bash
   $ open build/reports/tests/integrationTest/index.html
   ```

   ![Office-Delete-While-Update-Test-Fail.png]({{ '/assets/images/Office-Delete-While-Update-Test-Fail.png' | absolute_url }})

### Transactions

We need to make the two operations, `findById()` and `save()` methods, atomic.  We can use the [`@Transactional`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/transaction/annotation/Transactional.html) annotation, as shown next.

1. Use transactions

   Update file: `src/main/java/demo/boot/JpaContactUsService.java`

   ```java
   package demo.boot;

   import lombok.AllArgsConstructor;
   import org.springframework.stereotype.Service;
   import org.springframework.transaction.annotation.Transactional;

   import java.util.List;
   import java.util.Optional;
   import java.util.function.Function;
   import java.util.stream.Collectors;

   @Service
   @AllArgsConstructor
   public class JpaContactUsService implements ContactUsService {

     private final OfficesRepository repository;

     @Override
     public List<Office> list() { /* ... */ }

     @Override
     public Optional<Office> findOneByName( final String name ) { /* ... */ }

     @Override
     public List<Office> findAllInCountry( final String country ) { /* ... */ }

     @Override
     @Transactional
     public Optional<Office> update( final Office office ) {
       return repository
         .findById( office.getName() )
         .map( updateEntity( office ) )
         .map( repository::save )
         .map( mapToOffice() );
     }

     private List<Office> mapToOffices( final List<OfficeEntity> entities ) { /* ... */ }

     private Function<OfficeEntity, Office> mapToOffice() { /* ... */ }

     private Function<OfficeEntity, OfficeEntity> updateEntity( final Office office ) { /* ... */ }
   }
   ```

1. Run the tests

   ```bash
   $ ./gradlew clean integrationTest
   ```

   The test will still fail, for a new reason.

   ```bash
   $ open "build/reports/tests/integrationTest/classes/demo.boot.JpaContactUsServiceDeleteWhileUpdateTest.html"
   ```

   The `update()` method, within the `JpaContactUsService` class, is now failing with a [`ObjectOptimisticLockingFailureException`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/orm/ObjectOptimisticLockingFailureException.html).

   ```bash
   org.springframework.orm.ObjectOptimisticLockingFailureException: Object of class [demo.boot.OfficeEntity] with identifier [ThoughtWorks Test Office]: optimistic locking failed; nested exception is org.hibernate.StaleObjectStateException: Row was updated or deleted by another transaction (or unsaved-value mapping was incorrect) : [demo.boot.OfficeEntity#ThoughtWorks Test Office]
     at org.springframework.orm.jpa.vendor.HibernateJpaDialect.convertHibernateAccessException(HibernateJpaDialect.java:337)
     at org.springframework.orm.jpa.vendor.HibernateJpaDialect.translateExceptionIfPossible(HibernateJpaDialect.java:255)
   ...
   ```

   This is an improvement as now the concurrency problem is not unnoticed.

While this solves our race condition, we can do better as shown [next](#retry-on-error).

### Retry on error

We can do better than simply failing.  In our case, we can retry the office update operation when we encounter this error.  [Spring Batch](https://spring.io/projects/spring-batch) provides the retry functionality.

1. Add the [`spring-boot-starter-batch`](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-batch) dependency

   Update file: `build.gradle`

   ```groovy
   dependencies {
     implementation 'org.springframework.boot:spring-boot-starter-batch'
   }
   ```

   Following is a complete list of the dependencies used in the project.

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
     implementation 'org.springframework.boot:spring-boot-starter-batch'
     runtimeOnly 'org.flywaydb:flyway-core'
     runtimeOnly 'org.postgresql:postgresql'

     /* Prometheus */
     runtimeOnly 'io.micrometer:micrometer-registry-prometheus'

     /* Spring/OpenApi */
     implementation 'org.springdoc:springdoc-openapi-ui:1.3.9'
   }
   ```

1. Enable retry

   Update file: `src/main/java/demo/boot/ContactUsApplication.java`

   ```java
   package demo.boot;

   import org.springframework.boot.SpringApplication;
   import org.springframework.boot.autoconfigure.SpringBootApplication;
   import org.springframework.retry.annotation.EnableRetry;

   @EnableRetry
   @SpringBootApplication
   public class ContactUsApplication {
     public static void main(String[] args) { /* ... */ }
   }
   ```

   The [`@EnableRetry`](https://docs.spring.io/spring-retry/docs/api/current/org/springframework/retry/annotation/EnableRetry.html) annotation enables the use of the [`@Retryable`](https://docs.spring.io/spring-retry/docs/api/current/org/springframework/retry/annotation/Retryable.html) annotation.

   Spring will scan our services and will create a [proxy](https://docs.oracle.com/javase/8/docs/technotes/guides/reflection/proxy.html) for any service which contains methods annotated with the `@Retryable` annotation, as shown next.

   ![Proxy.png]({{ '/assets/images/Proxy.png' | absolute_url }})

   Using the proxy, Spring will intercept calls to our method (that is annotated with the `@Retryable` annotation) and will wrap them in a try/catch, similar to the image shown above.

1. Annotation the method with the `@Retryable` annotation

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
   public class JpaContactUsService implements ContactUsService {

     private final OfficesRepository repository;

     @Override
     public List<Office> list() { /* ... */ }

     @Override
     public Optional<Office> findOneByName( final String name ) { /* ... */ }

     @Override
     public List<Office> findAllInCountry( final String country ) { /* ... */ }

     @Override
     @Transactional
     @Retryable( ObjectOptimisticLockingFailureException.class )
     public Optional<Office> update( final Office office ) {
       return repository
         .findById( office.getName() )
         .map( updateEntity( office ) )
         .map( repository::save )
         .map( mapToOffice() );
     }

     private List<Office> mapToOffices( final List<OfficeEntity> entities ) { /* ... */ }

     private Function<OfficeEntity, Office> mapToOffice() { /* ... */ }

     private Function<OfficeEntity, OfficeEntity> updateEntity( final Office office ) { /* ... */ }
   }
   ```

1. Run the tests

   ```bash
   $ ./gradlew clean check

   ...

   BUILD SUCCESSFUL in 19s
   7 actionable tasks: 7 executed
   ```

   The test should now pass.

## Objectives status

- [X] Return one office by id
- [X] Return all offices in country
- [X] Update one office
- [ ] Delete an office
