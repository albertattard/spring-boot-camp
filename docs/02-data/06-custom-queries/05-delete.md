---
layout: default
title: Delete an office
parent: Custom Queries
grand_parent: Spring Data
nav_order: 5
permalink: docs/data/custom-queries/delete/
---

# Delete an office
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Delete an office

An existing office can be deleted using a new method, named `delete()`, that takes the office name and deletes it if this exists.  The `delete()` should return back the `Office` that was deleted.

**What should we return if the office does not exists?**

We can rely on the [`Optional`](https://docs.oracle.com/en/java/javase/14/docs/api/java.base/java/util/Optional.html) to indicate whether the delete happened successfully or not.  An [empty optional](https://docs.oracle.com/en/java/javase/14/docs/api/java.base/java/util/Optional.html#empty()) as we did in the `findOneByName()` method.

**Why do we need to return an office back?**

We need to indicate whether the delete operation was successful or not.  In the event an office does not exists, we may need to communicate that back to the caller.  We have three options

* Return an `Optional` together with the deleted office.  This can be very helpful as we can continue working with the deleted version of the office, should needs be.  The `Optional` type works very well with streams and makes it the ideal option.

* We can create an enum, such as `DeleteResult`, and return `DELETED` when the delete operation succeeds or `NOT_FOUND` when the office is not found.

* Alternatively, we can return a `boolean`, where `true` indicates a successful delete and `false` otherwise.  This is the least preferred option.

The [repository]( https://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/repository/CrudRepository.html) provides a [`delete()`](https://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/repository/CrudRepository.html#delete-T-) method which returns `void`.  Therefore, we cannot rely on this method to determine whether the office exists or not.  We have to first retrieve the office using the [`findById()`](https://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/repository/CrudRepository.html#findById-ID-) method, and then if the office is found we delete it and return the office found before deletion.

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

     Optional<Office> delete( final String name );
   }
   ```

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
     public Optional<Office> findOneByName( final String office ) { /* ... */ }

     @Override
     public List<Office> findAllInCountry( final String country ) { /* ... */ }

     @Override
     @Transactional
     @Retryable( ObjectOptimisticLockingFailureException.class )
     public Optional<Office> update( final Office office ) { /* ... */ }

     @Override
     public Optional<Office> delete( final String name ) {
       return Optional.empty();
     }

     private List<Office> mapToOffices( final List<OfficeEntity> entities ) { /* ... */ }

     private Function<OfficeEntity, Office> mapToOffice() { /* ... */ }

     private Function<OfficeEntity, OfficeEntity> updateEntity( final Office office ) { /* ... */ }
   }
   ```

   {% include custom/note.html details="The delete method simply returns an empty optional, just enough to compile." %}

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
     public void shouldSaveNonExistentOffice() { /* ... */ }

     @Test
     @DisplayName( "should update the office and return the updated version" )
     public void shouldSaveOffice() { /* ... */ }

     @Test
     @DisplayName( "should return empty optional if the office being deleted does not exist" )
     public void shouldDeleteNonExistentOffice() {
       final String name = "a1";

       final OfficesRepository repository = mock( OfficesRepository.class );
       when( repository.findById( eq( name ) ) ).thenReturn( Optional.empty() );

       final ContactUsService service = new JpaContactUsService( repository );
       final Optional<Office> deleted = service.delete( name );

       assertEquals( Optional.empty(), deleted );

       verify( repository, times( 1 ) ).findById( name );
       verifyNoMoreInteractions( repository );
     }
   }
   ```

   Run the tests

   ```bash
   $ ./gradlew clean test

   ...
   JPA contact us service > should return empty optional if the office being deleted does not exist FAILED
       org.mockito.exceptions.verification.WantedButNotInvoked at JpaContactUsServiceTest.java:156
   ```

   These should fail as despite the fact that the method returns what we expect (an empty optional).  The service did not interact with the repository as expected.

1. Make the test pass

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
     public Optional<Office> findOneByName( final String office ) { /* ... */ }

     @Override
     public List<Office> findAllInCountry( final String country ) { /* ... */ }

     @Override
     @Transactional
     @Retryable( ObjectOptimisticLockingFailureException.class )
     public Optional<Office> update( final Office office ) { /* ... */ }

     @Override
     public Optional<Office> delete( final String name ) {
       return repository
         .findById( name )
         .map( mapToOffice() );
     }

     private List<Office> mapToOffices( final List<OfficeEntity> entities ) { /* ... */ }

     private Function<OfficeEntity, Office> mapToOffice() { /* ... */ }

     private Function<OfficeEntity, OfficeEntity> updateEntity( final Office office ) { /* ... */ }
   }
   ```

   {% include custom/note.html details="The above implementation is not the complete implementation and only enough for the test to pass." %}

   ```bash
   $ ./gradlew clean test

   ...
   BUILD SUCCESSFUL in 5s
   5 actionable tasks: 5 executed
   ```

