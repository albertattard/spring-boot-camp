---
layout: default
title: Register Attendees
description: Setup a REST endpoint that accepts attendees registration
lang: en
parent: Advanced Message Queuing Protocol
nav_order: 5
permalink: docs/amqp/register/
---

# Register Attendees
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Objectives

- [ ] Attendees cannot register to an event that does not exist
- [ ] Attendees cannot register to an expired event (event that happened in the past)
- [ ] Attendees should receive a confirmation following a successful registration

## Controller

1. Create new `event` package

   ```bash
   $ mkdir src/main/java/demo/boot/event
   $ mkdir src/test/java/demo/boot/event
   $ mkdir src/test-integration/java/demo/boot/event
   ```

1. Create the controller

   Create file: `src/main/java/demo/boot/event/EventRegistrationController.java`

   ```java
   package demo.boot.event;

   import org.springframework.web.bind.annotation.RestController;

   @RestController
   public class EventRegistrationController {

   }
   ```

1. Create the controller test

   Create file: `src/test/java/demo/boot/event/EventRegistrationControllerTest.java`

   ```java
   package demo.boot.event;

   import org.junit.jupiter.api.DisplayName;
   import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;

   @DisplayName( "Registration controller" )
   @WebMvcTest( EventRegistrationController.class )
   public class EventRegistrationControllerTest {

   }
   ```

1. Test for registration for an event that does not exist

   Update file: `src/test/java/demo/boot/event/EventRegistrationControllerTest.java`

   {% include custom/dose_not_compile.html %}

   ```java
   package demo.boot.event;

   import com.fasterxml.jackson.databind.ObjectMapper;
   import org.junit.jupiter.api.DisplayName;
   import org.junit.jupiter.api.Test;
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
   import org.springframework.boot.test.mock.mockito.MockBean;
   import org.springframework.http.MediaType;
   import org.springframework.test.web.servlet.MockMvc;

   import java.nio.charset.StandardCharsets;
   import java.util.Optional;
   import java.util.UUID;

   import static org.mockito.ArgumentMatchers.eq;
   import static org.mockito.Mockito.times;
   import static org.mockito.Mockito.verify;
   import static org.mockito.Mockito.verifyNoMoreInteractions;
   import static org.mockito.Mockito.when;
   import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.post;
   import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

   @DisplayName( "Registration controller" )
   @WebMvcTest( EventRegistrationController.class )
   public class EventRegistrationControllerTest {

     @Autowired
     private MockMvc mockMvc;

     @MockBean
     private EventRegistrationService service;

     @Autowired
     private ObjectMapper jsonObjectMapper;

     @Test
     @DisplayName( "should return not found when registering for an event that does not exists" )
     public void shouldReturnNotFound() throws Exception {
       final UUID eventId = UUID.randomUUID();
       final String name = "Aden Attard";
       final FoodPreference foodPreference = FoodPreference.MEAT;
       final RegistrationRequest registrationRequest = new RegistrationRequest( name, foodPreference );
       final RegistrationDetails details = new RegistrationDetails( eventId, name, foodPreference );

       when( service.register( eq( details ) ) ).thenReturn( Optional.empty() );

       mockMvc
         .perform(
           post( "/event/{eventId}/register", eventId )
             .contentType( MediaType.APPLICATION_JSON )
             .characterEncoding( StandardCharsets.UTF_8.displayName() )
             .content( jsonObjectMapper.writeValueAsString( registrationRequest ) )
         )
         .andExpect( status().isNotFound() )
       ;

       verify( service, times( 1 ) ).register( details );
       verifyNoMoreInteractions( service );
     }
   }
   ```

   Make the test compile.

   1. Create file: `src/main/java/demo/boot/event/FoodPreference.java`

      ```java
      package demo.boot.event;

      public enum FoodPreference {
        NO_FOOD,
        VEGETARIAN,
        VEGAN,
        MEAT
      }
      ```

   1. Create file: `src/main/java/demo/boot/event/RegistrationDetails.java`

      ```java
      package demo.boot.event;

      import lombok.AllArgsConstructor;
      import lombok.Data;
      import lombok.NoArgsConstructor;

      import java.util.UUID;

      @Data
      @NoArgsConstructor
      @AllArgsConstructor
      public class RegistrationDetails {

        private UUID eventId;
        private String name;
        private FoodPreference foodPreference;
      }
      ```

   1. Create file: `src/main/java/demo/boot/event/RegistrationConfirmation.java`

      ```java
      package demo.boot.event;

      import lombok.AllArgsConstructor;
      import lombok.Data;
      import lombok.NoArgsConstructor;

      import java.util.UUID;

      @Data
      @NoArgsConstructor
      @AllArgsConstructor
      public class RegistrationConfirmation {

        private UUID id;
      }
      ```

   1. Create file: `src/main/java/demo/boot/event/EventRegistrationService.java`

      ```java
      package demo.boot.event;

      import java.util.Optional;

      public class EventRegistrationService {

        public Optional<RegistrationConfirmation> register( final RegistrationDetails registration ) {
          return Optional.empty();
        }
      }
      ```

   1. Create file: `src/main/java/demo/boot/event/RegistrationRequest.java`

      ```java
      package demo.boot.event;

      import lombok.AllArgsConstructor;
      import lombok.Data;
      import lombok.NoArgsConstructor;

      @Data
      @NoArgsConstructor
      @AllArgsConstructor
      public class RegistrationRequest {

        private String name;
        private FoodPreference foodPreference;
      }
      ```

   The test should not compile.  Run the test.

   ```bash
   $ ./gradlew clean test

   ...

   Registration controller > should return not found when registering for an event that does not exists FAILED
       org.mockito.exceptions.verification.WantedButNotInvoked at EventRegistrationControllerTest.java:58

   ...

   BUILD FAILED in 9s
   5 actionable tasks: 5 executed
   ```

   The test will fail, as expected.

   {% include custom/note.html details="The test had received a 404, as expected but the controller did not interact with the mocks as expected" %}

   ```bash
   $ open "build/reports/tests/test/classes/demo.boot.event.EventRegistrationControllerTest.html"
   ```

   ![Event-Registration-Controller-Test-shouldReturnNotFound.png]({{ '/assets/images/Event-Registration-Controller-Test-shouldReturnNotFound.png' | absolute_url }})
