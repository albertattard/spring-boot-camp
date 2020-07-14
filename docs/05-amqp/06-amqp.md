---
layout: default
title: Notify Event Food Service (using AMQP)
description: Establish a communication channel between two microservices using AMQP
lang: en
parent: Advanced Message Queuing Protocol
nav_order: 6
permalink: docs/amqp/register/
---

# Notify Event Food Service (using AMQP)
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Scenario

The event food service application is expecting a JSON message that contains the following information.

```json
{
  "eventId": "cd078d7b-471d-4836-8958-67da359be4bf",
  "attendeeId": "8e90c18b-4e9f-414a-9e66-b6b9722c5442",
  "foodPreference": "VEGAN"
}
```

The `eventId` represents the id of the event, while the `attendeeId` represents an attendee for this event.  A person registering for two events will have two different `attendeeId` as the `attendeeId` is unique accross all events.

The food preference is limited to the following values.

```
NO_FOOD
VEGETARIAN
VEGAN
MEAT
```

The event food service will read messages sent to message queue and will process them.  The food will be prepared and packaged on the day of the event and each item will have a label with the attendee ID and a QR-Code which can be easily scanned.

![Event-Food-QR-Code-Example]({{ '/assets/images/Event-Food-QR-Code-Example.png' | absolute_url }})

## Objectives

- [ ] Send a message to the event food service using AMQP

## High-level design

So far we are able to register attendees to events.  Now we need to communicate this to the food service using message queue.  After the registration is saved, we need to create a new object that represent the JSON object expected by the event food service, as shown in the following diagram.

![Contact-Us-Push-After-Save.png]({{ '/assets/images/Contact-Us-Push-After-Save.png' | absolute_url }})

The service will not interact with the queue, but instead will use a gateway to isolate our services from the way we are integrating with the event food service.  We can switch this to REST without having to change the service.

## Service

We will start from the service and then we will implement the gateway.

1. Create the model

   Create file: `src/main/java/demo/boot/event/AttendeeFoodPreference.java`

   ```java
   package demo.boot.event;

   import lombok.AllArgsConstructor;
   import lombok.Data;
   import lombok.NoArgsConstructor;

   import java.util.UUID;

   @Data
   @NoArgsConstructor
   @AllArgsConstructor
   public class AttendeeFoodPreference {

     private UUID eventId;
     private UUID attendeeId;
     private FoodPreference foodPreference;
   }
   ```

1. Create the gateway

   Create file: `src/main/java/demo/boot/event/EventFoodGateway.java`

   ```java
   package demo.boot.event;

   public class EventFoodGateway {

     public void submit( final AttendeeFoodPreference preference ) {
     }
   }
   ```

1. Add the gateway to the service

   We need to modify the service and the tests to work with the new gateway.

   1. Update file: `src/main/java/demo/boot/event/EventRegistrationService.java`

      {% include custom/project_dose_not_compile.html %}

      ```java
      package demo.boot.event;

      import lombok.AllArgsConstructor;

      import java.time.LocalDate;
      import java.util.Optional;

      @AllArgsConstructor
      public class EventRegistrationService {

        private final EventRepository repository;
        private final UuidGeneratorService uuidGeneratorService;
      /**/private final EventFoodGateway gateway;

        public Optional<RegistrationConfirmation> register( final RegistrationDetails registration ) { /* ... */ }
      }
      ```

   1. Update file: `src/test/java/demo/boot/event/EventRegistrationServiceTest.java`

```java
```
