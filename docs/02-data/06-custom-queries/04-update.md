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

1. a

   Update file: `src/main/java/demo/boot/ContactUsService.java`

   ```java
   package demo.boot;

   import java.util.List;
   import java.util.Optional;

   public interface ContactUsService {

     List<Office> list();

     Optional<Office> findOneByOffice( final String office );

     List<Office> findAllInCountry( final String country );

     Optional<Office> update( final Office office );
   }
   ```

   Update file: `src/main/java/demo/boot/JpaContactUsService.java`

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
     public Optional<Office> findOneByOffice( final String office ) { /* ... */ }

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

1. b

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
     @DisplayName( "should return empty optional if the office being saved does not exists" )
     public void shouldSaveNonExistentOffice() {
       final String id = "a1";
       final Office office = new Office( id, "a2", "a3", "a4" );

       final OfficesRepository repository = mock( OfficesRepository.class );
       when( repository.findById( eq( id ) ) ).thenReturn( Optional.empty() );

       final ContactUsService service = new JpaContactUsService( repository );
       final Optional<Office> saved = service.update( office );

       assertEquals( Optional.empty(), saved );

       verify( repository, times( 1 ) ).findById( id );
       verifyNoMoreInteractions( repository );
     }
   }
   ```

   ```bash
   $ ./gradlew clean test

   ...

   JPA contact us service > should return empty optional if the office being saved does not exists FAILED
       org.mockito.exceptions.verification.WantedButNotInvoked at JpaContactUsServiceTest.java:118

   ...
   BUILD FAILED in 7s
   5 actionable tasks: 5 executed
   ```

1. c

   Update file: `src/main/java/demo/boot/JpaContactUsService.java`

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
     public Optional<Office> findOneByOffice( final String office ) { /* ... */ }

     @Override
     public List<Office> findAllInCountry( final String country ) { /* ... */ }

     @Override
     public Optional<Office> update( final Office office ) {
       return repository
         .findById( office.getOffice() )
         .map( mapToOffice() );
     }

     private List<Office> mapToOffices( final List<OfficeEntity> entities ) { /* ... */ }

     private Function<OfficeEntity, Office> mapToOffice() { /* ... */ }
   }
   ```

   ```bash
   $ ./gradlew clean test

   ...

   BUILD SUCCESSFUL in 15s
   5 actionable tasks: 5 executed
   ```

1. d

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
     @DisplayName( "should return empty optional if the office being saved does not exists" )
     public void shouldSaveNonExistentOffice() { /* ... */ }

     @Test
     @DisplayName( "should update the office and return the updated version" )
     public void shouldSaveOffice() {
       final String id = "a1";
       final Office office = new Office( id, "a2", "a4", "a5" );
       final OfficeEntity existingEntity = new OfficeEntity( id, "o2", "o3", "o4", "o5", "o6" );
       final OfficeEntity updatedEntity = new OfficeEntity( id, "a2", "o3", "a4", "a5", "o6" );

       final OfficesRepository repository = mock( OfficesRepository.class );
       when( repository.findById( eq( id ) ) ).thenReturn( Optional.of( existingEntity ) );
       when( repository.save( eq( updatedEntity ) ) ).thenReturn( updatedEntity );

       final ContactUsService service = new JpaContactUsService( repository );
       final Optional<Office> saved = service.update( office );

       assertEquals( Optional.of( office ), saved );

       verify( repository, times( 1 ) ).findById( id );
       verify( repository, times( 1 ) ).save( updatedEntity );
     }
   }
   ```

   ```bash
   $ ./gradlew clean test

   ...

   BUILD FAILED in 8s
   5 actionable tasks: 5 executed
   ```

1. e

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
     public Optional<Office> findOneByOffice( final String office ) { /* ... */ }

     @Override
     public List<Office> findAllInCountry( final String country ) { /* ... */ }

     @Override
     public Optional<Office> update( final Office office ) {
       return repository
         .findById( office.getOffice() )
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
     public Optional<Office> findOneByOffice( final String office ) { /* ... */ }

     @Override
     public List<Office> findAllInCountry( final String country ) { /* ... */ }

     @Override
     public Optional<Office> update( final Office office ) {
       return repository
         .findById( office.getOffice() )
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

## Are concurrent updates a problem?

Each individual update is atomic by nature.  In our current example, if two or more updates happen at the same time, these are simply applied in the order they are received by the database.

In the [next section]({{ '/docs/data/custom-queries/delete/' | absolute_url }}), we will introduce the delete operation, which simply deletes the office from the database.  This will introduce a new complexity as the update should not happen if the office does not exist.  We should not be able to update an office that was delete.  Consider the following scenario, where a delete operation happens while an item is being updated on two different threads (_T1_ and _T2_).

| Time | Update (_T1_)              | Delete (_T2_)     |
| :--: | -------------------------- | ----------------- |
|   1  | `findById()` return office |                   |
|   2  |                            | `delete()` office |
|   3  | `save()` changes           |                   |

The above table shows two things happening at the same time, on two separate threads (_T1_ and _T2_).  The sequence shown above will leave the data in an invalid state as the office will be saved back to the database (Time: 3) after this was deleted (Time: 2).  Irrespective of the order of the update and delete operations, the item should be deleted.

* When the update happens before the delete, the update will succeed and then the item is deleted
* When the delete happens before the update, the update will not happen as the item does not exist

```java
package demo.boot;

import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.context.SpringBootTest.WebEnvironment;

import java.util.Optional;

import static org.assertj.core.api.Assertions.assertThat;

@DisplayName( "JPA contact us service" )
@SpringBootTest( webEnvironment = WebEnvironment.NONE )
public class JpaContactUsServiceTest {

  @Autowired
  private OfficesRepository repository;

  @Autowired
  private ContactUsService service;

  private static final OfficeEntity COLOGNE = new OfficeEntity(
    "ThoughtWorks Cologne",
    "Lichtstr. 43i, 50825 Cologne, Germany",
    "Germany",
    "+49 221 64 30 70 63",
    "contact-de@thoughtworks.com",
    "https://www.thoughtworks.com/locations/cologne"
  );

  @BeforeEach
  public void before() {
    repository.deleteAll();
    repository.save( COLOGNE );
  }

  @Test
  @DisplayName( "should update the office and return the updated version" )
  public void shouldHandleConcurrentUpdates() {
    final Office office = new Office( COLOGNE.getOffice(), "b", "c", "d" ) {
      @Override
      public String getAddress() {
        /* Delete the office between the findById() and save() */
        deleteOfficeFromAnotherThread();
        return super.getAddress();
      }
    };

    service.update( office );

    /* The office should not be in the database, as this was deleted and the update should not succeed */
    final Optional<OfficeEntity> entity = repository.findById( COLOGNE.getOffice() );
    assertThat( entity.isEmpty() ).isTrue();
  }

  private void deleteOfficeFromAnotherThread() {
    try {
      final Thread delete = new Thread( () -> repository.delete( COLOGNE ), "DELETE" );
      delete.start();
      delete.join();
    } catch ( InterruptedException e ) {
    }
  }
}
```

## Tasks status

- [X] Return one office by id
- [X] Return all offices in country
- [X] Update one office
- [ ] Delete an office