1. Assert that the office is deleted, and the deleted office is returned

   Update file: `src/test/java/demo/boot/JpaContactUsServiceTest.java`

   ```java
   package demo.boot;

   import org.junit.jupiter.api.DisplayName;
   import org.junit.jupiter.api.Test;

   import java.util.List;
   import java.util.Optional;

   import static org.junit.jupiter.api.Assertions.assertEquals;
   import static org.mockito.ArgumentMatchers.eq;
   import static org.mockito.Mockito.doNothing;
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
     public void shouldSaveOffice() { /* ... */ }

     @Test
     @DisplayName( "should return empty optional if the office being deleted does not exist" )
     public void shouldDeleteNonExistentOffice() { /* ... */ }

     @Test
     @DisplayName( "should delete the office and return the deleted office" )
     public void shouldDeleteOffice() {
       final String name = "a1";
       final Office office = new Office( name, "a2", "a4", "a5" );
       final OfficeEntity entity = new OfficeEntity( name, "a2", "a3", "a4", "a5", "a6" );

       final OfficesRepository repository = mock( OfficesRepository.class );
       when( repository.findById( eq( name ) ) ).thenReturn( Optional.of( entity ) );
       doNothing().when( repository ).delete( eq( entity ) );

       final ContactUsService service = new JpaContactUsService( repository );
       final Optional<Office> deleted = service.delete( name );

       assertEquals( Optional.of( office ), deleted );

       verify( repository, times( 1 ) ).findById( name );
       verify( repository, times( 1 ) ).delete( entity );
       verifyNoMoreInteractions( repository );
     }
   }
   ```

   Run the tests.

   ```bash
   $ ./gradlew clean test

   ...

   JPA contact us service > should delete the office and return the deleted office FAILED
       org.mockito.exceptions.verification.WantedButNotInvoked at JpaContactUsServiceTest.java:178

   ...

   BUILD FAILED in 7s
   5 actionable tasks: 5 executed
   ```

   The new test should fail.

