---
layout: default
title: Environment Variables
parent: Spring Data
nav_order: 4
permalink: docs/data/env/
---

# Environment Variables
{: .no_toc }

In this section we will switch from hardcoded properties to the use of [environment variables](https://en.wikipedia.org/wiki/Environment_variable).

Environment variables are quite popular means to share configuration details, such as credentials.  While this is a very common approach, it is not the safest approach and technologies, such as [Vault](https://www.vaultproject.io/), are preferred.

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## The `.env` file

Say that the application requires several environment variables to run.  When building the application, we need to provide these environment variables to Gradle so that it can interact with the test database, for example, as shown next.

```bash
$ DATABASE_NAME=contact-us \
  DATABASE_PORT= \
  DATABASE_URL=jdbc:h2:mem:contact-us;DATABASE_TO_UPPER=false; \
  DATABASE_DRIVER=org.h2.Driver \
  DATABASE_USERNAME=tw-data \
  DATABASE_PASSWORD=SomeRandomPassword \
  ./gradlew clean build
```

This can get a bit ugly especially when we have many environment variables.  Furthermore, if we like to trigger the Gradle `build` task from IntelliJ, we need to pass these variables there too.

Another approach is to use an `.env` file as shown next.

{% include custom/proceed_with_caution.html details="Do not save production passwords as plain text" %}

```properties
DATABASE_NAME=contact-us
DATABASE_PORT=
DATABASE_URL=jdbc:h2:mem:contact-us;DATABASE_TO_UPPER=false;
DATABASE_DRIVER=org.h2.Driver
DATABASE_USERNAME=tw-data
DATABASE_PASSWORD=SomeRandomPassword
```

The above example only contains development and test credentials.  While that may seem fine, given that we are working with a development environment, there may be cases where this may have serious implications.  There are cases where the development and test environments work with sensitive data.  In such cases, we always need to secure the credentials to such resources (such as files and databases) and make sure to destroy such resources when we are done with them.

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

   {% include custom/note.html details="The <a href='https://www.h2database.com/'>H2</a> in-memory database does not require a port number, yet we defined the <code>DATABASE_PORT</code> environment variable.  This will be used later on when we connect to a production grade database." %}

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

   All hardcoded values are now replaced by an environment variable.  Spring will replace `${ENV_NAME}` with the value found in the environment variables automatically allowing our application to behave differently based on where this is running.  In a production environment, our application can connect to the production database, while when in development our application will connect to the development database.

   Following is the complete example of the `application.yaml` properties file.

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

   Our application will not build successfully as Gradle is not using these environment variables as of yet.

{% include custom/note.html details="The <code>.env</code> should not be deployed with the application and is only intended to be used as part of the source to aid development." %}

## Integration tests

When we moved the configuration of the database to environment variables, we made our application dependent on external resources, such as the environment variables.  Any tests that depend on resources sitting outside of our application should be moved into their own category (namely _integration tests_).

The test class `ContactUsApplicationTests` requires the application to start, which in turn require the environment variables defined before.  Gradle is very customisable and we can easily have a new set of tests, referred to as _integration tests_, and move the tests that depend on external resources into this group.

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

   The name of the file (`integration-test.gradle`) is not very important, as long as you are consistent.  Note that we will use this file name (`integration-test.gradle`) to include the above Gradle definition in the `build.gradle` in the next step.

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

   1. Define the new Gradle task

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

      The environment variables are read from the `.env` file at the root of the project, and added to the task's environment.

      ```groovy
        doFirst {
          file("$rootDir/.env").readLines().each() {
            def (key, value) = it.split('=', 2)
            environment key, value
          }
        }
      ```

      When the integration test run, the environment variable defined in the `.env` file are loaded and made available to the integration tests.  This is quite convenient as the environment variables are centralised in one place.

   1. Make the integration tests part of the check flow

      ```groovy
      check {
        dependsOn integrationTest
      }
      ```

      The `build` task depends on the `check` task, which in turn it depends on our `integrationTest` task as shown in the following task tree.

      ```bash
      $ ./gradlew build taskTree

      ...

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

      Note that our `integrationTest` does not depend on the `test` task as we can run each of these Gradle tasks independent.   With that said, the integration test code, depends on the test code and we can reuse classes that are defined in the tests.

   **Why are we creating a second Gradle build file?**

   We can simply have all Gradle configuration in one file, the `gradle.build` file.  Splitting the definition into multiple files helps organising Gradle, keeping each file focused on one thing.  This is a matter of taste too as some like to split Gradle into smaller files, while others prefer to have everything in one place.

1. Import the new definition

   Update file `build.gradle`

   ```groovy
   /* Integration Tests */
   apply from: "$rootDir/integration-test.gradle"
   ```

   The above fragment uses the [script plugins](https://docs.gradle.org/current/userguide/plugins.html#sec:script_plugins) to include the Gradle configuration from another file.  The file name (`integration-test.gradle`) needs to match the file name used in the previous step.

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

   When you apply the Gradle change to IntelliJ.  The new folder structure should appear as a source folder as shown next.

   ![Test Integration Folder Structure]({{ '/assets/images/Test-Integration-Folder-Structure.png' | absolute_url }})

   The folders _main_, _test_ and _test-integration_ have a small blue square indicating that these are source folders.

1. The new `integrationTest` Gradle task

   ```groovy
   task integrationTest(type: Test) { /* ... */ }
   ```

   List the Gradle tasks

   ```bash
   $ ./gradlew tasks
   ```

   This will list all Gradle tasks available for this project.  Our new Gradle task (`integrationTest`) is part of the `verification` group together with `test` and `pitest`, as shown next.

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
   $ mv src/test/java/demo/boot/ContactUsApplicationTests.java src/test-integration/java/demo/boot
   ```

   The other two tests, the `JpaContactUsServiceTest.java` and `OfficeControllerTest.java`, should stay where these were, as shown in the following directory structure.

   ```bash
   tree src
   src
   ├── main
   ...
   ├── test
   │   └── java
   │       └── demo
   │           └── boot
   │               ├── JpaContactUsServiceTest.java
   │               └── OfficeControllerTest.java
   └── test-integration
       └── java
           └── demo
               └── boot
                   └── ContactUsApplicationTests.java

   16 directories, 15 files
   ```

1. Run the integration test

   ```bash
   $ ./gradlew integrationTest

   ...
   BUILD SUCCESSFUL in 8s
   5 actionable tasks: 1 executed, 4 up-to-date
   ```

   The integration tests are now using the environment variables defined in the `.env` file.

## Gradle `bootRun` task

1. Start the application using Gradle `bootRun` task

   ```bash
   $ ./gradlew bootRun

   ...
   BUILD FAILED in 4s
   3 actionable tasks: 1 executed, 2 up-to-date
   ```

   The integration tests are using the `.env` file, but this only applies to this task

1. Use the `.env` file from the `bootRun` task

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

   {% include custom/note.html details="I do not know how to reuse the above code fragment" %}

1. Start the application using Gradle `bootRun` task again

   ```bash
   $ ./gradlew bootRun
   ...
   2077-04-27 12:34:56.464  INFO 56351 --- [           main] o.s.b.a.e.web.EndpointLinksResolver      : Exposing 2 endpoint(s) beneath base path ''
   2077-04-27 12:34:56.496  INFO 56351 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
   2077-04-27 12:34:56.497  INFO 56351 --- [           main] DeferredRepositoryInitializationListener : Triggering deferred initialization of Spring Data repositories…
   2077-04-27 12:34:56.535  INFO 56351 --- [           main] DeferredRepositoryInitializationListener : Spring Data repositories initialized!
   2077-04-27 12:34:56.547  INFO 56351 --- [           main] demo.boot.ContactUsApplication           : Started ContactUsApplication in 3.597 seconds (JVM running for 4.951)
   ```

   The application should now start and connect to the H2 database using the environment variables defined in the `.env` file.

## IntelliJ

So far we have configured Gradle to use the `.env` file defined before.

1. Run the application from IntelliJ

   ![Run Application from IntelliJ]({{ '/assets/images/IntelliJ-Run-Application.png' | absolute_url }})

   This will fail if the environment variables are missing as shown next.

   ```bash
   ...
   Caused by: java.lang.IllegalStateException: Cannot load driver class: ${DATABASE_DRIVER}
       at org.springframework.util.Assert.state(Assert.java:94) ~[spring-core-5.2.6.RELEASE.jar:5.2.6.RELEASE]
       at org.springframework.boot.autoconfigure.jdbc.DataSourceProperties.determineDriverClassName(DataSourceProperties.java:223) ~[spring-boot-autoconfigure-2.3.0.RELEASE.jar:2.3.0.RELEASE]
       at org.springframework.boot.autoconfigure.jdbc.DataSourceProperties.initializeDataSourceBuilder(DataSourceProperties.java:175) ~[spring-boot-autoconfigure-2.3.0.RELEASE.jar:2.3.0.RELEASE]
       at org.springframework.boot.autoconfigure.jdbc.DataSourceConfiguration.createDataSource(DataSourceConfiguration.java:43) ~[spring-boot-autoconfigure-2.3.0.RELEASE.jar:2.3.0.RELEASE]
       at org.springframework.boot.autoconfigure.jdbc.DataSourceConfiguration$Hikari.dataSource(DataSourceConfiguration.java:85) ~[spring-boot-autoconfigure-2.3.0.RELEASE.jar:2.3.0.RELEASE]
       at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke0(Native Method) ~[na:na]
       at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62) ~[na:na]
       at java.base/jdk.internal.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43) ~[na:na]
       at java.base/java.lang.reflect.Method.invoke(Method.java:564) ~[na:na]
       at org.springframework.beans.factory.support.SimpleInstantiationStrategy.instantiate(SimpleInstantiationStrategy.java:154) ~[spring-beans-5.2.6.RELEASE.jar:5.2.6.RELEASE]
       ... 56 common frames omitted
   ...
   ```

1. Open the run configuration

   ![IntelliJ Edit Configuration]({{ '/assets/images/IntelliJ-Edit-Configuration.png' | absolute_url }})

   Import the `env` file

   1. Select the _EnvFile_ tab
   1. Check the _Enable EnvFile_ checkbox
   1. Click the _plus_ icon to select the `.env` file

   ![IntelliJ Env File]({{ '/assets/images/IntelliJ-Env-File.png' | absolute_url }})

   Note that the `.env` file is a hidden file and it may not show in the finder.  Press `[command] + [shift] + [.]` to toggle show/hide hidden files.

1. Run the application again

   ```bash
   ...
   2077-04-27 12:34:56.413  INFO 54324 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
   2077-04-27 12:34:56.413  INFO 54324 --- [           main] DeferredRepositoryInitializationListener : Triggering deferred initialization of Spring Data repositories…
   2077-04-27 12:34:56.450  INFO 54324 --- [           main] DeferredRepositoryInitializationListener : Spring Data repositories initialized!
   2077-04-27 12:34:56.463  INFO 54324 --- [           main] demo.boot.ContactUsApplication           : Started ContactUsApplication in 4.834 seconds (JVM running for 5.82)
   2077-04-27 12:34:56.684  INFO 54324 --- [-192.168.178.35] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring DispatcherServlet 'dispatcherServlet'
   2077-04-27 12:34:56.684  INFO 54324 --- [-192.168.178.35] o.s.web.servlet.DispatcherServlet        : Initializing Servlet 'dispatcherServlet'
   2077-04-27 12:34:56.692  INFO 54324 --- [-192.168.178.35] o.s.web.servlet.DispatcherServlet        : Completed initialization in 7 ms
   ```

The same `.env` file is used by Gradle and IntelliJ.  This centralised the usage of the environment variables.
