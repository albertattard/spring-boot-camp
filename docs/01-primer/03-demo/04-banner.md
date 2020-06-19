---
layout: default
title: Banner
parent: Demo
grand_parent: Primer
nav_order: 4
permalink: docs/primer/demo/banner/
---

# Banner
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Run the application

1. Run the application using the [Spring boot Gradle task `bootRun`](https://docs.spring.io/spring-boot/docs/current/gradle-plugin/api/org/springframework/boot/gradle/tasks/run/BootRun.html)

   ```bash
   $ ./gradlew bootRun
   ```

   This should start the application

   ```bash
   > Task :bootRun

     .   ____          _            __ _ _
    /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
   ( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
    \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
     '  |____| .__|_| |_|_| |_\__, | / / / /
    =========|_|==============|___/=/_/_/_/
    :: Spring Boot ::        (v2.3.0.RELEASE)

   2077-04-27 12:34:55.768  INFO 5554 --- [           main] demo.boot.GameApplication               : Starting GameApplication on Alberts-MBP.fritz.box with PID 5554 (build/classes/java/main started by albertattard in .)
   2077-04-27 12:34:55.771  INFO 5554 --- [           main] demo.boot.GameApplication               : No active profile set, falling back to default profiles: default
   2077-04-27 12:34:56.545  INFO 5554 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 8080 (http)
   2077-04-27 12:34:56.556  INFO 5554 --- [           main] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
   2077-04-27 12:34:56.556  INFO 5554 --- [           main] org.apache.catalina.core.StandardEngine  : Starting Servlet engine: [Apache Tomcat/9.0.35]
   2077-04-27 12:34:56.629  INFO 5554 --- [           main] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
   2077-04-27 12:34:56.629  INFO 5554 --- [           main] o.s.web.context.ContextLoader            : Root WebApplicationContext: initialization completed in 819 ms
   2077-04-27 12:34:56.752  INFO 5554 --- [           main] o.s.s.concurrent.ThreadPoolTaskExecutor  : Initializing ExecutorService 'applicationTaskExecutor'
   2077-04-27 12:34:56.890  INFO 5554 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
   2077-04-27 12:34:56.898  INFO 5554 --- [           main] demo.boot.GameApplication               : Started GameApplication in 1.508 seconds (JVM running for 1.838)
   <=========----> 75% EXECUTING [1m 23s]
   > :bootRun
   ```

1. Access the application: [http://localhost:8080/](http://localhost:8080/)

   ![Whitelabel Error Page]({{ '/assets/images/Whitelabel-Error-Page.png' | absolute_url }})

   The application does not expose any endpoints and we have no custom error handling.

1. Stop the application

   Use `[control] + [c]` to stop the application

## Customise the application banner (the `banner.txt` file)

1. Create a banner ([Ascii Art](http://patorjk.com/software/taag/#p=display&f=Big&t=Contact%20Us))

   Create file: `src/main/resources/banner.txt`

   ```bash
     _____            _             _     _    _
    / ____|          | |           | |   | |  | |
   | |     ___  _ __ | |_ __ _  ___| |_  | |  | |___
   | |    / _ \| '_ \| __/ _` |/ __| __| | |  | / __|
   | |___| (_) | | | | || (_| | (__| |_  | |__| \__ \
    \_____\___/|_| |_|\__\__,_|\___|\__|  \____/|___/
   ```

   Any text will do here.

1. Running the application will now show the new banner

   ```bash
   $ ./gradlew bootRun

   > Task :bootRun
      _____            _             _     _    _
     / ____|          | |           | |   | |  | |
    | |     ___  _ __ | |_ __ _  ___| |_  | |  | |___
    | |    / _ \| '_ \| __/ _` |/ __| __| | |  | / __|
    | |___| (_) | | | | || (_| | (__| |_  | |__| \__ \
     \_____\___/|_| |_|\__\__,_|\___|\__|  \____/|___/
    ...
   ```

   Use `[control] + [c]` to stop the application
