---
layout: default
title: Update individual office details
parent: Custom Queries
grand_parent: Spring Data
nav_order: 4
permalink: docs/data/custom-queries/update/
---

# Update individual office details
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Update individual office details

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

## Concurrent updates

## Tasks status

- [X] Return one office by id
- [X] Return all offices in country
- [X] Update individual office details
- [ ] Delete an office
