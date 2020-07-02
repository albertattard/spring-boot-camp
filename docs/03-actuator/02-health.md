---
layout: default
title: Health Information
parent: Spring Boot Actuator
nav_order: 2
permalink: docs/actuator/health/
---

# Health
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Start the application

The _Contact Us_ application depends on external resources such as, the [PostgreSQL](https://www.postgresql.org/) database, to be running.  These external resources were defined as containers in a docker composed file in a [previous section]({{ '/docs/data/postgresql/#docker-compose-setup-postgresql-and-pgadmin' | absolute_url }}).  Start the external resources (containers) using docker compose, as shown next.

```bash
$ docker-compose up -d
```

The external resources (containers) defined in the docker compose file will only be started if these were not already running.  It is safe to run the above command multiple times.

In the event you like to start from scratch, use the following set of commands to delete the existing containers and then start them back again.

```bash
$ docker-compose stop && docker system prune -f && docker-compose up -d
```

With the external dependencies running, start the application.

```bash
$ ./gradlew bootRun
```

## Health endpoint

The application is making use of the [Spring Boot Actuator](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-actuator) dependency, as shown next.

File: `build.gradle`

{% include custom/note.html details="You do not need to add this dependency as this was added in a <a href='/spring-boot-camp/docs/primer/demo/health/'>previous section</a>." %}

```groovy
dependencies {
  implementation 'org.springframework.boot:spring-boot-starter-actuator'
}
```

The application exposes the health endpoint: `/heath`, as configured in the `src/main/resources/application.yaml` properties file created in a [previous section]({{ '/docs/primer/demo/health/' | absolute_url }}).

```yaml
management:
  endpoints:
    web:
      base-path:
```

Running the following CURL command (while our application is running) will return the health status of our application.

```bash
$ curl "http://localhost:8080/health" | jq .
```

Our application is up and running.

```json
{
  "status": "UP"
}
```

If our application is not healthy, the health endpoint will return a different status as shown next.

```json
{
  "status": "DOWN"
}
```

## How does the health check works?

[Spring Boot](https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-features.html#production-ready-health) provides a registry, [`HealthContributorRegistry`](https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/actuate/health/HealthContributorRegistry.html), where all health information is gathered as shown in the following diagram.

![Health-Contributor-Registry.png]({{ '/assets/images/Health-Contributor-Registry.png' | absolute_url }})

Spring scans our application for beans that implement the [`HealthContributor`](https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/actuate/health/HealthContributor.html) marker interface and automatically adds them to the registry.  Every time we request for the health status, Spring will get all contributors and checks their health status, and aggregates their response.

Spring Boot provides several health indicators, such as database connectivity health indicator.  We can also add custom health indicators as shown next.

```java
package demo.boot;

import org.springframework.boot.actuate.health.Health;
import org.springframework.boot.actuate.health.HealthIndicator;
import org.springframework.stereotype.Service;

@Service
public class CustomHealthIndicator implements HealthIndicator {

  @Override
  public Health health() {
    return Health
      .up()
      .withDetail( "type", "demo" )
      .build();
  }
}
```

Our `CustomHealthIndicator` implements the [`HealthIndicator`](https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/actuate/health/HealthIndicator.html) which is one of the subtypes of `HealthContributor`.  The `HealthIndicator` defines two methods through which our health indicator can communicate the status of this indicator.

## Can we interact with other Spring beans from within the health indicator?

In our simple example, we are hard coding the status, but these can be retrieved using other beans.  Say that we depend on a third-party service, such as the astronaut API (`http://api.open-notify.org/astros.json`).  If this service is not available, then our application cannot function correctly.

{% include custom/note.html details="This is just a fictitious example and our application does not depend on this third-party API.  It is only shown here as an example and will not form part of our application." %}

Following is an implementation of such health indicator.

```java
package demo.boot;

import io.vavr.control.Either;
import lombok.AllArgsConstructor;
import lombok.Data;
import org.springframework.boot.actuate.health.Health;
import org.springframework.boot.actuate.health.HealthIndicator;
import org.springframework.stereotype.Service;
import org.springframework.web.client.RestTemplate;

import java.util.List;
import java.util.function.Function;

@Service
public class AstronautsCountHealthIndicator implements HealthIndicator {

  private final RestTemplate restTemplate;

  public AstronautsCountHealthIndicator( final RestTemplateBuilder builder ){
    builder.setConnectTimeout( Duration.ofSeconds( 5 ) );
    builder.setReadTimeout( Duration.ofSeconds( 5 ) );
    restTemplate = builder.build();
  }

  @Override
  public Health health() {
    return astros()
      .fold( mapError(), mapAstros() );
  }

  private Function<Astros, Health> mapAstros() {
    return astros ->
      "success".equals( astros.message )
        ? Health.up().withDetail( "astronauts", astros.people ).build()
        : Health.outOfService().build();
  }

  private Function<Exception, Health> mapError() {
    return e -> Health.down( e ).build();
  }

  private Either<Exception, Astros> astros() {
    /* TODO: Is this idiomatic? */
    try {
      final Astros astros =
        restTemplate.getForObject( "http://api.open-notify.org/astros.json", Astros.class );
      return Either.right( astros );
    } catch ( final Exception e ) {
      return Either.left( e );
    }
  }

  @Data
  private static class Astros {
    private String message;
    private int number;
    private List<People> people;
  }

  @Data
  private static class People {
    private String craft;
    private String name;
  }
}
```

The above example makes use from the [`vavr`](https://mvnrepository.com/artifact/io.vavr/vavr) dependency that provides the [`Either`](https://www.javadoc.io/doc/io.vavr/vavr/latest/io/vavr/control/Either.html) functional type.

```groovy
dependencies {
  /* Either/Functional functions */
  implementation 'io.vavr:vavr:0.10.3'
}
```

Our new health indicator requires a [`RestTemplate`](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/web/client/RestTemplate.html), which is used to retrieve the list of astronauts currently in space and map the received JSON response into a Java object.  Spring Boot does not provide us with a `RestTemplate`.  Instead, Spring provides us with a builder of type [`RestTemplateBuilder`](https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/web/client/RestTemplateBuilder.html) and we need to create one from it.

```java
  public AstronautsCountHealthIndicator( final RestTemplateBuilder builder ){
    builder.setConnectTimeout( Duration.ofSeconds( 5 ) );
    builder.setReadTimeout( Duration.ofSeconds( 5 ) );
    restTemplate = builder.build();
  }
```

This is just an example of a health indicator that checks the availability of a third-party endpoint that our application depends on using other Spring managed beans.

## How can we determine what health indicators we have?

Spring Boot retrieves the `Health` from each indicator and using a [`StatusAggregator`](https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/actuate/health/StatusAggregator.html) to constructs the application overall health status.  If one of the indicators is down, then the whole application status is considered a down, as shown next.

```json
{
  "status": "DOWN"
}
```

The above state is an aggregated state and does not provide any further detail.  The health endpoint can be configured such that it can provide more detailed information, a shown next.

Update file: `src/main/resources/application.yaml`

```yaml
management:
  endpoints:
    web:
      base-path:
  endpoint:
    health:
      show-details: always
```

The `show-details` property can be set to any of the following three values:

| Value             | Description                                 |
| ----------------- | ------------------------------------------- |
| `never`           | Details are never shown. (default)          |
| `when_authorised` | Details are only shown to authorized users. |
| `always`          | Details are shown to all users.             |

Retrieving the application health with the `show-details` property set to `always` will return a detailed breakdown for each health indicator.

```bash
$ curl "http://localhost:8080/health" | jq .
```

We will now receive a longer response with the information provided by each health indicator.

```json
{
  "status": "UP",
  "components": {
    "astronautsCount": {
      "status": "UP",
      "details": {
        "astronauts": [
          {
            "craft": "ISS",
            "name": "Chris Cassidy"
          },
          {
            "craft": "ISS",
            "name": "Anatoly Ivanishin"
          },
          {
            "craft": "ISS",
            "name": "Ivan Vagner"
          },
          {
            "craft": "ISS",
            "name": "Doug Hurley"
          },
          {
            "craft": "ISS",
            "name": "Bob Behnken"
          }
        ]
      }
    },
    "custom": {
      "status": "UP",
      "details": {
        "type": "demo"
      }
    },
    "db": {
      "status": "UP",
      "details": {
        "database": "PostgreSQL",
        "validationQuery": "isValid()"
      }
    },
    "diskSpace": {
      "status": "UP",
      "details": {
        "total": 250685575168,
        "free": 13821788160,
        "threshold": 10485760,
        "exists": true
      }
    },
    "ping": {
      "status": "UP"
    }
  }
}
```

In the above response, we have health information from five health indicators, only two of which we built.

| Component         | Description                                                                                          |
| ----------------- | ---------------------------------------------------------------------------------------------------- |
| `astronautsCount` | Checks the [http://api.open-notify.org/astros.json](http://api.open-notify.org/astros.json) endpoint |
| `custom`          | A basic health indicator that always returns the same response                                       |
| `db`              | Database health indicator provided by Spring Boot                                                    |
| `diskSpace`       | Disk space health indicator provided by Spring Boot                                                  |
| `ping`            | Ping health indicator provided by Spring Boot                                                        |

Each indicator has a name and provides two values, as shown next.

```json
"custom": {
  "status": "UP",
  "details": {
    "type": "demo"
  }
}
```

The `status` property shown the actual status for the respective health indicator, while the `details` property shows any details provided by the respective health indicator as shown in the following code fragment.

```java
  public Health health() {
    return Health
      .up()
      .withDetail( "type", "demo" )
      .build();
  }
```

In the example we saw above, all five health indicators are `UP`.  If one of these is down, using the details we can identify the root cause of the problem.

## Simulate a problem

Let's simulate a problem, by stopping the database while the application is running.

1. Start the external resources (docker-compose) if these are not already running.

   ```bash
   $ docker-compose up -d
   ```

1. Start our application (while the external resources running)

   ```
   $ ./gradlew bootRun
   ```

1. Check the health endpoint

   ```bash
   $ curl "http://localhost:8080/health" | jq .
   ```

   This should return `UP` together with the details of each component as we saw before.

1. Stop the database

   ```bash
   $ docker-compose stop
   ```

   Wait for the containers defined in the docker compose file to complete as shown next

   ```bash
   Stopping contact-us-pgadmin4   ... done
   Stopping contact-us-pg         ... done
   ```

1. With our application running and the database stopped, retrieve the application's health status

   {% include custom/note.html details="Each health indicator is evaluated on request.  In our example, the request will take about 30 seconds to reply, the time needed by the database connection to timeout." %}

   ```bash
   $ curl "http://localhost:8080/health" | jq .
   ```

   After about 30 seconds, we will receive an unhealthy reply with the status of the application marked as `DOWN`, as shown next.

   ```json
   {
     "status": "DOWN",
     "components": {
       "astronautsCount": {
         "status": "UP",
         "details": {
           "astronauts": [
             {
               "craft": "ISS",
               "name": "Chris Cassidy"
             },
             {
               "craft": "ISS",
               "name": "Anatoly Ivanishin"
             },
             {
               "craft": "ISS",
               "name": "Ivan Vagner"
             },
             {
               "craft": "ISS",
               "name": "Doug Hurley"
             },
             {
               "craft": "ISS",
               "name": "Bob Behnken"
             }
           ]
         }
       },
       "custom": {
         "status": "UP",
         "details": {
           "type": "demo"
         }
       },
       "db": {
         "status": "DOWN",
         "details": {
           "error": "org.springframework.jdbc.CannotGetJdbcConnectionException: Failed to obtain JDBC Connection; nested exception is java.sql.SQLTransientConnectionException: HikariPool-1 - Connection is not available, request timed out after 30004ms."
         }
       },
       "diskSpace": {
         "status": "UP",
         "details": {
           "total": 250685575168,
           "free": 13686542336,
           "threshold": 10485760,
           "exists": true
         }
       },
       "ping": {
         "status": "UP"
       }
     }
   }
   ```

   From the detailed description we can tell what's causing the problem and we can act on the root cause.

   The status of this response is a non-200 response as shown next

   ```bash
   $ curl -v --output /dev/null "http://localhost:8080/health"

   ...

   > GET /health HTTP/1.1
   > Host: localhost:8080
   > User-Agent: curl/7.64.1
   > Accept: */*
   >
   < HTTP/1.1 503
   < Content-Type: application/vnd.spring-boot.actuator.v3+json
   < Transfer-Encoding: chunked
   < Date: Tue, 27 April 2077 12:34:56 GMT
   < Connection: close
   ```

## Cleanup

In this section we created two health indicators (`AstronautsCountHealthIndicator` and `CustomHealthIndicator`) which are not required by our application.  These can be safely deleted from the application.

```bash
$ rm src/main/java/demo/boot/AstronautsCountHealthIndicator.java
$ rm src/main/java/demo/boot/CustomHealthIndicator.java
```
