---
layout: default
title: Reorganise Classes
description: Reorganise classes into packages to simplify the inclusion of the new feature
lang: en
parent: Advanced Message Queuing Protocol
nav_order: 4
permalink: docs/amqp/refactor/
---

# Reorganise classes
{: .no_toc }

Up to now, all classes are placed into one package, `demo.boot`.  This worked well as we only had one feature, that is, managing offices.  Now we will introduce a new feature, managing events.  While we can keep all class in one package, that can become hard to navigate.

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Reorganise classes

1. Create new `office` package

   ```bash
   $ mkdir src/main/java/demo/boot/office
   $ mkdir src/test/java/demo/boot/office
   $ mkdir src/test-integration/java/demo/boot/office
   ```

1. Move the _main_ classes to the `office` package

   {% include custom/note.html details="The class <code>ContactUsApplication</code> is not moved to the <code>office</code> package as this is not related to offices, or any other feature." %}

   ```
   ContactUsService
   JpaContactUsService
   Office
   OfficeController
   OfficeCountMetricDecorator
   OfficeEntity
   OfficesRepository
   ```

   The `src/main/java` should you look like the following

   ```bash
   $ tree src/main/java
   src/main/java
   └── demo
       └── boot
           ├── ContactUsApplication.java
           └── office
               ├── ContactUsService.java
               ├── JpaContactUsService.java
               ├── Office.java
               ├── OfficeController.java
               ├── OfficeCountMetricDecorator.java
               ├── OfficeEntity.java
               └── OfficesRepository.java
   ```

1. Move the _test_ classes to the `office` package

   ```
   JpaContactUsServiceTest
   OfficeControllerTest
   OfficeCountMetricDecoratorTest
   ```

   The `src/test/java` should you look like the following

   ```bash
   $ tree src/test/java
   src/test/java
   └── demo
       └── boot
           └── office
               ├── JpaContactUsServiceTest.java
               ├── OfficeControllerTest.java
               └── OfficeCountMetricDecoratorTest.java
   ```

1. Move the _test-integration_ classes to the `office` package

   {% include custom/note.html details="The class <code>ContactUsApplicationTests</code> is not moved to the <code>office</code> package as this is not related to offices, or any other feature." %}

   ```
   JpaContactUsServiceDeleteWhileUpdateTest
   JpaContactUsServiceMultiDeleteTest
   OfficesRepositoryTest
   ```

   The `src/test-integration/java` should you look like the following

   ```bash
   $ tree src/test-integration/java
   src/test-integration/java
   └── demo
       └── boot
           └── office
               ├── ContactUsApplicationTests.java
               ├── JpaContactUsServiceDeleteWhileUpdateTest.java
               ├── JpaContactUsServiceMultiDeleteTest.java
               └── OfficesRepositoryTest.java
   ```

1. Remove unnecessary imports

   While moving classes around, unnecessary imports may have been added to some of the classes by the IDE.  Right click on the _src_ folder and then select _Optimize Imports_

   ![Contact-Us-Organise-Imports.png]({{ '/assets/images/Contact-Us-Organise-Imports.png' | absolute_url }})

1. Build the project

   ```bash
   $ ./gradlew clean build

   ...

   BUILD SUCCESSFUL in 36s
   9 actionable tasks: 9 executed
   ```

   All tests should pass.
