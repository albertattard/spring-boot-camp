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

The `eventId` represents the id of the event, while the `attendeeId` represents an attendee for this event.  A person registering for two events will have two different `attendeeId` as the `attendeeId` is unique across all events.  The food service is not responsible from creating these ids and the caller should care for their uniqueness.

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

So far, we are able to register attendees to events.  Now we need to communicate this to the food service using message queue.  After the registration is saved, we need to create a new object that represent the JSON object expected by the event food service, as shown in the following diagram.

![Contact-Us-Push-After-Save.png]({{ '/assets/images/Contact-Us-Push-After-Save.png' | absolute_url }})

The service will not interact with the queue directly, but instead will use a [gateway](https://martinfowler.com/eaaCatalog/gateway.html) to isolate our services from the way we are integrating with the event food service.  We can switch this to REST without having to change the service.

### Should we modify the service or use a decorator?

The notification process is an integral part of our registration service.  Therefore, we can have this as part of the service.  [In another example, when we handled metrics, we preferred a decorator for a different reason]({{ '/docs/actuator/metrics/#can-we-create-custom-metrics' | absolute_url }}).

Creating a decorator that just handles this is not a bad idea and it is quite common to see this implementation.  In this example we will modify the service as this is part of our flow.

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
   package demo.boot.event;

   import lombok.AllArgsConstructor;
   import org.springframework.stereotype.Service;

   import java.time.LocalDate;
   import java.util.Optional;
   import java.util.function.Function;

   @Service
   @AllArgsConstructor
   public class EventRegistrationService {

     private final EventRepository repository;
     private final UuidGeneratorService uuidGeneratorService;
     private final EventFoodGateway gateway;

     public Optional<RegistrationConfirmation> register( final RegistrationDetails registration ) {
       return repository
         .findById( registration.getEventId() )
         .filter( event -> LocalDate.now().isBefore( event.getDate() ) )
         .map( registerAttendee( registration ) )
   /**/  .map( attendee -> {
   /**/    final AttendeeFoodPreference preference = new AttendeeFoodPreference();
   /**/    preference.setEventId( attendee.getEvent().getId() );
   /**/    preference.setAttendeeId( attendee.getId() );
   /**/    preference.setFoodPreference( attendee.getFoodPreference() );
   /**/    gateway.submit( preference );
   /**/    return attendee;
   /**/  } )
         .map( attendee -> new RegistrationConfirmation( attendee.getId() ) )
         ;
     }

     private Function<EventEntity, EventAttendeeEntity> registerAttendee( final RegistrationDetails registration ) { /* ... */ }
   }
   ```

   Run the test.

   ```bash
   $ ./gradlew clean test "--tests" "*EventRegistrationServiceTest"

   ...

   Event registration service > should return the registration confirmation when registering to an active event FAILED
       org.mockito.exceptions.verification.opentest4j.ArgumentsAreDifferent at EventRegistrationServiceTest.java:101

   ...

   BUILD FAILED in 5s
   5 actionable tasks: 5 executed
   ```

   Against some expectations, the test fails.  If we analyse the error, we will notice that the `AttendeeFoodPreference` instance we are passing to the gateway does not match the expected one.

   ```bash
   Argument(s) are different! Wanted:
   eventFoodGateway.submit(
       AttendeeFoodPreference(eventId=5bf802ca-d52d-4ae4-b7df-616b5701c615, attendeeId=90b93962-0c39-45fe-863f-938c7c0a2e09, foodPreference=VEGAN)
   );
   -> at demo.boot.event.EventRegistrationServiceTest.shouldReturnConfirmationWhenActive(EventRegistrationServiceTest.java:101)
   Actual invocations have different arguments:
   eventFoodGateway.submit(
       AttendeeFoodPreference(eventId=null, attendeeId=90b93962-0c39-45fe-863f-938c7c0a2e09, foodPreference=VEGAN)
   );
   -> at demo.boot.event.EventRegistrationService.lambda$register$1(EventRegistrationService.java:28)
   ```

   The one we are creating does not contain a UUID.

   ```bash
       AttendeeFoodPreference(eventId=null, attendeeId=90b93962-0c39-45fe-863f-938c7c0a2e09, foodPreference=VEGAN)
   ```

   In our mocking setup, we are not setting up the `eventEntity` to return the id.

1. Fix the mocks

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
       final AttendeeFoodPreference attendeeFoodPreference = new AttendeeFoodPreference( eventId, attendeeId, foodPreference );

       when( eventRepository.findById( eq( eventId ) ) ).thenReturn( Optional.of( eventEntity ) );
   /**/when( eventEntity.getId() ).thenReturn( eventId );
       when( eventEntity.getDate() ).thenReturn( LocalDate.now().plusDays( 1 ) );
       when( uuidGeneratorService.nextAttendeeId() ).thenReturn( attendeeId );
       doNothing().when( eventEntity ).addAttendee( attendeeEntity );
       when( eventRepository.save( eq( eventEntity ) ) ).thenReturn( eventEntity );
       doNothing().when( gateway ).submit( eq( attendeeFoodPreference ) );

       final EventRegistrationService service = new EventRegistrationService( eventRepository, uuidGeneratorService, gateway );
       final Optional<RegistrationConfirmation> confirmation = service.register( details );
       assertEquals( Optional.of( new RegistrationConfirmation( attendeeId ) ), confirmation );

       verify( eventRepository, times( 1 ) ).findById( eventId );
   /**/verify( eventEntity, times( 1 ) ).getId();
       verify( eventEntity, times( 1 ) ).getDate();
       verify( uuidGeneratorService, times( 1 ) ).nextAttendeeId();
       verify( eventEntity, times( 1 ) ).addAttendee( attendeeEntity );
       verify( eventRepository, times( 1 ) ).save( eventEntity );
       verify( gateway, times( 1 ) ).submit( attendeeFoodPreference );
       verifyNoMoreInteractions( eventRepository, eventEntity, uuidGeneratorService, gateway );
     }
   }
   ```

   Run the test again.

   ```bash
   $ ./gradlew clean test "--tests" "*EventRegistrationServiceTest"

   ...

   Event registration service > should return Optional empty when registering to an non existing event PASSED

   Event registration service > should return the registration confirmation when registering to an active event PASSED

   Event registration service > should return Optional empty when registering to an expired event PASSED

   BUILD SUCCESSFUL in 7s
   5 actionable tasks: 5 executed
   ```

   All three tests should pass.

1. Refactor the service

   ```java
   package demo.boot.event;

   import lombok.AllArgsConstructor;
   import org.springframework.stereotype.Service;

   import java.time.LocalDate;
   import java.util.Optional;
   import java.util.function.Function;

   @Service
   @AllArgsConstructor
   public class EventRegistrationService {

     private final EventRepository repository;
     private final UuidGeneratorService uuidGeneratorService;
     private final EventFoodGateway gateway;

     public Optional<RegistrationConfirmation> register( final RegistrationDetails registration ) {
       return repository
         .findById( registration.getEventId() )
         .filter( event -> LocalDate.now().isBefore( event.getDate() ) )
         .map( registerAttendee( registration ) )
   /**/  .map( submitFoodPreference() )
         .map( attendee -> new RegistrationConfirmation( attendee.getId() ) )
         ;
     }

   /**/private Function<EventAttendeeEntity, EventAttendeeEntity> submitFoodPreference() {
   /**/return attendee -> {
   /**/  final AttendeeFoodPreference preference = new AttendeeFoodPreference();
   /**/  preference.setEventId( attendee.getEvent().getId() );
   /**/  preference.setAttendeeId( attendee.getId() );
   /**/  preference.setFoodPreference( attendee.getFoodPreference() );
   /**/  gateway.submit( preference );
   /**/  return attendee;
   /**/};
   /**/}

     private Function<EventEntity, EventAttendeeEntity> registerAttendee( final RegistrationDetails registration ) { /* ... */ }
   }
   ```

   The above refactoring make all code within the `register()` method to be at the same abstraction level and the other two method are dealing with the respective logic.

   Run the tests.

   ```bash
   $ ./gradlew clean test "--tests" "*EventRegistrationServiceTest"

   ...

   Event registration service > should return Optional empty when registering to an non existing event PASSED

   Event registration service > should return the registration confirmation when registering to an active event PASSED

   Event registration service > should return Optional empty when registering to an expired event PASSED

   BUILD SUCCESSFUL in 7s
   5 actionable tasks: 5 executed
   ```

   All three tests should still pass.

The service is ready, and all hooked up.  It invokes the gateway with the food preference after the attendee has successfully registered to an event.  The gateway is still blank and still needs to be implement.

## Gateway

Spring framework provides an [`AmqpTemplate`](https://docs.spring.io/spring-amqp/api/org/springframework/amqp/core/AmqpTemplate.html) which we can use to send messaged to a queue.

1. Add the [AMQP starter](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-amqp) dependency

   Update file: `build.gradle`

   {% include custom/project_dose_not_compile.html %}

   ```groovy
     /* MQ */
     implementation 'org.springframework.boot:spring-boot-starter-amqp'
   ```

   Spring now expects the queue configuration details.

1. Update the application properties

   Update file: `src/main/resources/application.yaml`

   ```yaml
   spring:

     rabbitmq:
       host: ${MESSAGE_QUEUE_HOST}
       port: ${MESSAGE_QUEUE_PORT}
       username: ${MESSAGE_QUEUE_USERNAME}
       password: ${MESSAGE_QUEUE_PASSWORD}
   ```

   Spring use the environment variables [defined in the `.env` file]({{ '/docs/amqp/rabbit-mq/#docker-compose-setup-rabbitmq-management' | absolute_url }}).

1. Start the dependencies (_if these are not already running_)

   {% include custom/proceed_with_caution.html details="The following command will delete <strong>all</strong> stopped containers.  If you do not want to delete old containers, please do not run the <code>docker system prune -f</code> command." %}

   ```bash
   $ docker-compose stop && docker system prune -f && docker-compose up -d
   ```

1. Create an integration test

   Create file: `src/test-integration/java/demo/boot/event/EventFoodGatewayTest.java`

   ```java
   package demo.boot.event;

   import org.junit.jupiter.api.DisplayName;

   @DisplayName( "Event food gateway" )
   public class EventFoodGatewayTest {
   }
   ```

1. Add the gateway to the test

   Update file: `src/test-integration/java/demo/boot/event/EventFoodGatewayTest.java`

   ```java
   package demo.boot.event;

   import org.junit.jupiter.api.DisplayName;
   import org.junit.jupiter.api.Test;
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.boot.test.context.SpringBootTest;

   @DisplayName( "Event food gateway" )
   @SpringBootTest( webEnvironment = SpringBootTest.WebEnvironment.NONE )
   public class EventFoodGatewayTest {

     @Autowired
     private EventFoodGateway gateway;

     @Test
     @DisplayName( "should send the given preference to the queue" )
     public void shouldSendMessage() {
     }
   }
   ```

   Run the integration test

   ```bash
   $ ./gradlew clean integrationTest "--tests" "*EventFoodGatewayTest"

   ...

   Event food gateway > should send the given preference to the queue FAILED
     java.lang.IllegalStateException at DefaultCacheAwareContextLoaderDelegate.java:132
       Caused by: org.springframework.beans.factory.UnsatisfiedDependencyException at ConstructorResolver.java:798
         Caused by: org.springframework.beans.factory.UnsatisfiedDependencyException at ConstructorResolver.java:798
           Caused by: org.springframework.beans.factory.NoSuchBeanDefinitionException at DefaultListableBeanFactory.java:1716

   ...

   BUILD FAILED in 9s
   6 actionable tasks: 6 executed
   ```

   Spring will fail to wire the `EventFoodGateway`, as shown above.

1. Make the test pass

   Update file: `src/main/java/demo/boot/event/EventFoodGateway.java`

   ```java
   package demo.boot.event;

   import org.springframework.stereotype.Service;

   /**/@Service
   public class EventFoodGateway { /* ... */ }
   ```

   Run all tests

   ```bash
   $ ./gradlew clean check

   ...

   BUILD SUCCESSFUL in 16s
   7 actionable tasks: 7 executed
   ```

   {% include custom/note.html details="The integration tests will fail if we miss configure the Rabbit MQ as the health check will return a 503." %}

1. Assert that a message is actually sent to the queue

   Update file: `src/test-integration/java/demo/boot/event/EventFoodGatewayTest.java`

   {% include custom/note.html details="The following test is not asserting anything on purpose." %}

   ```java
   package demo.boot.event;

   import org.junit.jupiter.api.DisplayName;
   import org.junit.jupiter.api.Test;
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.boot.test.context.SpringBootTest;

   import java.util.UUID;

   @DisplayName( "Event food gateway" )
   @SpringBootTest( webEnvironment = SpringBootTest.WebEnvironment.NONE )
   public class EventFoodGatewayTest {

     @Autowired
     private EventFoodGateway gateway;

     @Test
     @DisplayName( "should send the given preference to the queue" )
     public void shouldSendMessage() {
       final AttendeeFoodPreference sent =
         new AttendeeFoodPreference( UUID.randomUUID(), UUID.randomUUID(), FoodPreference.NO_FOOD );

       gateway.submit( sent );
     }
   }
   ```

   Run the test

   ```bash
   $ ./gradlew clean integrationTest "--tests" "*EventFoodGatewayTest"

   ...

   BUILD SUCCESSFUL in 9s
   6 actionable tasks: 6 executed
   ```

   The test succeeds despite the fact the message is not being sent (as our `EventFoodGateway` is not doing anything).  Update the test and use `AmqpTemplate` to verify that the message is sent.

   Update file: `src/test-integration/java/demo/boot/event/EventFoodGatewayTest.java`

   ```java
   package demo.boot.event;

   import org.junit.jupiter.api.DisplayName;
   import org.junit.jupiter.api.Test;
   import org.springframework.amqp.core.AmqpTemplate;
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.boot.test.context.SpringBootTest;
   import org.springframework.core.ParameterizedTypeReference;

   import java.util.UUID;

   import static org.junit.jupiter.api.Assertions.assertEquals;

   @DisplayName( "Event food gateway" )
   @SpringBootTest( webEnvironment = SpringBootTest.WebEnvironment.NONE )
   public class EventFoodGatewayTest {

     @Autowired
     private EventFoodGateway gateway;

     @Autowired
     private AmqpTemplate template;

     @Test
     @DisplayName( "should send the given preference to the queue" )
     public void shouldSendMessage() {
       final AttendeeFoodPreference sent =
         new AttendeeFoodPreference( UUID.randomUUID(), UUID.randomUUID(), FoodPreference.NO_FOOD );

       gateway.submit( sent );

       final AttendeeFoodPreference received =
         template.receiveAndConvert( "food", 5000, ParameterizedTypeReference.forType( AttendeeFoodPreference.class ) );
       assertEquals( sent, received );
     }
   }
   ```

   The test first submits the food preference using the gateway, then reads the queue using the `AmqpTemplate`.  Running the tests now will fail.

   ```bash
   $ ./gradlew clean integrationTest "--tests" "*EventFoodGatewayTest"

   ...

   Event food gateway > should send the given preference to the queue FAILED
       org.opentest4j.AssertionFailedError at EventFoodGatewayTest.java:34

   ...

   BUILD FAILED in 14s
   6 actionable tasks: 6 executed
   ```

   The `AmqpTemplate` will timeout as nothing is being sent, and the template will return `null`, as shown next.

   ```bash
   org.opentest4j.AssertionFailedError: expected: <AttendeeFoodPreference(eventId=e3866e45-94d8-484f-8264-853d0d072490, attendeeId=0a547db1-7e7d-49d4-b1bf-b46d04e2d042, foodPreference=NO_FOOD)> but was: <null>
     at org.junit.jupiter.api.AssertionUtils.fail(AssertionUtils.java:55)
     at org.junit.jupiter.api.AssertionUtils.failNotEqual(AssertionUtils.java:62)
     at org.junit.jupiter.api.AssertEquals.assertEquals(AssertEquals.java:182)
     at org.junit.jupiter.api.AssertEquals.assertEquals(AssertEquals.java:177)
   ```

1. Make the test pass

   Update file: `src/main/java/demo/boot/event/EventFoodGateway.java`

   {% include custom/note.html details="The queue name is currently hard-coded.  We will parametrise this once we make the test pass." %}

   ```java
   package demo.boot.event;

   import lombok.AllArgsConstructor;
   import org.springframework.amqp.core.AmqpTemplate;
   import org.springframework.stereotype.Service;

   @Service
   @AllArgsConstructor
   public class EventFoodGateway {

     private final AmqpTemplate template;

     public void submit( final AttendeeFoodPreference preference ) {
   /**/template.convertAndSend( "food", preference );
     }
   }
   ```

   Run the test

   ```bash
   $ ./gradlew clean integrationTest "--tests" "*EventFoodGatewayTest"

   ...

   Event food gateway > should send the given preference to the queue FAILED
     java.lang.IllegalArgumentException at EventFoodGatewayTest.java:30

   ...

   BUILD FAILED in 8s
   6 actionable tasks: 6 executed
   ```

   To our surprise, the test fails as Spring is not able to convert our non-Serializable object into a message.

   ```bash
   java.lang.IllegalArgumentException: SimpleMessageConverter only supports String, byte[] and Serializable payloads, received: demo.boot.event.AttendeeFoodPreference
     at org.springframework.amqp.support.converter.SimpleMessageConverter.createMessage(SimpleMessageConverter.java:161)
     at org.springframework.amqp.support.converter.AbstractMessageConverter.createMessage(AbstractMessageConverter.java:88)
     at org.springframework.amqp.support.converter.AbstractMessageConverter.toMessage(AbstractMessageConverter.java:70)
     at org.springframework.amqp.support.converter.AbstractMessageConverter.toMessage(AbstractMessageConverter.java:58)
   ```

1. Add a JSON to Object converter

   Create file: `src/main/java/demo/boot/event/MessageQueueConfiguration.java`

   ```java
   package demo.boot.event;

   import org.springframework.amqp.support.converter.Jackson2JsonMessageConverter;
   import org.springframework.amqp.support.converter.MessageConverter;
   import org.springframework.context.annotation.Bean;
   import org.springframework.context.annotation.Configuration;

   @Configuration
   public class MessageQueueConfiguration {

     @Bean
     public MessageConverter jsonMessageConverter() {
       return new Jackson2JsonMessageConverter();
     }
   }
   ```

   Run the test

   ```bash
   $ ./gradlew clean integrationTest "--tests" "*EventFoodGatewayTest"

   ...

   BUILD SUCCESSFUL in 11s
   6 actionable tasks: 6 executed
   ```

   The test should now pass.

1. Parametrise the queue name

   We hardcoded the queue name, `food`, in both the test and the gateway.  We can parametrise this instead.

   1. Add new environment variable

      Update file: `.env`

      ```properties
      # Application queues name
      APP_FOOD_QUEUE_NAME=food
      APP_EVENT_QUEUE_NAME=event
      ```

   1. Create application properties

      Update file: `src/main/resources/application.yaml`

      ```yaml
      app:
        queue:
          food: ${APP_FOOD_QUEUE_NAME}
          event: ${APP_EVENT_QUEUE_NAME}
      ```

   1. Use the properties from within the test

      Update file: `src/test-integration/java/demo/boot/event/EventFoodGatewayTest.java`

      ```java
      package demo.boot.event;

      import org.junit.jupiter.api.DisplayName;
      import org.junit.jupiter.api.Test;
      import org.springframework.amqp.core.AmqpTemplate;
      import org.springframework.beans.factory.annotation.Autowired;
      import org.springframework.beans.factory.annotation.Value;
      import org.springframework.boot.test.context.SpringBootTest;
      import org.springframework.core.ParameterizedTypeReference;

      import java.util.UUID;

      import static org.junit.jupiter.api.Assertions.assertEquals;

      @DisplayName( "Event food gateway" )
      @SpringBootTest( webEnvironment = SpringBootTest.WebEnvironment.NONE )
      public class EventFoodGatewayTest {

        @Autowired
        private EventFoodGateway gateway;

        @Autowired
        private AmqpTemplate template;

      /**/@Value( "${app.queue.food}" )
      /**/private String queueName;

        @Test
        @DisplayName( "should send the given preference to the queue" )
        public void shouldSendMessage() {
          final AttendeeFoodPreference sent =
            new AttendeeFoodPreference( UUID.randomUUID(), UUID.randomUUID(), FoodPreference.NO_FOOD );

          gateway.submit( sent );

          final AttendeeFoodPreference received =
      /**/  template.receiveAndConvert( queueName, 5000, ParameterizedTypeReference.forType( AttendeeFoodPreference.class ) );
          assertEquals( sent, received );
        }
      }
      ```

      Run the test

      ```bash
      $ ./gradlew clean integrationTest "--tests" "*EventFoodGatewayTest"

      ...

      BUILD SUCCESSFUL in 8s
      6 actionable tasks: 6 executed
      ```

      The test should still pass.

   1. Use the properties from within the gateway

      Update file: `src/main/java/demo/boot/event/EventFoodGateway.java`

      {% include custom/note.html details="We stopped making use of lombok and instead created our constructor and annotated the <code>queueName</code> parameter." %}

      ```java
      package demo.boot.event;

      import org.springframework.amqp.core.AmqpTemplate;
      import org.springframework.beans.factory.annotation.Value;
      import org.springframework.stereotype.Service;

      @Service
      /* DELETE @AllArgsConstructor */
      public class EventFoodGateway {

      /**/private final String queueName;
        private final AmqpTemplate template;

      /**/public EventFoodGateway(
      /**/@Value( "${app.queue.food}" ) final String queueName,
      /**/final AmqpTemplate template ) {
      /**/this.queueName = queueName;
      /**/this.template = template;
      /**/}

        public void submit( final AttendeeFoodPreference preference ) {
      /**/template.convertAndSend( queueName, preference );
        }
      }
      ```

      Run all tests

      ```bash
      $ ./gradlew clean check

      ...

      BUILD SUCCESSFUL in 8s
      6 actionable tasks: 6 executed
      ```

      All test should still pass.
