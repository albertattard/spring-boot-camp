---
layout: default
title: Design of REST Controller
parent: REST
nav_order: 6
permalink: docs/rest/design/
---

# Design of REST Controller
{: .no_toc }

In most cases the REST Controller is only one part of whole application. In most cases we expect some kind of evolvability from the system. So the interaction between the Controller and other part of the system have to be carefully designed.

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}


## About clean code

The everyday job of a developer is to create a new code or to change already existing code. Most of the investigations consider the las one as the most important, it takes about 80 - 95 % of time. This means that the code have to be easy to read and easy to change. Such a code is called clean.

Some sources [John Ousterhout, CS 190 (Software Design Studio)](https://web.stanford.edu/~ouster/cgi-bin/home.php) suggest three difficulties that prevent the code to be easily understandable and changeable:
* Change amplification - a simple change requires code modifications in many places,
* Cognitive load - how much a developer needs to know in order to complete a simple change,
* Unknown unknowns - it is not obvious which pieces of code have to be modified in order to complete a simple change.

 To decrease the impact of these difficulties the come must have good organisation, the number of dependencies have to be minimal and all the important information is not obvious (for example, the name of a variable). In the next sections some discussion about the organisation of the code and about dependencies follows.

## Code organisation issues

In the example code all classes are located in the same package:

```
office
-- ControllerExceptionHandler.java  // Controller Advice
-- ContactUsService.java            // Interface to communicate with services
-- JpsContactUsService.java         // Implementation of the Interface
-- Office.java                      // Business Object
-- OfficeEntity.java                // Entity for database
-- OfficeNotFoundException.java     // Exception if an office not found in the system
-- OfficesController.java           // REST Controller
-- OfficeRepository.java            // Interface to communicate with database
```

The simplest way to organize the code is to group the classes according the functionality, for example - web part, business core part and persistence part:
```
office
-- web
  -- ControllerExceptionHandler.java  // Controller Advice
  -- OfficeNotFoundException.java     // Exception if an office not found in the system
  -- OfficesController.java           // REST Controller
-- core
  -- ContactUsService.java            // Interface to communicate with services
  -- JpsContactUsService.java         // Implementation of the Interface
  -- Office.java                      // Business Object
-- persistency
  -- OfficeEntity.java                // Entity for database
  -- OfficeRepository.java            // Interface to communicate with database
```

Such code organisation is called Layered Architecture. There are also another approaches, for example [Hexagonal Architecture](https://madewithlove.com/hexagonal-architecture-demystified/). The right choice always depends on particular case under development.


## Dependency issues

Sometimes dependencies between different part of the application will be accidentally introduced. Some cases are listed below.


### The mix of responsibilities

Some classes, for example REST controller have a lot of responsibilities - listening of the socket, serialisation and validation of the input, calling business logic and so on. We shouldn't increase this by adding some business logic into a controller, instead we will extract the business functionality into dedicated class.


### Dependency from persistence layer

Because in layered architecture the database can be considered as a base for the application, sometimes a dependency between database on upper layers, like a controller will be accidentally introduced.

![Controller-database-dependency.jpg]({{ 'assets/images/Controller-database-dependency.jpg' | absolute_url }})
For example, the cache management in REST controller will be implemented using "Last Modified" header, and the source for the header's value a Timestamp from the databased will be used. This solution introduces a dependency between REST Controller and database table Offices. Careless change of the database structure will affect the functionality of the REST Controller.

Alternative: Use a HASH function and ``` eTag ``` Header for Cache management. The calculation of the HASH can be performed by Office object itself.


### Dependency in validation

The REST Controller provide us an option to perform input validation, for example for __Path Variable__ or __Request Parameter__. Unfortunately sometimes it is possible to introduce unnecessary dependency between a controller and business logic.

For example, we will have the next line in the controller's code:

```public Office getOffice(final @PathVariable("name") @Size(min = 5) String name)```

The code ``` @Size(min = 5) ``` tell us that the length of the name have to be at least 5 characters. It seems that such a restriction is rather business rule. One day when we need to change the rule, all the controllers have to be changed.

Alternative: Controller will validate the format restrictions only - ```@NotNull```, ```@NotBlank```. The validation of business rules will be done by service. To define the exact valid values a property file will be used.


## Conclusion

The definition of the clean code contain two part - easy to understand and easy to change. How can one argue about what is easy enough?

The main consideration will be the goal of the code. For example, in one case a developer tries to implement a prototype that will be used for short time during feedback gathering from a user and then thrown away. In other case a developer works with some part of production code from payment processing system, that will be evolved in the future.

Clearly, in first case the developer will allow making less "clean" code without affecting the quality of the whole product.
