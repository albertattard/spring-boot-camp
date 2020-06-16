---
layout: default
title: Demo
parent: Spring Data
nav_order: 99
permalink: docs/data/demo/
---

# Contact us
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

```java
package demo.boot;

import lombok.Data;

import javax.persistence.Entity;
import javax.persistence.Id;
import javax.persistence.Table;

@Data
@Entity
@Table( name = "offices" )
public class OfficeEntity {

  @Id
  private String office;
  private String address;
  private String country;
  private String phone;
  private String email;
  private String webpage;
}
```

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
      entity.getOffice(),
      entity.getAddress(),
      entity.getPhone(),
      entity.getEmail()
    );
  }
}
```

```bash
$ ./gradlew clean build

...
BUILD SUCCESSFUL in 9s
6 actionable tasks: 6 executed
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
public class JpaContactUsService implements ContactUsService { /* ... */ }
```


View the SQL

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

Enable H2 Console

```yaml
spring:

  h2:
    console:
      enabled: true
      path: /h2
```


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

Remove CSV

src/main/java/demo/boot/CsvContactUsService.java
src/main/resources/offices.csv
src/test/java/demo/boot/CsvContactUsServiceTest.java

```groovy
  /* CSV */
  implementation 'org.apache.commons:commons-csv:1.8'
```


![H2 Console]({{ '/assets/images/H2-Console.png' | absolute_url }})

## Use environment variables

```properties
DATABASE_NAME=contact-us
DATABASE_PORT=
DATABASE_URL=jdbc:h2:mem:contact-us;DATABASE_TO_UPPER=false;
DATABASE_DRIVER=org.h2.Driver
DATABASE_USERNAME=tw-data
DATABASE_PASSWORD=SomeRandomPassword
```

```yaml
spring:
  datasource:
    url: ${DATABASE_URL}
    driver-class-name: ${DATABASE_DRIVER}
    username: ${DATABASE_USERNAME}
    password: ${DATABASE_PASSWORD}
```

```bash
$ ./gradlew clean build

...
BUILD FAILED in 9s
6 actionable tasks: 6 executed
```

## Integration tests

Create file: `integration-test.gradle`

```groovy
configurations {
  integrationTestImplementation.extendsFrom testImplementation
  integrationTestRuntimeOnly.extendsFrom testRuntimeOnly
}

sourceSets {
  integrationTest {
    java {
      compileClasspath += main.output + test.output
      runtimeClasspath += main.output + test.output
      srcDir file('src/test-integration/java')
    }

    resources.srcDir file('src/test-integration/resources')
  }
}

task integrationTest(type: Test) {
  description = 'Runs the integration tests.'
  group = 'verification'
  testClassesDirs = sourceSets.integrationTest.output.classesDirs
  classpath = sourceSets.integrationTest.runtimeClasspath
  outputs.upToDateWhen { false }
  mustRunAfter test

  useJUnitPlatform()

  testLogging {
    events = ['FAILED', 'PASSED', 'SKIPPED', 'STANDARD_OUT']
  }

  doFirst {
    file("$rootDir/.env").readLines().each() {
      def (key, value) = it.split('=', 2)
      environment key, value
    }
  }
}

check {
  dependsOn integrationTest
}
```

```groovy
/* Integration Tests */
apply from: "$rootDir/integration-test.gradle"
```

Create dir structure

```bash
$ tree src/test-integration
src/test-integration
├── java
└── resources
```

![Test Integration Folder Structure]({{ '/assets/images/Test-Integration-Folder-Structure.png' | absolute_url }})

```bash
$ ./gradlew tasks
```

```bash
...
Verification tasks
------------------
check - Runs all checks.
integrationTest - Runs the integration tests.
pitest - Run PIT analysis for java classes
test - Runs the unit tests.
...
```

Move the `ContactUsApplicationTests` to the `test-intergration`

```bash
tree src
src
├── main
...
├── test
│   └── java
│       └── demo
│           └── boot
│               ├── JpaContactUsServiceTest.java
│               └── OfficeControllerTest.java
└── test-integration
    └── java
        └── demo
            └── boot
                └── ContactUsApplicationTests.java

16 directories, 15 files
```

```bash
$ ./gradlew integrationTest