1. Add the missing functionality.

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
     public Optional<Office> findOneByName( final String office ) { /* ... */ }

     @Override
     public List<Office> findAllInCountry( final String country ) { /* ... */ }

     @Override
     @Transactional
     @Retryable( ObjectOptimisticLockingFailureException.class )
     public Optional<Office> update( final Office office ) { /* ... */ }

     @Override
     public Optional<Office> delete( final String name ) {
       return repository
         .findById( name )
         .map( entity -> {
           repository.delete( entity );
           return entity;
         } )
         .map( mapToOffice() );
     }

     private List<Office> mapToOffices( final List<OfficeEntity> entities ) { /* ... */ }

     private Function<OfficeEntity, Office> mapToOffice() { /* ... */ }

     private Function<OfficeEntity, OfficeEntity> updateEntity( final Office office ) { /* ... */ }
   }
   ```

   Run the tests.

   ```bash
   $ ./gradlew clean test

   ...

   BUILD SUCCESSFUL in 6s
   5 actionable tasks: 5 executed
   ```

   The `delete()` method can be refactored and the delete operation moved out of it, as shown next.

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
     public List<Office> list()  { /* ... */ }

     @Override
     public Optional<Office> findOneByName( final String office )  { /* ... */ }

     @Override
     public List<Office> findAllInCountry( final String country )  { /* ... */ }

     @Override
     @Transactional
     @Retryable( ObjectOptimisticLockingFailureException.class )
     public Optional<Office> update( final Office office )  { /* ... */ }

     @Override
     public Optional<Office> delete( final String name ) {
       return repository
         .findById( name )
         .map( deleteEntity() )
         .map( mapToOffice() );
     }

     private Function<OfficeEntity, OfficeEntity> deleteEntity() {
       return entity -> {
         repository.delete( entity );
         return entity;
       };
     }

     private List<Office> mapToOffices( final List<OfficeEntity> entities )  { /* ... */ }

     private Function<OfficeEntity, Office> mapToOffice()  { /* ... */ }

     private Function<OfficeEntity, OfficeEntity> updateEntity( final Office office ) { /* ... */ }
   }
   ```

   This makes the `delete()` method easier to read.

## Are concurrent deletes a problem?

**YES**

Let's analyse the office delete operation we have implemented so far.

```java
  @Override
  public Optional<Office> delete( final String name ) {
    return repository
      .findById( name )
      .map( deleteEntity() )
      .map( mapToOffice() );
  }
```

The above office delete operation is subject to the _race condition_, or as also know to the _check then act_ concurrency problem.  We _check_ whether the office exists when we invoke the `findById()` method.  Then we _act_ by deleting it by calling the `delete()` method (from within the `deleteEntity()` method).

```java
    return repository
      .findById( name )      /* Check */
      .map( deleteEntity() ) /* Act */
      .map( mapToOffice() );
```

Each individual database delete is atomic by nature, as the database ensures that.  In our current example, if two or more deletes happen at the same time, these are simply applied in the order they are received by the database.  The _check then act_ problem does not affect the result in the database state, but our method output.  The database cannot delete the same office twice.  It will simply delete the office the first time it receives the command, while any consecutive deletes will have no effect.

Our contract for this method says that the method should return the deleted office if the office exists, otherwise optional empty.  The caller of the delete operation may act upon the response received and trigger further actions.  While this will not have any impact at the data level, this may have unexpected effects on other things that depends on this atomic action.

Consider the following scenario, where two delete operations happen simultaneously on two different threads (_T1_ and _T2_).

| Time | Delete (_T1_)                        | Delete (_T2_)                        |
| :--: | ------------------------------------ | ------------------------------------ |
|   1  | `findById()` return office (_check_) |                                      |
|   2  |                                      | `findById()` return office (_check_) |
|   3  |                                      | `delete()` office (_act_)            |
|   4  | `delete()` office (_act_)            |                                      |

Both actions will return a successful delete, where only one of them should succeed.  At a first glance this may not be a problem, as in either case, the office is deleted from the database.

As mentioned before, both calls may do further action based on the result of the delete operation.  For example, an email is sent to a third party saying the that office was deleted only when the office is actually deleted.  In this example, the receiver will receive two identical emails, which may be confusing.  Say that a payment needs to be affected once an office is deleted.  In this case the payment may be affected more than once.

## How can we protect ourselves against issues related to concurrency?

Testing for concurrency is not an easy task.  First you need to identify where the problem lies, and then we need to be able to replicate it.  The latter is not an easy task as we need to get the problematic sequence right.

Our example is subject to the _check then act_ problem.

{% include custom/note.html details="Such problems may lie hidden for a long time and only surface once in a while." %}

### Replicate problem

We need to write a test that is able to delete the office right after the `findById()` method (the _check_ part) is invoked but before the office is deleted from to the database (the _act_ part), as shown next.

