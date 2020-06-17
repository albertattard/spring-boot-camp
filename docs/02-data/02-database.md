---
layout: default
title: Setup Database
parent: Spring Data
nav_order: 2
permalink: docs/data/database/
---

# Setup Database
{: .no_toc }

Will leverage Spring Boot to connect to an in-memory database and manage the database migration as part of the code.

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Spring Data JPA

[Spring Data](https://spring.io/projects/spring-data) provides an abstraction layer for several data persistence models, such as [Spring Data JPA]().  This promotes consistency across different range of database vendors and access models as shown in the following image.

![Spring-Data-Subprojects.png]({{ '/assets/images/Spring-Data-Subprojects.png' | absolute_url }})

In our example we will use the Spring Data JPA as this allows us to interact with a relational database, such as [MySQL](https://www.mysql.com/) or [PostgreSQL](https://www.postgresql.org/s), without having to write the code that reads from and write to the database.

1. Import the [`spring-boot-starter-data-jpa`](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-data-jpa) starter dependency

   Update file: `build.gradle`

   ```groovy
   dependencies {
     /* Data */
     implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
   }
   ```

1. Build the project

   ```bash
   ./gradlew clean build

   ...
   BUILD FAILED in 9s
   6 actionable tasks: 6 executed
   ```

   The tests will fail as the `spring-boot-starter-data-jpa` is expecting a database.

## H2 database

[H2 database](https://www.h2database.com/) is a relational database management system written in Java.  It is very popular as it can be easily embedded in Java applications.  The H2 database can run in-memory, which makes it ideal for testing and prototyping.  With that said, the H2 database is not usually used in production and other databases, such as PostgreSQL, are used.

1. Add the dependency for the [H2 database](https://www.h2database.com/)

   Update file: `build.gradle`

   ```groovy
   dependencies {
     runtimeOnly 'com.h2database:h2'
   }
   ```

1. Provide the database connection properties

   {% include custom/proceed_with_caution.html details="Do not save production passwords as plain text" %}

   Note that we will refactor this code and will make use of [environment variables]({{ '/docs/data/env/' | absolute_url }}), instead.

   Update file: `src/main/resources/application.yaml`

   ```yaml
   spring:
     datasource:
       url: jdbc:h2:mem:contact-us;DATABASE_TO_UPPER=false;
       driver-class-name: org.h2.Driver
       username: tw-data
       password: SomeRandomPassword
   ```

   We are configuring the connection through the `DATABASE_TO_UPPER=false` property.  Without this, the connection may not work as expected.

1. Build the project, again

   ```bash
   $ ./gradlew clean build

   ...
   BUILD SUCCESSFUL in 9s
   6 actionable tasks: 6 executed
   ```

   This time the project should build successfully

## Flyway

The database will change over time.  One way to manage the database is to use a library such as [Flyway](https://flywaydb.org/) or [Liquibase](https://www.liquibase.org/).  ThoughtWorks [technology radar](https://www.thoughtworks.com/radar/) has [Flyway as adopt](https://www.thoughtworks.com/radar/tools/flyway).  Unfortunately, the [technology radar does not mention Liquibase](https://www.thoughtworks.com/radar/a-z#Liquibase), which means that ThoughtWorkers have not used Liquibase in production.

Both libraries share the same popularity according to [Google trends](https://trends.google.com/trends/explore?q=Flyway,Liquibase).

![Google Trends: Flyway vs. Liquibase.png]({{ '/assets/images/Google-Trends-Flyway-Liquibase.png' | absolute_url }})

1. Add the Flyway dependency

   Update file: `build.gradle`

   ```groovy
   dependencies {
     runtimeOnly 'org.flywaydb:flyway-core'
   }
   ```

   Note that in many _Spring Boot + Flyway_ documentation, this step omitted as it is said that Spring Boot configures Flyway automatically.

   ![Flyway Dependency]({{ '/assets/images/Flyway-Dependency.png' | absolute_url }})

   Spring Boot only configures Flyway if Flyway's dependency is present in the classpath.  If the library is omitted, then Flyway will not be configured and will not run.

1. Create the SQL scripts to migrate the database

   Create file: `src/main/resources/db/migration/V1__create_offices_table.sql`

   ```sql
   CREATE TABLE "offices" (
     "office"  VARCHAR(64) PRIMARY KEY,
     "address" VARCHAR(255) NOT NULL,
     "country" VARCHAR(64) NOT NULL,
     "phone"   VARCHAR(64),
     "email"   VARCHAR(64),
     "webpage" VARCHAR(128)
   );
   ```

   We can have more than one SQL statement in one file.  More information about database migration can be found [in Flyway's migration documentation](https://flywaydb.org/documentation/migrations).

1. Populate the table using a second file

   Create file: `src/main/resources/db/migration/V2__populate_offices_table.sql`

   ```sql
   INSERT INTO "offices" ("office","address","country","phone","email","webpage") VALUES ('ThoughtWorks Cologne','Lichtstr. 43i, 50825 Cologne, Germany','Germany','+49 221 64 30 70 63','contact-de@thoughtworks.com','https://www.thoughtworks.com/locations/cologne');
   INSERT INTO "offices" ("office","address","country","phone","email","webpage") VALUES ('ThoughtWorks Berlin','Zimmerstraße 23, 1. OG, 10969 Berlin, Germany','Germany','+49 (0)30 555 73 5890','contact-de@thoughtworks.com','https://www.thoughtworks.com/locations/berlin');
   INSERT INTO "offices" ("office","address","country","phone","email","webpage") VALUES ('ThoughtWorks Hamburg','Caffamacherreihe 7, 20355 Hamburg, Germany','Germany','+49 (040) 300 95 880','contact-de@thoughtworks.com','https://www.thoughtworks.com/locations/hamburg');
   INSERT INTO "offices" ("office","address","country","phone","email","webpage") VALUES ('ThoughtWorks Munich','Bothestraße 11, 81675 Munich, Germany','Germany','+49 (0)89 262 057 72','contact-de@thoughtworks.com','https://www.thoughtworks.com/locations/munich');
   INSERT INTO "offices" ("office","address","country","phone","email","webpage") VALUES ('ThoughtWorks London','76 Wardour Street, London W1F 0UR, UK','UK','+44 (0)20 3437 0990',null,'https://www.thoughtworks.com/locations/london');
   INSERT INTO "offices" ("office","address","country","phone","email","webpage") VALUES ('ThoughtWorks Manchester','4th Floor Federation House, 2 Federation St., Manchester M4 4BF, UK','UK','+44 (0)161 923 6810',null,'https://www.thoughtworks.com/locations/manchester');
   ```

   These are the same offices found in the [`offices.csv`]({{ '/assets/demo/01-primer/offices.csv' | absolute_url }}) file.

### When is Flyway triggered?

Spring Boot will run [Flyway migration](https://flywaydb.org/documentation/command/migrate) before it makes the database connection available to the rest of application.  This ensures that the database is prepared before the application makes use of it.
