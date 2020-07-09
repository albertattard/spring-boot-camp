---
layout: default
title: Reactive Web
nav_order: 905
has_children: true
permalink: docs/reactive/
---

# Reactive Web

## Notes

"_Reactive Streams is a [small spec](https://github.com/reactive-streams/reactive-streams-jvm/blob/master/README.md#specification) (also adopted in [Java 9](https://docs.oracle.com/javase/9/docs/api/java/util/concurrent/Flow.html)) that defines the interaction between asynchronous components with back pressure. For example a data repository (acting as [Publisher](https://www.reactive-streams.org/reactive-streams-1.0.1-javadoc/org/reactivestreams/Publisher.html)) can produce data that an HTTP server (acting as [Subscriber](https://www.reactive-streams.org/reactive-streams-1.0.1-javadoc/org/reactivestreams/Subscriber.html)) can then write to the response. The main purpose of Reactive Streams is to let the subscriber control how quickly or how slowly the publisher produces data._"<br/>
([Reference](https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html#webflux-why-reactive))

"_[Reactor](https://github.com/reactor/reactor) is the reactive library of choice for Spring WebFlux. It provides the [`Mono`](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Mono.html) and [`Flux`](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html) API types to work on data sequences of 0..1 (`Mono`) and 0..N (`Flux`) through a rich set of operators aligned with the ReactiveX [vocabulary of operators](http://reactivex.io/documentation/operators.html). Reactor is a Reactive Streams library and, therefore, all of its operators support non-blocking back pressure. Reactor has a strong focus on server-side Java. It is developed in close collaboration with Spring._"<br/>
([Reference](https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html#webflux-reactive-api))

![Spring MVC & WebFlux](https://docs.spring.io/spring/docs/current/spring-framework-reference/images/spring-mvc-and-webflux-venn.png)
