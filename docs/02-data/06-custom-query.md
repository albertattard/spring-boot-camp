---
layout: default
title: Custom Query
parent: Spring Data
nav_order: 99
permalink: docs/data/custom-query/
---

# Custom Query
{: .no_toc }

We are able to return all office using the provided [`findAll()`]() method.  In this section we will introduce custom queries.

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#jpa.query-methods

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
