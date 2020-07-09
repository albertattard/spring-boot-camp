---
layout: default
title: OpenAPI (swagger)
parent: Demo
grand_parent: Primer
nav_order: 7
permalink: docs/primer/demo/openapi/
---

# OpenAPI (swagger)
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## OpenApi (swagger)

1. Add that [OpenApi dependency](https://github.com/springdoc/springdoc-openapi) dependency

   Update file: `build.gradle`

   ```groovy
   dependencies {
    /* OpenApi/Swagger */
    implementation 'org.springdoc:springdoc-openapi-ui:1.3.9'
   }
   ```

1. Start the application

   {% include custom/note.html details="Note that you need to restart it if this was already running" %}

   ```bash
   $ ./gradlew bootRun
   ```

1. Access the OpenApi from browser: [http://localhost:8080/swagger-ui/index.html?configUrl=/v3/api-docs/swagger-config](http://localhost:8080/swagger-ui/index.html?configUrl=/v3/api-docs/swagger-config)

   ![OpenApi Offices Endpoint]({{ '/assets/images/OpenApi-Offices-Endpoint.png' | absolute_url }})

   You can try the API too.

   ![OpenApi Demo]({{ '/assets/gifs/OpenApi-Demo.gif' | absolute_url }})

## Objectives status

The application endpoints are exposed through OpenAPI

- [X] Health endpoint
- [X] OpenAPI
- [X] Return one office contact details
- [ ] Dockerize application
- [ ] Return all offices contact details
