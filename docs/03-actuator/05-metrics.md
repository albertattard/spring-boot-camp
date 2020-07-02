---
layout: default
title: Metrics
parent: Spring Boot Actuator
nav_order: 5
permalink: docs/actuator/metrics/
---

# Metrics
{: .no_toc }

The _Contact Us_ application needs to be monitored and we need to make sure that the application does not behave funnily, and any anomalies needs to be tracked.  Metrics can help us with this task.  [Spring Boot Actuator](https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-features.html#production-ready-metrics) provides dependency management and auto-configuration for [Micrometer](https://micrometer.io/), an application metrics facade that supports numerous monitoring systems, such as [Prometheus](https://prometheus.io/) and [DataDog](https://www.datadoghq.com/).

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## What are metrics?

Our application will receive traffic, use CPU and memory, connect to a database and carry out other tasks.  We can measure these tasks to gain insight on how our application behaves.  This can help us identify slow areas in our code and prioritise work where it is needed the most.

There are many tools from different vendors that can help us with this, such as Prometheus and DataDog.

## Which tool/vendor should we use?

I like to use [Google trends](https://trends.google.com/trends/explore?q=Prometheus,Datadog,StatsD,New%20Relic,JMX) to get a feel about their popularity, as shown next.

{% include custom/note.html details="The following result is very skewed as some of these names have different meanings.  For example, <em>Prometheus</em> is the name of a <a href='https://prometheus.io/'>monitoring and analysis tool</a>, and also the name of a <a href='https://en.wikipedia.org/wiki/Prometheus'>Greek God</a>, and there is a <a href='https://en.wikipedia.org/wiki/Prometheus_(2012_film)'>movie</a> with that name too." %}

![Metrics-Vendors-Trends.png]({{ '/assets/images/Metrics-Vendors-Trends.png' | absolute_url }})

**I have little experience in this field**, and only used Prometheus with [Grafana](https://grafana.com/) as the visualisation tool.  Prometheus seems to be very popular in this field, irrespective from the Google trends results shown above.  This is an open-source project and that joined the [Cloud Native Computing Foundation](https://www.cncf.io/) in 2016 as the second hosted project, after [Kubernetes](https://kubernetes.io/) ([reference](https://prometheus.io/docs/introduction/overview/)).

Each vendor has its own API and switching to a different vendor may be quite a daunting task as we need to redo all metrics code to fit the new vendor's API.  An alternative solution is to use a facade, such as Micrometer, to standardise the interaction between our application and the vendor we picked.  Switching from one vendor to another should be a seamless thing, at least from the application point of view.

## What is Micrometer?

Metrics, about our application, can be collected and analysed by different tools from different vendors.  Micrometer acts as a facade between our application and various vendors, as shown in the following diagram.

![Micrometer]({{ '/assets/images/Micrometer.png' | absolute_url }})

"_Micrometer provides a simple facade over the instrumentation clients for the most popular monitoring systems, allowing you to instrument your JVM-based application code without vendor lock-in. Think [SLF4J](http://www.slf4j.org/), but for metrics._"<br/>
([Reference](https://micrometer.io/))

Micrometer is a facade, a dependency that our project will depend on, for various vendors.  Micrometer is not a service running somewhere but will be packed part of our application like other Spring related dependencies.

This provides us a single interface which we can use to publish metrics about our application.  Having one facade simplifies life as we do not have to worry about each individual vendor.  For example, Prometheus polls metrics from our application, whereas DataDog requires our application to push the metrics to it.  These are not trivial differences, as in the latter case (DataDog) we need to make a request while in the former case (Prometheus) we need to expose an endpoint.

## Enable metrics

Spring Boot Actuator provides metrics.  The metrics endpoint needs to be enabled through the `src/main/resources/application.yaml`

```yaml
management:
  endpoints:
    web:
      base-path:
      exposure:
        include: env, health, info, metrics
```

The above example shows four, from the many, [Actuator endpoints](https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-features.html#production-ready-endpoints) enabled.

We can retrieve the metrics produced by our application using the `/metrics` endpoint

```bash
$ curl "http://localhost:8080/metrics" | jq .
```

This will return a list of metrics available as shown next.

```yaml
{
  "names": [
    "hikaricp.connections",
    "hikaricp.connections.acquire",
    "hikaricp.connections.active",
    "hikaricp.connections.creation",
    "hikaricp.connections.idle",
    "hikaricp.connections.max",
    "hikaricp.connections.min",
    "hikaricp.connections.pending",
    "hikaricp.connections.timeout",
    "hikaricp.connections.usage",
    "http.server.requests",
    "jdbc.connections.active",
    "jdbc.connections.idle",
    "jdbc.connections.max",
    "jdbc.connections.min",
    "jvm.buffer.count",
    "jvm.buffer.memory.used",
    "jvm.buffer.total.capacity",
    "jvm.classes.loaded",
    "jvm.classes.unloaded",
    "jvm.gc.live.data.size",
    "jvm.gc.max.data.size",
    "jvm.gc.memory.allocated",
    "jvm.gc.memory.promoted",
    "jvm.gc.pause",
    "jvm.memory.committed",
    "jvm.memory.max",
    "jvm.memory.used",
    "jvm.threads.daemon",
    "jvm.threads.live",
    "jvm.threads.peak",
    "jvm.threads.states",
    "logback.events",
    "process.cpu.usage",
    "process.files.max",
    "process.files.open",
    "process.start.time",
    "process.uptime",
    "system.cpu.count",
    "system.cpu.usage",
    "system.load.average.1m",
    "tomcat.sessions.active.current",
    "tomcat.sessions.active.max",
    "tomcat.sessions.alive.max",
    "tomcat.sessions.created",
    "tomcat.sessions.expired",
    "tomcat.sessions.rejected"
  ]
}
```

We can retrieve more information about individual metric by adding the metric name to the path as shown next.

```bash
$ curl "http://localhost:8080/metrics/http.server.requests" | jq .
```

The above example, retrieved the metric for the `http.server.requests`, as shown next.

```json
{
  "name": "http.server.requests",
  "description": null,
  "baseUnit": "seconds",
  "measurements": [
    {
      "statistic": "COUNT",
      "value": 3
    },
    {
      "statistic": "TOTAL_TIME",
      "value": 0.089996207
    },
    {
      "statistic": "MAX",
      "value": 0.01274303
    }
  ],
  "availableTags": [
    {
      "tag": "exception",
      "values": [
        "None"
      ]
    },
    {
      "tag": "method",
      "values": [
        "GET"
      ]
    },
    {
      "tag": "uri",
      "values": [
        "/env",
        "/health",
        "/metrics"
      ]
    },
    {
      "tag": "outcome",
      "values": [
        "SUCCESS"
      ]
    },
    {
      "tag": "status",
      "values": [
        "200"
      ]
    }
  ]
}
```

The details returned vary between different metrics.

## What is the difference between metrics and tracing?

Metrics can be used to measure various parts and aspects of our applications.  [Tracing]({{ '/docs/sleuth/' | absolute_url }}) is the ability to connect a series of steps that one request goes through.  Consider a request made to our application as shown in the following diagram.

![Request-From-Browser-To-Db]({{ '/assets/images/Request-From-Browser-To-Db.png' | absolute_url }})

With metrics we can measure the number of such requests, the time taken to process our requests, how much memory is being used and other things like these.  Tracing enables us to follow this one request from start to finish.  These two provide two different types of information:

* Metrics shed light on the application and its parts
* Tracing shed light on individual requests

## Can we create custom metrics?

**YES**

Our application manages the offices [ThoughtWorks](https://www.thoughtworks.com/) has around the world.  Let's create a custom metrics that measures the number of offices ThoughtWorks currently has.

We would like to add a new metrics, with the name `app.office.count`, which returns the number of offices in the database.

```bash
$ curl "http://localhost:8080/metrics/app.office.count" | jq .
```

The above command should return the number of office present as shown next.

```json
{
  "name": "app.office.count",
  "description": null,
  "baseUnit": null,
  "measurements": [
    {
      "statistic": "COUNT",
      "value": 6
    }
  ],
  "availableTags": []
}
```

The database currently has 6 office, therefore the count should reflect that number.

No tags need to be used in this example (`availableTags`), and no need to get fancy with `description` and `baseUnit` to keep things simple.  If these are needed we can use [`DistributionSummary`](https://www.javadoc.io/doc/io.micrometer/micrometer-core/latest/io/micrometer/core/instrument/DistributionSummary.html) instead.

Spring Boot Actuator will provide us with an instance of [`MeterRegistry`](https://www.javadoc.io/doc/io.micrometer/micrometer-core/latest/io/micrometer/core/instrument/MeterRegistry.html) which we can use to register our metrics.  We will use a [`Counter`](https://www.javadoc.io/doc/io.micrometer/micrometer-core/latest/io/micrometer/core/instrument/Counter.html) to keep track of the number of offices.

There are several approaches we can take, some of which are highlighted below.

* One may be tempted to update the existing `JpaContactUsService` and handle the metrics from there.

  I do not recommend this approach as we will be mixing concerns.  The `JpaContactUsService` class would be handling two different things: handling the business logic related to offices and also handling the metrics for the offices.

* We can use [AOP](https://en.wikipedia.org/wiki/Aspect-oriented_programming) and intercept calls to the `JpaContactUsService`

  This is a very tempting solution and I've used this approach in the past.  It is very elegant but may seem a bit cryptic.

* Use the [decorator pattern](https://en.wikipedia.org/wiki/Decorator_pattern)

  We can create another instance of `ContactUsService` service and decorate the `JpaContactUsService` service, as shown in the following image.

  ![Office-Count-Decorator.png]({{ '/assets/images/Office-Count-Decorator.png' | absolute_url }})

  The office count will be adjusted when offices are deleted, as that's the only operation we currently have that changes the number of offices we have.

  {% include custom/note.html details="The decorator will delegate all office service related business logic to the <code>JpaContactUsService</code>.  The decorator will only adjust the metrics counters based in the results of some of the interactions." %}

Will use the decorator pattern, not because it uses less code, but because it is easier to follow and understand.

1. Create a test

   Create file `src/test/java/demo/boot/OfficeCountMetricDecoratorTest.java`

   {% include custom/dose_not_compile.html %}

   ```java
   package demo.boot;

   import io.micrometer.core.instrument.Counter;
   import io.micrometer.core.instrument.MeterRegistry;
   import org.junit.jupiter.api.DisplayName;
   import org.junit.jupiter.api.Test;

   import static org.mockito.ArgumentMatchers.eq;
   import static org.mockito.Mockito.mock;
   import static org.mockito.Mockito.times;
   import static org.mockito.Mockito.verify;
   import static org.mockito.Mockito.verifyNoMoreInteractions;
   import static org.mockito.Mockito.when;

   @DisplayName( "Office count metric" )
   public class OfficeCountMetricDecoratorTest {

     @Test
     @DisplayName( "should set the counter to the number of offices in the repository" )
     public void shouldInitCounter() {
       final long initialNumberOfOffices = 10L;
       final String counterName = "app.office.count";

       final MeterRegistry registry = mock( MeterRegistry.class );
       final Counter counter = mock( Counter.class );
       final OfficesRepository repository = mock( OfficesRepository.class );

       when( registry.counter( eq( counterName ) ) ).thenReturn( counter );
       when( repository.count() ).thenReturn( initialNumberOfOffices );

       new OfficeCountMetricDecorator( registry, repository );

       verify( registry, times( 1 ) ).counter( counterName );
       verify( repository, times( 1 ) ).count();
       verify( counter, times( 1 ) ).increment( eq( (double) initialNumberOfOffices ) );
       verifyNoMoreInteractions( registry, repository, counter );
     }
   }
   ```

   Our decorator will start by registering a `Counter` with the `MeterRegistry` and set the initial value to the current number of offices in the database.  We will use the `OfficesRepository` to get the number of offices in the database using the [`count()`](https://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/repository/CrudRepository.html?is-external=true#count--) method, as this more efficient than calling the `list()` method on `JpaContactUsService` and then get its size.

1. Make the test compile

   Create file `src/main/java/demo/boot/OfficeCountMetricDecorator.java`

   ```java
   package demo.boot;

   import io.micrometer.core.instrument.MeterRegistry;

   import java.util.Collections;
   import java.util.List;
   import java.util.Optional;

   public class OfficeCountMetricDecorator implements ContactUsService {

     public OfficeCountMetricDecorator( final MeterRegistry registry, final OfficesRepository repository ) {
     }

     @Override
     public List<Office> list() {
       return Collections.emptyList();
     }

     @Override
     public Optional<Office> findOneByName( final String name ) {
       return Optional.empty();
     }

     @Override
     public List<Office> findAllInCountry( final String country ) {
       return Collections.emptyList();
     }

     @Override
     public Optional<Office> update( final Office office ) {
       return Optional.empty();
     }

     @Override
     public Optional<Office> delete( final String name ) {
       return Optional.empty();
     }
   }
   ```

   Run the tests.

   ```bash
   $ ./gradlew clean test
   ```

   The test will fail as the `Counter` is not properly setup.


1. Fix the failing test.

   Update file `src/main/java/demo/boot/OfficeCountMetricDecorator.java`

   ```java
   package demo.boot;

   import io.micrometer.core.instrument.Counter;
   import io.micrometer.core.instrument.MeterRegistry;

   import java.util.Collections;
   import java.util.List;
   import java.util.Optional;

   public class OfficeCountMetricDecorator implements ContactUsService {

     private final Counter officeCounter;

     public OfficeCountMetricDecorator( final MeterRegistry registry, final OfficesRepository repository ) {
       officeCounter = registry.counter( "app.office.count" );
       officeCounter.increment( repository.count() );
     }

     @Override
     public List<Office> list() { /* ... */ }

     @Override
     public Optional<Office> findOneByName( final String name ) { /* ... */ }

     @Override
     public List<Office> findAllInCountry( final String country ) { /* ... */ }

     @Override
     public Optional<Office> update( final Office office ) { /* ... */ }

     @Override
     public Optional<Office> delete( final String name ) { /* ... */ }
   }
   ```

   Run the tests.  These should all pass.

1. Prepare the delegator, `OfficeCountMetricDecorator`, to work with `JpaContactUsService`

   We need to introduce the `JpaContactUsService` and pass all requests to it.  Before we can add any new tests, we need to refactor the existing test to take the new parameter.

   Update file: `src/test/java/demo/boot/OfficeCountMetricDecoratorTest.java`

   {% include custom/dose_not_compile.html %}

   ```java
   package demo.boot;

   import io.micrometer.core.instrument.Counter;
   import io.micrometer.core.instrument.MeterRegistry;
   import org.junit.jupiter.api.DisplayName;
   import org.junit.jupiter.api.Test;

   import static org.mockito.ArgumentMatchers.eq;
   import static org.mockito.Mockito.mock;
   import static org.mockito.Mockito.times;
   import static org.mockito.Mockito.verify;
   import static org.mockito.Mockito.verifyNoMoreInteractions;
   import static org.mockito.Mockito.when;

   @DisplayName( "Office count metric" )
   public class OfficeCountMetricDecoratorTest {

     @Test
     @DisplayName( "should set the counter to the number of offices in the repository" )
     public void shouldInitCounter() {
       final long initialNumberOfOffices = 10L;
       final String counterName = "app.office.count";

       final MeterRegistry registry = mock( MeterRegistry.class );
       final Counter counter = mock( Counter.class );
       final OfficesRepository repository = mock( OfficesRepository.class );
       final JpaContactUsService target = mock( JpaContactUsService.class );

       when( registry.counter( eq( counterName ) ) ).thenReturn( counter );
       when( repository.count() ).thenReturn( initialNumberOfOffices );

       new OfficeCountMetricDecorator( registry, repository, target );

       verify( registry, times( 1 ) ).counter( counterName );
       verify( repository, times( 1 ) ).count();
       verify( counter, times( 1 ) ).increment( eq( (double) initialNumberOfOffices ) );
       verifyNoMoreInteractions( registry, repository, counter, target );
     }
   }
   ```

   Make the test compile.

   Update file: `src/main/java/demo/boot/OfficeCountMetricDecorator.java`

   ```java
   package demo.boot;

   import io.micrometer.core.instrument.Counter;
   import io.micrometer.core.instrument.MeterRegistry;

   import java.util.Collections;
   import java.util.List;
   import java.util.Optional;

   public class OfficeCountMetricDecorator implements ContactUsService {

     private final Counter officeCounter;

     public OfficeCountMetricDecorator( final MeterRegistry registry, final OfficesRepository repository,
       final JpaContactUsService target ) {
       officeCounter = registry.counter( "app.office.count" );
       officeCounter.increment( repository.count() );
     }

     @Override
     public List<Office> list() { /* ... */ }

     @Override
     public Optional<Office> findOneByName( final String name ) { /* ... */ }

     @Override
     public List<Office> findAllInCountry( final String country ) { /* ... */ }

     @Override
     public Optional<Office> update( final Office office ) { /* ... */ }

     @Override
     public Optional<Office> delete( final String name ) { /* ... */ }
   }
   ```

   Run the tests.

   ```bash
   $ ./gradlew clean test

   BUILD SUCCESSFUL in 6s
   6 actionable tasks: 6 executed
   ```

   These should still pass.

1. Make sure that `list()` call is passed to the `JpaContactUsService`

   Update file: `src/test/java/demo/boot/OfficeCountMetricDecoratorTest.java`

   ```java
   package demo.boot;

   import io.micrometer.core.instrument.Counter;
   import io.micrometer.core.instrument.MeterRegistry;
   import org.junit.jupiter.api.DisplayName;
   import org.junit.jupiter.api.Test;

   import java.util.List;

   import static org.assertj.core.api.Assertions.assertThat;
   import static org.mockito.ArgumentMatchers.eq;
   import static org.mockito.Mockito.mock;
   import static org.mockito.Mockito.times;
   import static org.mockito.Mockito.verify;
   import static org.mockito.Mockito.verifyNoMoreInteractions;
   import static org.mockito.Mockito.when;

   @DisplayName( "Office count metric" )
   public class OfficeCountMetricDecoratorTest {

     @Test
     @DisplayName( "should set the counter to the number of offices in the repository" )
     public void shouldInitCounter() { /* ... */ }

     @Test
     @DisplayName( "should call the target list() method without changing the counter's value" )
     public void shouldPassListRequestsThrough() {
       final long initialNumberOfOffices = 10L;
       final String counterName = "app.office.count";

       final MeterRegistry registry = mock( MeterRegistry.class );
       final Counter counter = mock( Counter.class );
       final OfficesRepository repository = mock( OfficesRepository.class );
       final JpaContactUsService target = mock( JpaContactUsService.class );
       final Office office = mock( Office.class );

       when( registry.counter( eq( counterName ) ) ).thenReturn( counter );
       when( repository.count() ).thenReturn( initialNumberOfOffices );

       final List<Office> expected = List.of( office );
       when( target.list() ).thenReturn( expected );

       final ContactUsService subject = new OfficeCountMetricDecorator( registry, repository, target );
       final List<Office> actual = subject.list();
       assertThat( actual ).isSameAs( expected );

       verify( target, times( 1 ) ).list();

       verify( registry, times( 1 ) ).counter( counterName );
       verify( repository, times( 1 ) ).count();
       verify( counter, times( 1 ) ).increment( eq( (double) initialNumberOfOffices ) );
       verifyNoMoreInteractions( registry, repository, counter, target, office );
     }
   }
   ```

   Run the tests.

   ```bash
   $ ./gradlew clean test

   ...

   Office count metric > should call the target list() method without changing the counter's value FAILED
       java.lang.AssertionError at OfficeCountMetricDecoratorTest.java:63

   ...

   BUILD FAILED in 7s
   6 actionable tasks: 6 executed
   ```

   As expected, the tests failed.  Make the test pass.

   Update file: `src/main/java/demo/boot/OfficeCountMetricDecorator.java`

   ```java
   package demo.boot;

   import io.micrometer.core.instrument.Counter;
   import io.micrometer.core.instrument.MeterRegistry;

   import java.util.Collections;
   import java.util.List;
   import java.util.Optional;

   public class OfficeCountMetricDecorator implements ContactUsService {

     private final Counter officeCounter;
     private final JpaContactUsService target;

     public OfficeCountMetricDecorator( final MeterRegistry registry, final OfficesRepository repository, final JpaContactUsService target ) {
       officeCounter = registry.counter( "app.office.count" );
       officeCounter.increment( repository.count() );
       this.target = target;
     }

     @Override
     public List<Office> list() {
       return target.list();
     }

     @Override
     public Optional<Office> findOneByName( final String name ) { /* ... */ }

     @Override
     public List<Office> findAllInCountry( final String country ) { /* ... */ }

     @Override
     public Optional<Office> update( final Office office ) { /* ... */ }

     @Override
     public Optional<Office> delete( final String name ) { /* ... */ }
   }
   ```

   Run the tests.

   ```bash
   $ ./gradlew clean test

   ...

   BUILD SUCCESSFUL in 9s
   6 actionable tasks: 6 executed
   ```

   This time they should pass.

1. Refactor the tests

   Both test methods have lots of similarities and these can be refactored as shown next.

   Update file: `src/test/java/demo/boot/OfficeCountMetricDecoratorTest.java`

   ```java
   package demo.boot;

   import io.micrometer.core.instrument.Counter;
   import io.micrometer.core.instrument.MeterRegistry;
   import org.junit.jupiter.api.AfterEach;
   import org.junit.jupiter.api.BeforeEach;
   import org.junit.jupiter.api.DisplayName;
   import org.junit.jupiter.api.Test;

   import java.util.List;

   import static org.assertj.core.api.Assertions.assertThat;
   import static org.mockito.ArgumentMatchers.eq;
   import static org.mockito.Mockito.mock;
   import static org.mockito.Mockito.reset;
   import static org.mockito.Mockito.times;
   import static org.mockito.Mockito.verify;
   import static org.mockito.Mockito.verifyNoMoreInteractions;
   import static org.mockito.Mockito.when;

   @DisplayName( "Office count metric" )
   public class OfficeCountMetricDecoratorTest {

     private final long initialNumberOfOffices = 10L;
     private final String counterName = "app.office.count";

     private final MeterRegistry registry = mock( MeterRegistry.class );
     private final Counter counter = mock( Counter.class );
     private final OfficesRepository repository = mock( OfficesRepository.class );
     private final JpaContactUsService target = mock( JpaContactUsService.class );
     private final Office office = mock( Office.class );

     @BeforeEach
     public void setUp() {
       reset( registry, repository, counter, target, office );

       when( registry.counter( eq( counterName ) ) ).thenReturn( counter );
       when( repository.count() ).thenReturn( initialNumberOfOffices );
     }

     @AfterEach
     public void tearDown() {
       verify( registry, times( 1 ) ).counter( counterName );
       verify( repository, times( 1 ) ).count();
       verify( counter, times( 1 ) ).increment( eq( (double) initialNumberOfOffices ) );
       verifyNoMoreInteractions( registry, repository, counter, target, office );
     }

     private ContactUsService withService() {
       return new OfficeCountMetricDecorator( registry, repository, target );
     }

     @Test
     @DisplayName( "should set the counter to the number of offices in the repository" )
     public void shouldInitCounter() {
       withService();
     }

     @Test
     @DisplayName( "should call the target list() method without changing the counter's value" )
     public void shouldPassListRequestsThrough() {
       final List<Office> expected = List.of( office );
       when( target.list() ).thenReturn( expected );

       final List<Office> actual = withService().list();
       assertThat( actual ).isSameAs( expected );

       verify( target, times( 1 ) ).list();
     }
   }
   ```

   Each test method now is very concise, just focuses on the test being carried out.

   {% include custom/note.html details="Our <code>OfficeCountMetricDecorator</code> is initialising the counter at the constructor.  This means that we need to call the <code>OfficesRepository</code> and the <code>Counter</code> with every test we run.  Alternatively we can use a post construct annotation, such as <a href='https://docs.oracle.com/javaee/7/api/javax/annotation/PostConstruct.html'><code>@PostConstruct</code></a>, or implement the <a href='https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/beans/factory/InitializingBean.html'><code>InitializingBean</code></a> interface and have these only called once.<p>The downside with these approaches is that we will need to hang on the <code>OfficesRepository</code> instance (and possibly the <code>MeterRegistry</code> instance too) from within the <code>OfficeCountMetricDecorator</code> class." %}

1. Test the `findOneByName()`, `findAllInCountry()` and `update()` methods.  None of these should affect the counter.

   Update file: `src/test/java/demo/boot/OfficeCountMetricDecoratorTest.java`

   ```java
   package demo.boot;

   import io.micrometer.core.instrument.Counter;
   import io.micrometer.core.instrument.MeterRegistry;
   import org.junit.jupiter.api.AfterEach;
   import org.junit.jupiter.api.BeforeEach;
   import org.junit.jupiter.api.DisplayName;
   import org.junit.jupiter.api.Test;

   import java.util.List;
   import java.util.Optional;

   import static org.assertj.core.api.Assertions.assertThat;
   import static org.mockito.ArgumentMatchers.eq;
   import static org.mockito.ArgumentMatchers.same;
   import static org.mockito.Mockito.mock;
   import static org.mockito.Mockito.reset;
   import static org.mockito.Mockito.times;
   import static org.mockito.Mockito.verify;
   import static org.mockito.Mockito.verifyNoMoreInteractions;
   import static org.mockito.Mockito.when;

   @DisplayName( "Office count metric" )
   public class OfficeCountMetricDecoratorTest {

     private final long initialNumberOfOffices = 10L;
     private final String counterName = "app.office.count";

     private final MeterRegistry registry = mock( MeterRegistry.class );
     private final Counter counter = mock( Counter.class );
     private final OfficesRepository repository = mock( OfficesRepository.class );
     private final JpaContactUsService target = mock( JpaContactUsService.class );
     private final Office office = mock( Office.class );

     @BeforeEach
     public void setUp() { /* ... */ }

     @AfterEach
     public void tearDown() { /* ... */ }

     private ContactUsService withService() { /* ... */ }

     @Test
     @DisplayName( "should set the counter to the number of offices in the repository" )
     public void shouldInitCounter() { /* ... */ }

     @Test
     @DisplayName( "should call the target list() method without changing the counter's value" )
     public void shouldPassListRequestsThrough() { /* ... */ }

     @Test
     @DisplayName( "should call the target findOneByName() method without changing the counter's value" )
     public void shouldPassFindOneByNameRequestsThrough() {
       final String name = "office name";
       final Optional<Office> expected = Optional.of( office );
       when( target.findOneByName( same( name ) ) ).thenReturn( expected );

       final Optional<Office> actual = withService().findOneByName( name );
       assertThat( actual ).isSameAs( expected );

       verify( target, times( 1 ) ).findOneByName( name );
     }

     @Test
     @DisplayName( "should call the target findAllInCountry() method without changing the counter's value" )
     public void shouldPassFindAllInCountryRequestsThrough() {
       final String country = "Germany";
       final List<Office> expected = List.of( office );
       when( target.findAllInCountry( same( country ) ) ).thenReturn( expected );

       final List<Office> actual = withService().findAllInCountry( country );
       assertThat( actual ).isSameAs( expected );

       verify( target, times( 1 ) ).findAllInCountry( country );
     }

     @Test
     @DisplayName( "should call the target update() method without changing the counter's value" )
     public void shouldPassUpdateRequestsThrough() {
       final Optional<Office> expected = Optional.of( office );
       when( target.update( same( office ) ) ).thenReturn( expected );

       final Optional<Office> actual = withService().update( office );
       assertThat( actual ).isSameAs( expected );

       verify( target, times( 1 ) ).update( office );
     }
   }
   ```

   Run the tests. These should fail.  Fix the failing tests.

   Update file: `src/main/java/demo/boot/OfficeCountMetricDecorator.java`

   ```java
   package demo.boot;

   import io.micrometer.core.instrument.Counter;
   import io.micrometer.core.instrument.MeterRegistry;

   import java.util.List;
   import java.util.Optional;

   public class OfficeCountMetricDecorator implements ContactUsService {

     private final Counter officeCounter;
     private final JpaContactUsService target;

     public OfficeCountMetricDecorator( final MeterRegistry registry, final OfficesRepository repository, final JpaContactUsService target ) { /* ... */ }

     @Override
     public List<Office> list() { /* ... */ }

     @Override
     public Optional<Office> findOneByName( final String name ) {
       return target.findOneByName( name );
     }

     @Override
     public List<Office> findAllInCountry( final String country ) {
       return target.findAllInCountry( country );
     }

     @Override
     public Optional<Office> update( final Office office ) {
       return target.update( office );
     }

     @Override
     public Optional<Office> delete( final String name ) { /* ... */ }
   }
   ```

   The tests should now all pass.

1. Delete an office that does not exists

   When an office that does not exists is deleted, the counter should not be modified, as the office count is not affected.

   Update file: `src/test/java/demo/boot/OfficeCountMetricDecoratorTest.java`

   ```java
   package demo.boot;

   import io.micrometer.core.instrument.Counter;
   import io.micrometer.core.instrument.MeterRegistry;
   import org.junit.jupiter.api.AfterEach;
   import org.junit.jupiter.api.BeforeEach;
   import org.junit.jupiter.api.DisplayName;
   import org.junit.jupiter.api.Test;

   import java.util.List;
   import java.util.Optional;

   import static org.assertj.core.api.Assertions.assertThat;
   import static org.mockito.ArgumentMatchers.eq;
   import static org.mockito.ArgumentMatchers.same;
   import static org.mockito.Mockito.mock;
   import static org.mockito.Mockito.reset;
   import static org.mockito.Mockito.times;
   import static org.mockito.Mockito.verify;
   import static org.mockito.Mockito.verifyNoMoreInteractions;
   import static org.mockito.Mockito.when;

   @DisplayName( "Office count metric" )
   public class OfficeCountMetricDecoratorTest {

     private final long initialNumberOfOffices = 10L;
     private final String counterName = "app.office.count";

     private final MeterRegistry registry = mock( MeterRegistry.class );
     private final Counter counter = mock( Counter.class );
     private final OfficesRepository repository = mock( OfficesRepository.class );
     private final JpaContactUsService target = mock( JpaContactUsService.class );
     private final Office office = mock( Office.class );

     @BeforeEach
     public void setUp() { /* ... */ }

     @AfterEach
     public void tearDown() { /* ... */ }

     private ContactUsService withService() { /* ... */ }

     @Test
     @DisplayName( "should set the counter to the number of offices in the repository" )
     public void shouldInitCounter() { /* ... */ }

     @Test
     @DisplayName( "should call the target list() method without changing the counter's value" )
     public void shouldPassListRequestsThrough() { /* ... */ }

     @Test
     @DisplayName( "should call the target findOneByName() method without changing the counter's value" )
     public void shouldPassFindOneByNameRequestsThrough() { /* ... */ }

     @Test
     @DisplayName( "should call the target findAllInCountry() method without changing the counter's value" )
     public void shouldPassFindAllInCountryRequestsThrough() { /* ... */ }

     @Test
     @DisplayName( "should call the target update() method without changing the counter's value" )
     public void shouldPassUpdateRequestsThrough() { /* ... */ }

     @Test
     @DisplayName( "should call the target delete() method for an office that does not exists and without changing the counter's value" )
     public void shouldPassDeleteRequestsThroughAndDoesNotAdjustTheCounter() {
       final String name = "Office name";
       final Optional<Office> expected = Optional.empty();
       when( target.delete( same( name ) ) ).thenReturn( expected );

       final Optional<Office> actual = withService().delete( name );
       assertThat( actual ).isSameAs( expected );

       verify( target, times( 1 ) ).delete( name );
     }
   }
   ```

   Run the tests.  These should fail.  Fix the failing test.

   Update file: `src/main/java/demo/boot/OfficeCountMetricDecorator.java`

   ```java
   package demo.boot;

   import io.micrometer.core.instrument.Counter;
   import io.micrometer.core.instrument.MeterRegistry;

   import java.util.List;
   import java.util.Optional;

   public class OfficeCountMetricDecorator implements ContactUsService {

     private final Counter officeCounter;
     private final JpaContactUsService target;

     public OfficeCountMetricDecorator( final MeterRegistry registry, final OfficesRepository repository, final JpaContactUsService target ) { /* ... */ }

     @Override
     public List<Office> list() { /* ... */ }

     @Override
     public Optional<Office> findOneByName( final String name ) { /* ... */ }

     @Override
     public List<Office> findAllInCountry( final String country ) { /* ... */ }

     @Override
     public Optional<Office> update( final Office office ) { /* ... */ }

     @Override
     public Optional<Office> delete( final String name ) {
       return target.delete( name );
     }
   }
   ```

   Run the tests.  These should now all pass.

1. Delete an office that exists

   When an office is deleted, we need to decrement the office count to reflect the new deletion.

   {% include custom/note.html details="In a <a href='/spring-boot-camp/docs/data/custom-queries/delete/'>previous section</a>, we invested some effort in making the `delete()` method thread-safe.  This will pay off now as this decorator relies on the output of the `delete()` method.  We do not want our decorator to decrement the number of offices more that it should." %}

   Update file: `src/test/java/demo/boot/OfficeCountMetricDecoratorTest.java`

   ```java
   package demo.boot;

   import io.micrometer.core.instrument.Counter;
   import io.micrometer.core.instrument.MeterRegistry;
   import org.junit.jupiter.api.AfterEach;
   import org.junit.jupiter.api.BeforeEach;
   import org.junit.jupiter.api.DisplayName;
   import org.junit.jupiter.api.Test;

   import java.util.List;
   import java.util.Optional;

   import static org.assertj.core.api.Assertions.assertThat;
   import static org.mockito.ArgumentMatchers.eq;
   import static org.mockito.ArgumentMatchers.same;
   import static org.mockito.Mockito.mock;
   import static org.mockito.Mockito.reset;
   import static org.mockito.Mockito.times;
   import static org.mockito.Mockito.verify;
   import static org.mockito.Mockito.verifyNoMoreInteractions;
   import static org.mockito.Mockito.when;

   @DisplayName( "Office count metric" )
   public class OfficeCountMetricDecoratorTest {

     private final long initialNumberOfOffices = 10L;
     private final String counterName = "app.office.count";

     private final MeterRegistry registry = mock( MeterRegistry.class );
     private final Counter counter = mock( Counter.class );
     private final OfficesRepository repository = mock( OfficesRepository.class );
     private final JpaContactUsService target = mock( JpaContactUsService.class );
     private final Office office = mock( Office.class );

     @BeforeEach
     public void setUp() { /* ... */ }

     @AfterEach
     public void tearDown() { /* ... */ }

     private ContactUsService withService() { /* ... */ }

     @Test
     @DisplayName( "should set the counter to the number of offices in the repository" )
     public void shouldInitCounter() { /* ... */ }

     @Test
     @DisplayName( "should call the target list() method without changing the counter's value" )
     public void shouldPassListRequestsThrough() { /* ... */ }

     @Test
     @DisplayName( "should call the target findOneByName() method without changing the counter's value" )
     public void shouldPassFindOneByNameRequestsThrough() { /* ... */ }

     @Test
     @DisplayName( "should call the target findAllInCountry() method without changing the counter's value" )
     public void shouldPassFindAllInCountryRequestsThrough() { /* ... */ }

     @Test
     @DisplayName( "should call the target update() method without changing the counter's value" )
     public void shouldPassUpdateRequestsThrough() { /* ... */ }

     @Test
     @DisplayName( "should call the target delete() method for an office that does not exists and without changing the counter's value" )
     public void shouldPassDeleteRequestsThroughAndDoesNotAdjustTheCounter() { /* ... */ }

     @Test
     @DisplayName( "should call the target delete() method for an office that exists and decrement the counter's value" )
     public void shouldPassDeleteRequestsThroughAndAdjustTheCounter() {
       final String name = "Office name";
       final Optional<Office> expected = Optional.of( office );
       when( target.delete( same( name ) ) ).thenReturn( expected );

       final Optional<Office> result = withService().delete( name );
       /* The result will be wrapped in a new Optional, thus we cannot use the isSameAs() for the Optional */
       assertThat( result ).isEqualTo( expected );
       assertThat( result.get() ).isSameAs( office );

       verify( target, times( 1 ) ).delete( name );
       verify( counter, times( 1 ) ).increment( -1D );
     }
   }
   ```

   The test will fail as we are not decrementing the counter as expected.

   ```bash
   $ ./gradlew clean test

   ...

   Office count metric > should call the target delete() method for an office that exists and decrement the counter's value FAILED
       org.mockito.exceptions.verification.opentest4j.ArgumentsAreDifferent at OfficeCountMetricDecoratorTest.java:137

   ...

   BUILD FAILED in 15s
   6 actionable tasks: 6 executed
   ```

   Fix the failing test.

   ```java
   package demo.boot;

   import io.micrometer.core.instrument.Counter;
   import io.micrometer.core.instrument.MeterRegistry;

   import java.util.List;
   import java.util.Optional;

   public class OfficeCountMetricDecorator implements ContactUsService {

     private final Counter officeCounter;
     private final JpaContactUsService target;

     public OfficeCountMetricDecorator( final MeterRegistry registry, final OfficesRepository repository, final JpaContactUsService target ) { /* ... */ }

     @Override
     public List<Office> list() { /* ... */ }

     @Override
     public Optional<Office> findOneByName( final String name ) { /* ... */ }

     @Override
     public List<Office> findAllInCountry( final String country ) { /* ... */ }

     @Override
     public Optional<Office> update( final Office office ) { /* ... */ }

     @Override
     public Optional<Office> delete( final String name ) {
       return target
         .delete( name )
         .map( office -> {
           officeCounter.increment( -1 );
           return office;
         } );
     }
   }
   ```

   Run the tests.  These should now all pass.

   We can refactor the `delete()` method and extract the decrement part into a new method, as shown next.

   ```java
   package demo.boot;

   import io.micrometer.core.instrument.Counter;
   import io.micrometer.core.instrument.MeterRegistry;

   import java.util.List;
   import java.util.Optional;
   import java.util.function.Function;

   public class OfficeCountMetricDecorator implements ContactUsService {

     private final Counter officeCounter;
     private final JpaContactUsService target;

     public OfficeCountMetricDecorator( final MeterRegistry registry, final OfficesRepository repository, final JpaContactUsService target ) { /* ... */ }

     @Override
     public List<Office> list() { /* ... */ }

     @Override
     public Optional<Office> findOneByName( final String name ) { /* ... */ }

     @Override
     public List<Office> findAllInCountry( final String country ) { /* ... */ }

     @Override
     public Optional<Office> update( final Office office ) { /* ... */ }

     @Override
     public Optional<Office> delete( final String name ) {
       return target
         .delete( name )
         .map( decrementOfficeCount() );
     }

     private Function<Office, Office> decrementOfficeCount() {
       return office -> {
         officeCounter.increment( -1 );
         return office;
       };
     }
   }
   ```

1. Make the `OfficeCountMetricDecorator` the primary service

   Update file: `src/test-integration/java/demo/boot/ContactUsApplicationTests.java`

   ```java
   package demo.boot;

   import org.junit.jupiter.api.DisplayName;
   import org.junit.jupiter.api.Test;
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.boot.test.context.SpringBootTest;
   import org.springframework.boot.test.web.client.TestRestTemplate;
   import org.springframework.http.HttpStatus;

   import static org.assertj.core.api.Assertions.assertThat;
   import static org.springframework.boot.test.context.SpringBootTest.WebEnvironment;

   @DisplayName( "Contact Us application" )
   @SpringBootTest( webEnvironment = WebEnvironment.RANDOM_PORT )
   public class ContactUsApplicationTests {

     @Autowired
     private TestRestTemplate restTemplate;

     @Test
     @DisplayName( "should return 200 when the health endpoint is accessed" )
     public void shouldReturn200HealthEndpoint() { /* ... */ }

     @Test
     @DisplayName( "should return 200 when the metrics office count endpoint is accessed" )
     public void shouldReturn200MetricsOfficeCountEndpoint() {
       assertThat( restTemplate.getForEntity( "/metrics/app.office.count", String.class ) )
         .matches( r -> r.getStatusCode() == HttpStatus.OK );
     }

     @Test
     @DisplayName( "should return the offices" )
     public void shouldReturnTheOffices() { /* ... */ }
   }
   ```

   Run the integration tests.

   ```bash
   $ ./gradlew clean integrationTest

   ...

   Contact Us application > should return 200 when the metrics office count endpoint is accessed FAILED
       java.lang.AssertionError at ContactUsApplicationTests.java:31

   ...

   BUILD FAILED in 13s
   7 actionable tasks: 7 executed
   ```

   Annotate the `OfficeCountMetricDecorator` with both the [`@Primary`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/annotation/Primary.html) and [`@Service`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/stereotype/Service.html) annotations

   ```java
   package demo.boot;

   import io.micrometer.core.instrument.Counter;
   import io.micrometer.core.instrument.MeterRegistry;
   import org.springframework.context.annotation.Primary;
   import org.springframework.stereotype.Service;

   import java.util.List;
   import java.util.Optional;
   import java.util.function.Function;

   @Primary
   @Service
   public class OfficeCountMetricDecorator implements ContactUsService { /* ... */ }
   ```

   Run the integration tests.  These should now work.

## Prometheus

```groovy
dependencies {
  /* Prometheus */
  runtimeOnly 'io.micrometer:micrometer-registry-prometheus'
}
```

```yaml
management:
  endpoints:
    web:
      base-path:
      exposure:
        include: env, health, info, metrics, prometheus
  endpoint:
    health:
      show-details: always
```

```bash
$ curl "http://localhost:8080/prometheus"
```

`docker/prometheus/prometheus.yaml`
```yaml
global:
  scrape_interval: 15s
scrape_configs:
  - job_name: 'contact-us'
    scrape_interval: 5s
    metrics_path: '/prometheus'
    static_configs:
      - targets: ['host.docker.internal:8080'] # Points to the laptop
```

`docker-compose.yml`
```yaml
  prometheus:
    image: prom/prometheus:latest
    container_name: ${DATABASE_NAME}-prometheus
    ports:
      - 9090:9090
    env_file:
      - .env
    command:
      - --config.file=/etc/prometheus/prometheus.yaml
    volumes:
      - ./docker/prometheus/prometheus.yaml:/etc/prometheus/prometheus.yaml:ro
```


http://localhost:9090/targets

![Prometheus Targets]({{ '/assets/images/Prometheus-Targets.png' | absolute_url }})

HTTP Requests

![Prometheus Graph]({{ '/assets/images/Prometheus-Graph.png' | absolute_url }})

