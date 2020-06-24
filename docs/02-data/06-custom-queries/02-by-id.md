---
layout: default
title: Return one office by id
parent: Custom Queries
grand_parent: Spring Data
nav_order: 2
permalink: docs/data/custom-queries/find-by-id/
---

# Return one office by id
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Return one office by id (`findById()`)

When querying the repository for an entity by its id, there is a chance where the entity for the given id is not found.

**What should we do when no office is found for the given office id?**

There are several approaches to handle this.

* Throw an exception, such as `OfficeNotFoundException` (_Not recommended_)

  This is not recommended as exceptions are very slow and add little value in this case.  Furthermore, exceptions tend to disrupt the flow as these need to be wrapped within a `try/catch` and then dealt with accordingly.

* Return `null` (_Less recommended_)

  This is very common approach to indicate that something was not found, but discouraged as we have better alternatives, such as the one mentioned next.  Using `null` tend to lead to [`NullPointerException`](https://docs.oracle.com/en/java/javase/14/docs/api/java.base/java/lang/NullPointerException.html), referred to as [the Billion Dollar Mistake](https://www.infoq.com/presentations/Null-References-The-Billion-Dollar-Mistake-Tony-Hoare/).  This is very error prone and does not fit very well with functional programming.

* Return an [empty optional](https://docs.oracle.com/en/java/javase/14/docs/api/java.base/java/util/Optional.html#empty()) (_Recommended_)

  This is the recommended approach to indicate that the value was not found.  [Spring Data JPA](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/) was improved and now returns [an empty optional when the entity with the given id is not found](https://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/repository/CrudRepository.html#findById-ID-).  Furthermore, the [`Optional`](https://docs.oracle.com/en/java/javase/14/docs/api/java.base/java/util/Optional.html), plays well with functional Java as we will see later on.

[Java 8](https://openjdk.java.net/projects/jdk8/) introduced the[`Optional`](https://docs.oracle.com/en/java/javase/14/docs/api/java.base/java/util/Optional.html) type, which we will use to wrap our type in.  Our service will add a new method `findOneByOffice()`, that takes a `String` and returns `Optional<Office>`.

1. Add method to interface

   Update file: `src/main/java/demo/boot/ContactUsService.java`

   ```java
   package demo.boot;

   import java.util.List;
   import java.util.Optional;

   public interface ContactUsService {

     List<Office> list();

     Optional<Office> findOneByOffice( final String office );
   }
   ```

1. Make the service implementation compile

   Update file: `src/main/java/demo/boot/JpaContactUsService.java`

   ```java
     @Override
     public Optional<Office> findOneByOffice( final String office ) {
       return Optional.empty();
     }
   ```

   {% include custom/note.html details="The above example simply returns empty optional, just enough to compile." %}

   Following is a complete example

   ```java
   package demo.boot;

   import lombok.AllArgsConstructor;
   import org.springframework.context.annotation.Primary;
   import org.springframework.stereotype.Service;

   import java.util.List;
   import java.util.Optional;
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
     public Optional<Office> findOneByOffice( final String office ) {
       return Optional.empty();
     }

     private Function<OfficeEntity, Office> mapToOffice() { /* ... */ }
   }
   ```

1. Test when office is not found

   The service should return an empty optional when an office with the given id is not found.

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
   import static org.mockito.Mockito.when;

   @DisplayName( "JPA contact us service" )
   public class JpaContactUsServiceTest {

     @Test
     @DisplayName( "should return all offices returned by the repository" )
     public void shouldReturnOffices() { /* ... */ }

     @Test
     @DisplayName( "should query for the with a given id and return optional empty when the office is not found" )
     public void shouldReturnOptionEmpty() {
       final String id = "a1";

       final OfficesRepository repository = mock( OfficesRepository.class );
       when( repository.findById( eq( id ) ) ).thenReturn( Optional.empty() );

       final ContactUsService service = new JpaContactUsService( repository );
       final Optional<Office> office = service.findOneByOffice( id );

       assertEquals( Optional.empty(), office );

       verify( repository, times( 1 ) ).findById( id );
     }
   }
   ```

   Run the test.

   ```bash
   $ ./gradlew clean test

   ...
   JPA contact us service > should query for the with a given id and return optional empty when the office is not found FAILED
       org.mockito.exceptions.verification.WantedButNotInvoked at JpaContactUsServiceTest.java:56
   ...

   BUILD FAILED in 6s
   5 actionable tasks: 5 executed
   ```

   The test will fail despite the fact that our service returns an empty optional.  That's because the service did not interact with the repository as it was expected by the test.

1. Make the test pass

   Update file: `src/main/java/demo/boot/JpaContactUsService.java`

   ```java
     @Override
     public Optional<Office> findOneByOffice( final String office ) {
       return repository
         .findById( office )
         .map( mapToOffice() );
     }
   ```

   Our new test is reusing a method created before, `mapToOffice()`, to convert form `OfficeEntity` to `Office`.  Following is the complete example.

   ```java
   package demo.boot;

   import lombok.AllArgsConstructor;
   import org.springframework.context.annotation.Primary;
   import org.springframework.stereotype.Service;

   import java.util.List;
   import java.util.Optional;
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
     public Optional<Office> findOneByOffice( final String office ) {
       return repository
         .findById( office )
         .map( mapToOffice() );
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

   Run the tests.

   ```bash
   $ ./gradlew clean test

   ...
   BUILD SUCCESSFUL in 7s
   5 actionable tasks: 5 executed
   ```

   The test should now pass.

1. Test when the office with the given id is found

   Update file: `/Users/albertattard/work/projects/albertattard/contact-us/src/test/java/demo/boot/JpaContactUsServiceTest.java`

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
     public void shouldReturnOffice() {
       final String id = "a1";
       final OfficeEntity entity = new OfficeEntity( id, "a2", "a3", "a4", "a5", "a6" );

       final OfficesRepository repository = mock( OfficesRepository.class );
       when( repository.findById( eq( id ) ) ).thenReturn( Optional.of( entity ) );

       final ContactUsService service = new JpaContactUsService( repository );
       final Optional<Office> office = service.findOneByOffice( id );

       final Optional<Office> expected = Optional.of( new Office( "a1", "a2", "a4", "a5" ) );
       assertEquals( expected, office );

       verify( repository, times( 1 ) ).findById( id );
     }
   }
   ```

   Run the test.

   ```bash
   $ ./gradlew clean test

   ...
   BUILD SUCCESSFUL in 7s
   5 actionable tasks: 5 executed
   ```

   The tests pass as the interaction with the repository was already implemented in the previous test.  This works for both tests.

## What is the difference between `findById()` and `getOne()` repository methods?

In our example we used the [`findById()`](https://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/repository/CrudRepository.html#findById-ID-) method, defined by the [`CrudRepository`](https://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/repository/CrudRepository.html) interface to retrieve the office by the given id.  The [`JpaRepository`](https://docs.spring.io/spring-data/jpa/docs/current/api/org/springframework/data/jpa/repository/JpaRepository.html) interface defines the [`getOne()`](https://docs.spring.io/spring-data/jpa/docs/current/api/org/springframework/data/jpa/repository/JpaRepository.html#getOne-ID-) method, which too returns an office by its id.

The difference between these two methods is that the `findById()` method is eager, while the `getOne()` method is lazy.  This has no impact in our case as we only have one table and we do not have any relations.

Say that we have another table containing all employees which is linked to the offices table and also another entity that represent the employee in Java.  Then using the `findById()` method will retrieve the office and all employees that are linked to this office immediately.  On the other hand, if we use the `getOne()` method, the office details are read from the table but the employees are not.  Only when we request the list of employees, these are fetched from the database and then returned.

## Tasks

- [X] Return one office by id
- [ ] Filter by country
- [ ] Update individual office details
- [ ] Delete an office
