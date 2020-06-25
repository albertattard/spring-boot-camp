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

{% include custom/pending.html %}

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

```bash
$ ./gradlew clean test

...
JPA contact us service > should return empty optional if the office being deleted does not exist FAILED
    org.mockito.exceptions.verification.WantedButNotInvoked at JpaContactUsServiceTest.java:156
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
    return repository
      .findById( name )
      .map( mapToOffice() );
  }

  private List<Office> mapToOffices( final List<OfficeEntity> entities ) { /* ... */ }

  private Function<OfficeEntity, Office> mapToOffice() { /* ... */ }

  private Function<OfficeEntity, OfficeEntity> updateEntity( final Office office ) { /* ... */ }
}
```

```bash
$ ./gradlew clean test

...
BUILD SUCCESSFUL in 5s
5 actionable tasks: 5 executed
```

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
  }
}
```

```bash
$ ./gradlew clean test

...

JPA contact us service > should delete the office and return the deleted office FAILED
    org.mockito.exceptions.verification.WantedButNotInvoked at JpaContactUsServiceTest.java:178

...

BUILD FAILED in 7s
5 actionable tasks: 5 executed
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

```bash
$ ./gradlew clean test

...

BUILD SUCCESSFUL in 6s
5 actionable tasks: 5 executed
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

```java
  @Override
  @Transactional
  @Retryable( ObjectOptimisticLockingFailureException.class )
  public Optional<Office> delete( final String name ) {
    return repository
      .findById( name )
      .map( deleteEntity() )
      .map( mapToOffice() );
  }
```

## Tasks status

- [X] Return one office by id
- [X] Return all offices in country
- [X] Update one office
- [X] Delete an office
