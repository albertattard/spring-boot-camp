---
layout: default
title: Notify Event Food Service (using AMQP)
description: Establish a communication channel between two microservices using AMQP
lang: en
parent: Advanced Message Queuing Protocol
nav_order: 6
permalink: docs/amqp/amqp/
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

### Should we modify the service or use a decorator?

The notification process is an integrated part of our registration service.  Therefore, we can have this as part of the service.  Creating a decorator that just handles this is not a bad idea and it is quite common to see this implementation.

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
      package demo.boot.event;

      import org.junit.jupiter.api.DisplayName;
      import org.junit.jupiter.api.Test;

      import java.time.LocalDate;
      import java.util.Optional;
      import java.util.UUID;

      import static org.junit.jupiter.api.Assertions.assertEquals;
      import static org.mockito.ArgumentMatchers.eq;
      import static org.mockito.Mockito.doNothing;
      import static org.mockito.Mockito.mock;
      import static org.mockito.Mockito.times;
      import static org.mockito.Mockito.verify;
      import static org.mockito.Mockito.verifyNoMoreInteractions;
      import static org.mockito.Mockito.when;

      @DisplayName( "Event registration service" )
      public class EventRegistrationServiceTest {

        @Test
        @DisplayName( "should return Optional empty when registering to an non existing event" )
        public void shouldReturnOptionalEmptyWhenNotFound() {
          final EventRepository eventRepository = mock( EventRepository.class );
          final UuidGeneratorService uuidGeneratorService = mock( UuidGeneratorService.class );
      /**/final EventFoodGateway gateway = mock( EventFoodGateway.class );

          final UUID eventId = UUID.randomUUID();
          final String name = "Aden Attard";
          final FoodPreference foodPreference = FoodPreference.MEAT;
          final RegistrationDetails details = new RegistrationDetails( eventId, name, foodPreference );

          when( eventRepository.findById( eq( eventId ) ) ).thenReturn( Optional.empty() );

      /**/final EventRegistrationService service = new EventRegistrationService( eventRepository, uuidGeneratorService, gateway );
          final Optional<RegistrationConfirmation> confirmation = service.register( details );
          assertEquals( Optional.empty(), confirmation );

          verify( eventRepository, times( 1 ) ).findById( eventId );
      /**/verifyNoMoreInteractions( eventRepository, uuidGeneratorService, gateway );
        }

        @Test
        @DisplayName( "should return Optional empty when registering to an expired event" )
        public void shouldReturnOptionalEmptyWhenExpired() {
          final EventRepository eventRepository = mock( EventRepository.class );
          final EventEntity eventEntity = mock( EventEntity.class );
          final UuidGeneratorService uuidGeneratorService = mock( UuidGeneratorService.class );
      /**/final EventFoodGateway gateway = mock( EventFoodGateway.class );

          final UUID eventId = UUID.randomUUID();
          final String name = "Jade Attard";
          final FoodPreference foodPreference = FoodPreference.VEGETARIAN;
          final RegistrationDetails details = new RegistrationDetails( eventId, name, foodPreference );

          when( eventRepository.findById( eq( eventId ) ) ).thenReturn( Optional.of( eventEntity ) );
          when( eventEntity.getDate() ).thenReturn( LocalDate.now().minusDays( 1 ) );

      /**/final EventRegistrationService service = new EventRegistrationService( eventRepository, uuidGeneratorService, gateway );
          final Optional<RegistrationConfirmation> confirmation = service.register( details );
          assertEquals( Optional.empty(), confirmation );

          verify( eventRepository, times( 1 ) ).findById( eventId );
          verify( eventEntity, times( 1 ) ).getDate();
      /**/verifyNoMoreInteractions( eventRepository, eventEntity, uuidGeneratorService, gateway );
        }

        @Test
        @DisplayName( "should return the registration confirmation when registering to an active event" )
        public void shouldReturnConfirmationWhenActive() {
          final EventRepository eventRepository = mock( EventRepository.class );
          final EventEntity eventEntity = mock( EventEntity.class );
          final UuidGeneratorService uuidGeneratorService = mock( UuidGeneratorService.class );
      /**/final EventFoodGateway gateway = mock( EventFoodGateway.class );

          final UUID eventId = UUID.randomUUID();
          final UUID attendeeId = UUID.randomUUID();
          final String name = "Aden Attard";
          final FoodPreference foodPreference = FoodPreference.VEGAN;
          final RegistrationDetails details = new RegistrationDetails( eventId, name, foodPreference );
          final EventAttendeeEntity attendeeEntity = new EventAttendeeEntity( attendeeId, name, foodPreference, eventEntity );

          when( eventRepository.findById( eq( eventId ) ) ).thenReturn( Optional.of( eventEntity ) );
          when( eventEntity.getDate() ).thenReturn( LocalDate.now().plusDays( 1 ) );
          when( uuidGeneratorService.nextAttendeeId() ).thenReturn( attendeeId );
          doNothing().when( eventEntity ).addAttendee( attendeeEntity );
          when( eventRepository.save( eq( eventEntity ) ) ).thenReturn( eventEntity );

      /**/final EventRegistrationService service = new EventRegistrationService( eventRepository, uuidGeneratorService, gateway );
          final Optional<RegistrationConfirmation> confirmation = service.register( details );
          assertEquals( Optional.of( new RegistrationConfirmation( attendeeId ) ), confirmation );

          verify( eventRepository, times( 1 ) ).findById( eventId );
          verify( eventEntity, times( 1 ) ).getDate();
          verify( uuidGeneratorService, times( 1 ) ).nextAttendeeId();
          verify( eventEntity, times( 1 ) ).addAttendee( attendeeEntity );
          verify( eventRepository, times( 1 ) ).save( eventEntity );
      /**/verifyNoMoreInteractions( eventRepository, eventEntity, uuidGeneratorService, gateway );
        }
      }
      ```

   Run the above test.

   ```bash
   $ ./gradlew clean test "--tests" "*EventRegistrationServiceTest"

   ...

   Event registration service > should return Optional empty when registering to an non existing event PASSED

   Event registration service > should return the registration confirmation when registering to an active event PASSED

   Event registration service > should return Optional empty when registering to an expired event PASSED

   BUILD SUCCESSFUL in 4s
   5 actionable tasks: 5 executed
   ```

   All three tests should pass.

1. Update the `shouldReturnConfirmationWhenActive()` test

   The food service needs to be sent a message when an attendee registers for an event.  This test does not capture this yet.

   Update file: `src/test/java/demo/boot/event/EventRegistrationServiceTest.java`

   ```java
   package demo.boot.event;

   import org.junit.jupiter.api.DisplayName;
   import org.junit.jupiter.api.Test;

   import java.time.LocalDate;
   import java.util.Optional;
   import java.util.UUID;

   import static org.junit.jupiter.api.Assertions.assertEquals;
   import static org.mockito.ArgumentMatchers.eq;
   import static org.mockito.Mockito.doNothing;
   import static org.mockito.Mockito.mock;
   import static org.mockito.Mockito.times;
   import static org.mockito.Mockito.verify;
   import static org.mockito.Mockito.verifyNoMoreInteractions;
   import static org.mockito.Mockito.when;

   @DisplayName( "Event registration service" )
   public class EventRegistrationServiceTest {

     @Test
     @DisplayName( "should return Optional empty when registering to an non existing event" )
     public void shouldReturnOptionalEmptyWhenNotFound() { /* ... */ }

     @Test
     @DisplayName( "should return Optional empty when registering to an expired event" )
     public void shouldReturnOptionalEmptyWhenExpired() { /* ... */ }

     @Test
     @DisplayName( "should return the registration confirmation when registering to an active event" )
     public void shouldReturnConfirmationWhenActive() {
       final EventRepository eventRepository = mock( EventRepository.class );
       final EventEntity eventEntity = mock( EventEntity.class );
       final UuidGeneratorService uuidGeneratorService = mock( UuidGeneratorService.class );
       final EventFoodGateway gateway = mock( EventFoodGateway.class );

       final UUID eventId = UUID.randomUUID();
       final UUID attendeeId = UUID.randomUUID();
       final String name = "Aden Attard";
       final FoodPreference foodPreference = FoodPreference.VEGAN;
       final RegistrationDetails details = new RegistrationDetails( eventId, name, foodPreference );
       final EventAttendeeEntity attendeeEntity = new EventAttendeeEntity( attendeeId, name, foodPreference, eventEntity );
   /**/final AttendeeFoodPreference attendeeFoodPreference = new AttendeeFoodPreference( eventId, attendeeId, foodPreference );

       when( eventRepository.findById( eq( eventId ) ) ).thenReturn( Optional.of( eventEntity ) );
       when( eventEntity.getDate() ).thenReturn( LocalDate.now().plusDays( 1 ) );
       when( uuidGeneratorService.nextAttendeeId() ).thenReturn( attendeeId );
       doNothing().when( eventEntity ).addAttendee( attendeeEntity );
       when( eventRepository.save( eq( eventEntity ) ) ).thenReturn( eventEntity );
   /**/doNothing().when( gateway ).submit( eq( attendeeFoodPreference ) );

       final EventRegistrationService service = new EventRegistrationService( eventRepository, uuidGeneratorService, gateway );
       final Optional<RegistrationConfirmation> confirmation = service.register( details );
       assertEquals( Optional.of( new RegistrationConfirmation( attendeeId ) ), confirmation );

       verify( eventRepository, times( 1 ) ).findById( eventId );
       verify( eventEntity, times( 1 ) ).getDate();
       verify( uuidGeneratorService, times( 1 ) ).nextAttendeeId();
       verify( eventEntity, times( 1 ) ).addAttendee( attendeeEntity );
       verify( eventRepository, times( 1 ) ).save( eventEntity );
   /**/verify( gateway, times( 1 ) ).submit( attendeeFoodPreference );
       verifyNoMoreInteractions( eventRepository, eventEntity, uuidGeneratorService, gateway );
     }
   }
   ```

   Run the test.

   ```bash
   $ ./gradlew clean test "--tests" "*EventRegistrationServiceTest"

   ...

   Event registration service > should return the registration confirmation when registering to an active event FAILED
       org.mockito.exceptions.verification.WantedButNotInvoked at EventRegistrationServiceTest.java:101

   ...

   BUILD FAILED in 6s
   5 actionable tasks: 5 executed
   ```

   The test should now fail as we are not interacting with the gateway as expected.

1. Make the test pass.

   Update file: `src/main/java/demo/boot/event/EventRegistrationService.java`

   ```java
   ```