```java
    return repository
      .findById( name )      /* Check */
      .map( deleteEntity() ) /* Act */
      .map( mapToOffice() );
```

Different from [the update test example we saw in the previous section]({{ '/docs/data/custom-queries/update/#replicate-problem' | absolute_url }}), we don't have a method we can take advantage from to coordinate the delete operations.

We have to rely on another approach.  Instead, of coordinating two threads to reproduce the problem, we can bombard the delete operation using multiple threads and assert that only one of them succeeds.  At a first glance this seems easy and should do the trick.  Unfortunately, this is a matter of chance and we may be able to run the test and the succeed when these should fail.  Using such approach, we are subject to chance.

Say that we are running the tests on a machine that has one core/CPU and each thread that is used to bombard the delete operation is executed in sequence.  This setup will make our test ineffective as each thread is only executed after the previous one finishes.

Alternatively, we can introduce a hook method, which we can use to coordinate the flow.  The downside of using a hook method is that, we are modifying the code to accommodate our tests.  The hook method is only required by this particular test and nothing else.

We will use the first approach (bombarding the delete operation) which is less invasive on the code.

1. Create new integration test

   Create file: `src/test-integration/java/demo/boot/JpaContactUsServiceMultiDeleteTest.java`

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
   import java.util.concurrent.atomic.AtomicInteger;

   import static org.assertj.core.api.Assertions.assertThat;

   @DisplayName( "JPA contact us service (multi-delete)" )
   @SpringBootTest( webEnvironment = WebEnvironment.NONE )
   public class JpaContactUsServiceMultiDeleteTest {

     @Autowired
     private OfficesRepository repository;

     @Autowired
     private ContactUsService service;

     private static final OfficeEntity ENTITY = new OfficeEntity(
       "ThoughtWorks Test Office",
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

     @Test
     @DisplayName( "should delete the office only once" )
     public void shouldHandleConcurrentDeletes() {
       final AtomicInteger deletedCount = new AtomicInteger();

       final Optional<Office> deleted = service.delete( ENTITY.getName() );
       deleted.ifPresent( e -> deletedCount.incrementAndGet() );

       assertThat( deletedCount.intValue() ).isEqualTo( 1 );
     }
   }
   ```

   {% include custom/note.html details="The delete test shown above only deletes the office once using one thread." %}

   Run the test.

   ```bash
   $ ./gradlew clean integrationTest

   ...
   BUILD SUCCESSFUL in 14s
   6 actionable tasks: 6 executed
   ```

   The test should pass as so far, we have not yet tried to perform concurrent deletes.

1. Simulate concurrent deletes (replicate the problem)

   Update file: `src/test-integration/java/demo/boot/JpaContactUsServiceMultiDeleteTest.java`

   ```java
   package demo.boot;

   import org.junit.jupiter.api.AfterEach;
   import org.junit.jupiter.api.BeforeEach;
   import org.junit.jupiter.api.DisplayName;
   import org.junit.jupiter.api.Test;
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.boot.test.context.SpringBootTest;
   import org.springframework.boot.test.context.SpringBootTest.WebEnvironment;

   import java.util.ArrayList;
   import java.util.List;
   import java.util.Optional;
   import java.util.concurrent.CyclicBarrier;
   import java.util.concurrent.TimeUnit;
   import java.util.concurrent.atomic.AtomicInteger;

   import static org.assertj.core.api.Assertions.assertThat;

   @DisplayName( "JPA contact us service (multi-delete)" )
   @SpringBootTest( webEnvironment = WebEnvironment.NONE )
   public class JpaContactUsServiceMultiDeleteTest {

     @Autowired
     private OfficesRepository repository;

     @Autowired
     private ContactUsService service;

     private static final OfficeEntity ENTITY = new OfficeEntity( /* ... */ );

     @BeforeEach
     public void setUp() { /* ... */ }

     @AfterEach
     public void tearDown() { /* ... */ }

     @Test
     @DisplayName( "should delete the office only once" )
     public void shouldHandleConcurrentDeletes() throws Exception {
       /* Will use 12 threads to bombard the delete operation */
       final int numberOfThreads = 12;

       /* Count the number of successful deletes (which should be 0) and any errors (which should be none) */
       final AtomicInteger deletedCount = new AtomicInteger();
       final AtomicInteger errorCount = new AtomicInteger();

       /* Coordinate the threads to hit the delete operation all at 'the same time' */
       final CyclicBarrier barrier = new CyclicBarrier( numberOfThreads );

       /* Create the task which will trigger the delete operation in a coordinated fashion (coordinated through the CyclicBarrier) */
       final Runnable deleteTask = () -> {
         try {
           /* Wait for all threads to reach this point */
           barrier.await( 10, TimeUnit.SECONDS );
           final Optional<Office> deleted = service.delete( ENTITY.getName() );
           deleted.ifPresent( e -> deletedCount.incrementAndGet() );
         } catch ( final Exception e ) {
           errorCount.incrementAndGet();
         }
       };

       /* Invoke the delete operation from multiple threads at 'the same time' (coordinated through the CyclicBarrier) */
       final List<Thread> threads = new ArrayList<>( numberOfThreads );
       for ( int i = 1; i <= numberOfThreads; i++ ) {
         final Thread thread = new Thread( deleteTask, String.format( "DELETE-%d", i ) );
         thread.start();
         threads.add( thread );
       }

       /* Wait for all threads to finish */
       for ( final Thread thread : threads ) {
         thread.join( TimeUnit.SECONDS.toMillis( 5 ) );
       }

       /* Make sure that all threads ran */
       assertThat( barrier.getNumberWaiting() ).isEqualTo( 0 );

       /* Make sure that only one delete operation succeeded */
       assertThat( deletedCount.intValue() ).isEqualTo( 1 );

       /* Make sure that no errors occurred  */
       assertThat( errorCount.intValue() ).isEqualTo( 0 );
     }
   }
   ```

   Run the test.

   ```bash
   $ ./gradlew clean integrationTest

   ...
   BUILD FAILED in 14s
   6 actionable tasks: 6 executed
   ```

   As expected, the test now fails.

   ```bash
   $ open "build/reports/tests/integrationTest/classes/demo.boot.JpaContactUsServiceMultiDeleteTest.html"
   ```

   ![Office-Multiple-Delete-Test-Fail-1.png]({{ '/assets/images/Office-Multiple-Delete-Test-Fail-1.png' | absolute_url }})

   Different from what is expected by the test, more than one thread was able to delete the same office, as shown in the above image.

1. (_Optional_) Refactor the test

   {% include custom/note.html details="The test works well, but the test method is doing too many things and can be simplified.  While optional, this refactoring is recommended." %}

   Update file: `src/test-integration/java/demo/boot/JpaContactUsServiceMultiDeleteTest.java`

   ```java
   package demo.boot;

   import org.junit.jupiter.api.AfterEach;
   import org.junit.jupiter.api.BeforeEach;
   import org.junit.jupiter.api.DisplayName;
   import org.junit.jupiter.api.Test;
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.boot.test.context.SpringBootTest;
   import org.springframework.boot.test.context.SpringBootTest.WebEnvironment;

   import java.util.ArrayList;
   import java.util.List;
   import java.util.Optional;
   import java.util.concurrent.CyclicBarrier;
   import java.util.concurrent.TimeUnit;
   import java.util.concurrent.atomic.AtomicInteger;

   import static org.assertj.core.api.Assertions.assertThat;

   @DisplayName( "JPA contact us service (multi-delete)" )
   @SpringBootTest( webEnvironment = WebEnvironment.NONE )
   public class JpaContactUsServiceMultiDeleteTest {

     @Autowired
     private OfficesRepository repository;

     @Autowired
     private ContactUsService service;

     private static final OfficeEntity ENTITY = new OfficeEntity( /* ... */ );

     @BeforeEach
     public void setUp() { /* ... */ }

     @AfterEach
     public void tearDown() { /* ... */ }

     @Test
     @DisplayName( "should delete the office only once" )
     public void shouldHandleConcurrentDeletes() throws Exception {
       /* Will use 12 threads to bombard the delete operation */
       TestHelper
         .withThreads( 12 )
         .runTestAgainstService( service )
         .assertAllWentAsExpected()
       ;
     }

     private static class TestHelper {
       /* The number of threads to use */
       private final int numberOfThreads;

       /* Coordinate the threads to hit the delete operation all at 'the same time' */
       private final CyclicBarrier barrier;

       /* Count the number of successful deletes (which should be 0) and any errors (which should be none) */
       private final AtomicInteger deletedCount = new AtomicInteger();
       private final AtomicInteger errorCount = new AtomicInteger();

       private static TestHelper withThreads( final int numberOfThreads ) {
         return new TestHelper( numberOfThreads );
       }

       private TestHelper( final int numberOfThreads ) {
         this.numberOfThreads = numberOfThreads;
         this.barrier = new CyclicBarrier( numberOfThreads );
       }

       private TestHelper runTestAgainstService( final ContactUsService service ) throws InterruptedException {
         final Runnable deleteTask = createDeleteTask( service );
         final List<Thread> threads = runTaskOnThreads( deleteTask );
         waitForAllThreadsToFinish( threads );
         return this;
       }

       private Runnable createDeleteTask( final ContactUsService service ) {
         return () -> {
           try {
             /* Wait for all threads to reach this point */
             barrier.await( 10, TimeUnit.SECONDS );
             final Optional<Office> deleted = service.delete( ENTITY.getName() );
             deleted.ifPresent( e -> deletedCount.incrementAndGet() );
           } catch ( final Exception e ) {
             errorCount.incrementAndGet();
           }
         };
       }

       private List<Thread> runTaskOnThreads( final Runnable deleteTask ) {
         final List<Thread> threads = new ArrayList<>( numberOfThreads );

         for ( int i = 1; i <= numberOfThreads; i++ ) {
           final Thread thread = new Thread( deleteTask, String.format( "DELETE-%d", i ) );
           thread.start();
           threads.add( thread );
         }
         return threads;
       }

       private void waitForAllThreadsToFinish( final List<Thread> threads ) throws InterruptedException {
         for ( final Thread thread : threads ) {
           thread.join( TimeUnit.SECONDS.toMillis( 5 ) );
         }
       }

       private void assertAllWentAsExpected() {
         assertAllThreadsRan();
         assertOnlyOneOfficeIsDeleted();
         assertNoErrorsOccurred();
       }

       private TestHelper assertOnlyOneOfficeIsDeleted() {
         assertThat( deletedCount.intValue() )
           .describedAs( "only one deletion should succeed" )
           .isEqualTo( 1 );
         return this;
       }

       private TestHelper assertNoErrorsOccurred() {
         assertThat( errorCount.intValue() )
           .describedAs( "no errors should occur" )
           .isEqualTo( 0 );
         return this;
       }

       private TestHelper assertAllThreadsRan() {
         assertThat( barrier.getNumberWaiting() )
           .describedAs( "all threads should have ran" )
           .isEqualTo( 0 );
         return this;
       }
     }
   }
   ```

   Run the test.

   ```bash
   $ ./gradlew clean integrationTest

   ...
   BUILD FAILED in 14s
   6 actionable tasks: 6 executed
   ```

   As expected, the test should still fail as it did before.

   ```bash
   $ open "build/reports/tests/integrationTest/classes/demo.boot.JpaContactUsServiceMultiDeleteTest.html"
   ```

   ![Office-Multiple-Delete-Test-Fail-1.png]({{ '/assets/images/Office-Multiple-Delete-Test-Fail-1.png' | absolute_url }})

### Transactions

We need to make the two operations, `findById()` and `delete()` methods, atomic.  We can use the [`@Transactional`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/transaction/annotation/Transactional.html) annotation, as shown next.

1. Use transactions

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
     public Optional<Office> findOneByName( final String office ) { /* ... */ }

     @Override
     public List<Office> findAllInCountry( final String country ) { /* ... */ }

     @Override
     @Transactional
     @Retryable( ObjectOptimisticLockingFailureException.class )
     public Optional<Office> update( final Office office ) { /* ... */ }

     @Override
     @Transactional
     public Optional<Office> delete( final String name ) {
       return repository
         .findById( name )
         .map( deleteEntity() )
         .map( mapToOffice() );
     }

     private Function<OfficeEntity, OfficeEntity> deleteEntity() { /* ... */ }

     private List<Office> mapToOffices( final List<OfficeEntity> entities ) { /* ... */ }

     private Function<OfficeEntity, Office> mapToOffice() { /* ... */ }

     private Function<OfficeEntity, OfficeEntity> updateEntity( final Office office ) { /* ... */ }
   }
   ```

1. Run the tests

   ```bash
   $ ./gradlew clean integrationTest
   ```

   The test will still fail, but for a different reason.

   ```bash
   $ open "build/reports/tests/integrationTest/classes/demo.boot.JpaContactUsServiceMultiDeleteTest.html"
   ```

   ![Office-Multiple-Delete-Test-Fail-2.png]({{ '/assets/images/Office-Multiple-Delete-Test-Fail-2.png' | absolute_url }})

   We have three assertions, two of which are passing.

   ```java
      assertAllThreadsRan();
      assertOnlyOneOfficeIsDeleted();
      assertNoErrorsOccurred();
   ```

   The last assertion failed.  Even though it is not visible, as the exceptions were suppressed for simplicity, the `delete()` method, within the `JpaContactUsService` class, is now failing with a [`ObjectOptimisticLockingFailureException`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/orm/ObjectOptimisticLockingFailureException.html).

   This is an improvement as now the concurrency problem is not unnoticed.

While this solves our race condition, we can do better as shown [next](#retry-on-error).

### Retry on error

We can do better than simply failing.  In our case, we can retry the office delete operation when we encounter this error.  [Spring Batch](https://spring.io/projects/spring-batch) provides the retry functionality.

1. Add the [`spring-boot-starter-batch`](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-batch) dependency

   Update file: `build.gradle`

   {% include custom/note.html details="The following dependency may already be in the file as this may have been added while following a different example." %}

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

   {% include custom/note.html details="The <code>@EnableRetry</code> annotation may already be in the file as this may have been added while following a different example." %}

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
     public List<Office> list() {
       return mapToOffices( repository.findAll() );
     }

     @Override
     public Optional<Office> findOneByName( final String office ) {
       return repository
         .findById( office )
         .map( mapToOffice() );
     }

     @Override
     public List<Office> findAllInCountry( final String country ) {
       return mapToOffices( repository.findAllByCountryIgnoreCase( country ) );
     }

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

     @Override
     @Transactional
     @Retryable( ObjectOptimisticLockingFailureException.class )
     public Optional<Office> delete( final String name ) {
       return repository
         .findById( name )
         .map( deleteEntity() )
         .map( mapToOffice() );
     }

     private Function<OfficeEntity, OfficeEntity> deleteEntity() {
       return entity -> {
         repository.delete( entity );
         return entity;
       };
     }

     private List<Office> mapToOffices( final List<OfficeEntity> entities ) {
       return entities
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

1. Run the tests

   ```bash
   $ ./gradlew clean check

   ...

   BUILD SUCCESSFUL in 19s
   7 actionable tasks: 7 executed
   ```

   The test should now pass.

## Tasks status

- [X] Return one office by id
- [X] Return all offices in country
- [X] Update one office
- [X] Delete an office
