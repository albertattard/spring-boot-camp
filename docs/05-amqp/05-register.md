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
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
   import org.springframework.test.web.servlet.MockMvc;

   @DisplayName( "Registration controller" )
   @WebMvcTest( EventRegistrationController.class )
   public class EventRegistrationControllerTest {

     @Autowired
     private MockMvc mockMvc;

   }
   ```

1. Register for an event that does not exist
