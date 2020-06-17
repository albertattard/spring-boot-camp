---
layout: default
title: Environment Variables
parent: Spring Data
nav_order: 4
permalink: docs/data/env/
---

# Environment Variables
{: .no_toc }

In this section we will switch from hardcoded properties to use of [environment variables](https://en.wikipedia.org/wiki/Environment_variable).

Environment variables are quite popular means to share configuration details, such as credentials.  While this is a very common approach, it is not the safest approach and technologies, such as [Vault](https://www.vaultproject.io/), are preferred.

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## The `.env` file

Say that my application requires several environment variables to run.  When building the application we need to provide these environment variables to gradle so that it can interact with the test database, for example, as shown next.

```bash
$ DATABASE_NAME=contact-us \
  DATABASE_PORT= \
  DATABASE_URL=jdbc:h2:mem:contact-us;DATABASE_TO_UPPER=false; \
  DATABASE_DRIVER=org.h2.Driver \
  DATABASE_USERNAME=tw-data \
  DATABASE_PASSWORD=SomeRandomPassword \
  ./gradlew clean build
```

This can get a bit ugly especially when we have many environment variables.  Furthermore, if we like to trigger the gradle `build` task from IntelliJ, we need to pass these variables there too.

Another approach is to use an `.env` file as shown next.

{% include proceed_with_caution.html details="Do not save production passwords as plain text" %}

```properties
DATABASE_NAME=contact-us
DATABASE_PORT=
DATABASE_URL=jdbc:h2:mem:contact-us;DATABASE_TO_UPPER=false;
DATABASE_DRIVER=org.h2.Driver
DATABASE_USERNAME=tw-data
DATABASE_PASSWORD=SomeRandomPassword
```

Note that the above example only contains development and test credentials.  While that may seem fine, given that we are working with a dev environment, there may be cases where this may have serious implications.  There are cases where the development and test environments work with sensitive information.  In this case, we always need to secure the credentials to such resources (such as databases) and make sure to destroy such resources when we are done with them.

1. Create the `.env` file

   Create file `.env`

   ```properties
   DATABASE_NAME=contact-us
   DATABASE_PORT=
   DATABASE_URL=jdbc:h2:mem:contact-us;DATABASE_TO_UPPER=false;
   DATABASE_DRIVER=org.h2.Driver
   DATABASE_USERNAME=tw-data
   DATABASE_PASSWORD=SomeRandomPassword
   ```

   Note that the H2 in-memory database does not require a port number, yet we defined the `DATABASE_PORT` environment variable, as this will be used later on when we connect to a production grade database.

1. Update the properties

   Update file `src/main/resources/application.yaml`

   ```yaml
   spring:
     datasource:
       url: ${DATABASE_URL}
       driver-class-name: ${DATABASE_DRIVER}
       username: ${DATABASE_USERNAME}
       password: ${DATABASE_PASSWORD}
   ```

   Note that all hardcoded values are now replaced by an environment variable.  Spring will replace `${ENV_NAME}` with the value found in the environment variables.

   Following is the complete example of the `application.yaml` file.

   ```yaml
   spring:
     datasource:
       url: ${DATABASE_URL}
       driver-class-name: ${DATABASE_DRIVER}
       username: ${DATABASE_USERNAME}
       password: ${DATABASE_PASSWORD}

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

1. Build the project

   ```bash
   $ ./gradlew clean build

   ...
   BUILD FAILED in 9s
   6 actionable tasks: 6 executed
   ```

   Our application will not build successfully as gradle is not using these environment variables as yet.

## Integration tests

When we moved the configuration of the database to environment variables, we made our application dependent on external resources, such as the environment variable.  Any tests that depend on things outside of our application should be moved into their own category.

The test class `ContactUsApplicationTests` requires the application to start, which in turn require the environment variables defined before.  Gradle is very customisable and we can easily have a new set of tests, referred to as _integration tests_ and move any tests into this group.

1. Define integration tests

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

   The name of the file is not very important, as long as you are consistent.  Note that we will use this file name to include the above gradle definition in the `build.gradle` in the next step.

   The above file deserves some explanation.

   1. Create new configuration

      ```groovy
      configurations {
        integrationTestImplementation.extendsFrom testImplementation
        integrationTestRuntimeOnly.extendsFrom testRuntimeOnly
      }
      ```

      We can have dependencies specific to only integration tests, which extend the test scope.

   1. Define the new source set

      ```groovy
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
      ```

      This is where the integration test files will be saved.

   1. Define the new gradle task

      ```groovy
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
      ```

      The environment variables are read from the `.env` file and added the task's environment.

      ```groovy
        doFirst {
          file("$rootDir/.env").readLines().each() {
            def (key, value) = it.split('=', 2)
            environment key, value
          }
        }
      ```

      When the integration test run, the environment variable defined in the file are loaded and made available to the integration tests.

   1. Make the integration tests part of the check flow

      ```groovy
      check {
        dependsOn integrationTest
      }
      ```

      The `check` task, depends on our `integrationTest` task as shown in the following task tree.

      ```bash
      $ ./gradlew build taskTree

      > Task :taskTree

      ------------------------------------------------------------
      Root project
      ------------------------------------------------------------

      :build
      +--- :assemble
      |    +--- :bootJar
      |    |    \--- :classes
      |    |         +--- :compileJava
      |    |         \--- :processResources
      |    \--- :jar
      |         \--- :classes
      |              +--- :compileJava
      |              \--- :processResources
      \--- :check
           +--- :integrationTest
           |    +--- :classes
           |    |    +--- :compileJava
           |    |    \--- :processResources
           |    +--- :integrationTestClasses
           |    |    +--- :compileIntegrationTestJava
           |    |    |    +--- :classes
           |    |    |    |    +--- :compileJava
           |    |    |    |    \--- :processResources
           |    |    |    \--- :testClasses
           |    |    |         +--- :compileTestJava
           |    |    |         |    \--- :classes
           |    |    |         |         +--- :compileJava
           |    |    |         |         \--- :processResources
           |    |    |         \--- :processTestResources
           |    |    \--- :processIntegrationTestResources
           |    \--- :testClasses
           |         +--- :compileTestJava
           |         |    \--- :classes
           |         |         +--- :compileJava
           |         |         \--- :processResources
           |         \--- :processTestResources
           \--- :test
                +--- :classes
                |    +--- :compileJava
                |    \--- :processResources
                \--- :testClasses
                     +--- :compileTestJava
                     |    \--- :classes
                     |         +--- :compileJava
                     |         \--- :processResources
                     \--- :processTestResources
      ```

   **Why are we creating a second gradle build file?**

   We can simply have all gradle configuration in one file, the `gradle.build` file.  Splitting the definition into multiple files helps organising gradle, keeping each file focused on one thing.  This is a matter of taste too as some like to split gradle into smaller files, while others prefer to have everything in one place.

1. Import the new definition

   Update file `build.gradle`

   ```groovy
   /* Integration Tests */
   apply from: "$rootDir/integration-test.gradle"
   ```

   The above fragment uses the [script plugins](https://docs.gradle.org/current/userguide/plugins.html#sec:script_plugins) to include the gradle configuration from another file.  The file name needs to match the file name used in the previous step.

1. Create the new directory structure

   ```bash
   $ mkdir -p src/test-integration/java
   $ mkdir -p src/test-integration/resources
   ```

   The new `test-integration` should have the following sub-directories

   ```bash
   $ tree src/test-integration
   src/test-integration
   ├── java
   └── resources
   ```

   When you apply the gradle change to IntelliJ, the new folder structure should appear as a source folcer as shown next.

![Test Integration Folder Structure]({{ '/assets/images/Test-Integration-Folder-Structure.png' | absolute_url }})

   The folders _main_, _test_ and _test-integration_ have a small blue square indicating that these are source folders.

1. The new `integrationTest` gradle task

   ```groovy
   task integrationTest(type: Test) { /* ... */ }
   ```

   List the gradle tasks

   ```bash
   $ ./gradlew tasks
   ```

   This will list all gradle tasks available for this project.  Our new gradle task is part of the `verification` group and this it will be part of this group as shown next.

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

1. Move the `ContactUsApplicationTests` to the `test-intergration`, and retain the package structure

   ```bash
   $ mkdir -p src/test-integration/java/demo/boot
   $ contact-us mv src/test/java/demo/boot/ContactUsApplicationTests.java src/test-integration/java/demo/boot
   ```

   The other two tests should stay where these were, as shown in the following directory structure.

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

  /* OpenApi/Swagger */
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