...
BUILD SUCCESSFUL in 8s
5 actionable tasks: 1 executed, 4 up-to-date
```

## PostgreSQL

Create file: `docker-compose.yml`

```yaml
version: "3"
services:
  postgres:
    container_name: ${DATABASE_NAME}
    image: postgres:11.1
    restart: unless-stopped
    networks:
      - app-net
    env_file:
      - .env
    ports:
      - ${DATABASE_PORT}:5432
    environment:
      PGUSER: ${DATABASE_USERNAME}
      POSTGRES_USER: ${DATABASE_USERNAME}
      POSTGRES_PASSWORD: ${DATABASE_PASSWORD}
      POSTGRES_DB: ${DATABASE_NAME}
    healthcheck:
      test: ["CMD", "pg_isready", "-U", "postgres", "-d", "${DATABASE_NAME}"]
      interval: 10s
      timeout: 5s
      retries: 5
  pgadmin4:
    container_name: pgadmin4
    image: dpage/pgadmin4:4.22
    restart: unless-stopped
    networks:
      - app-net
    env_file:
      - .env
    volumes:
      - ./docker/pgadmin4:/pgadmin4/external
    environment:
      PGADMIN_DEFAULT_EMAIL: ${DATABASE_USERNAME}
      PGADMIN_DEFAULT_PASSWORD: ${DATABASE_PASSWORD}
      PGADMIN_SERVER_JSON_FILE: /pgadmin4/external/servers.json
    ports:
      - "8000:80"
networks:
  app-net:
    driver: bridge
```


```properties
DATABASE_NAME=contact-us
DATABASE_PORT=5432
DATABASE_URL=jdbc:postgresql://localhost:5432/contact-us
DATABASE_DRIVER=org.postgresql.Driver
DATABASE_USERNAME=tw-data
DATABASE_PASSWORD=SomeRandomPassword
```


```bash
$ docker-compose up -d

Creating network "contact-us_game-net" with driver "bridge"
Creating contact-us ... done
Creating pgadmin4   ... done
```

```bash
docker-compose stop && docker system prune -f
Stopping contact-us ... done
Stopping pgadmin4   ... done
Deleted Containers:
68e15b698c143816149f0463399a394dcf97b6529478d65ae2d1bb1c10a11685
7c3325978fbf79d1e41f0c60c4901aac7b65a443802d8a7bd7605743f5678396

Deleted Networks:
contact-us_app-net

Total reclaimed space: 154B
```

Create file `docker/pgadmin4/servers.json`

```json
{
  "Servers": {
    "1": {
      "Name": "Contact US DB",
      "Group": "Servers",
      "Host": "postgres",
      "Port": 5432,
      "MaintenanceDB": "contact-us",
      "Username": "tw-data",
      "SSLMode": "prefer",
      "SSLCert": "<STORAGE_DIR>/.postgresql/postgresql.crt",
      "SSLKey": "<STORAGE_DIR>/.postgresql/postgresql.key",
      "SSLCompression": 0,
      "Timeout": 10,
      "UseSSHTunnel": 0,
      "TunnelPort": "22",
      "TunnelAuthentication": 0
    }
  }
}
```

```groovy
  runtimeOnly 'com.h2database:h2'
```

```groovy
  runtimeOnly 'org.postgresql:postgresql'
```

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
  implementation 'org.flywaydb:flyway-core'
  runtimeOnly 'org.postgresql:postgresql'

  /* Spring/OpenApi */
  implementation 'org.springdoc:springdoc-openapi-ui:1.3.9'
}
```

```bash
$ ./gradlew integrationTest
```

![PgAdmin Tables Created]({{ '/assets/images/PgAdmin-Tables-Created.png' | absolute_url }})


```bash
$ ./gradlew bootRun

...
BUILD FAILED in 4s
3 actionable tasks: 1 executed, 2 up-to-date
```

Update file `build.gradle`

```groovy
bootRun {
  doFirst {
    file("$rootDir/.env").readLines().each() {
      def (key, value) = it.split('=', 2)
      environment key, value
    }
  }
}
```



docker-compose stop && docker system prune -f && docker-compose up -d && ./gradlew clean integrationTest
