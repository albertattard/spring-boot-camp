---
layout: default
title: Advanced Message Queuing Protocol
nav_order: 5
has_children: true
permalink: docs/amqp/
---

# Advanced Message Queuing Protocol (AMQP)

[Spring](https://spring.io/) simplifies messaging through the [Spring AMQP project](https://spring.io/projects/spring-amqp).  The Spring AMQP project abstracts the exchange of messages through annotations, such as [`@RabbitListener`](https://docs.spring.io/spring-amqp/api/org/springframework/amqp/rabbit/annotation/RabbitListener.html) (part of the [RabbitMQ](https://www.rabbitmq.com/) implementation), and templates, such as [`AmqpTemplate`](https://docs.spring.io/spring-amqp/api/org/springframework/amqp/core/AmqpTemplate.html) and [`RabbitTemplate`](https://docs.spring.io/spring-amqp/api/org/springframework/amqp/rabbit/core/RabbitTemplate.html).

Spring AMQP can be used with any message queue vendor that supports AMQP, such as [ActiveMQ](https://activemq.apache.org/).  Producers can exchange [POJO](https://en.wikipedia.org/wiki/Plain_old_Java_object)s with subscribers, who in turn will consume these messages.
