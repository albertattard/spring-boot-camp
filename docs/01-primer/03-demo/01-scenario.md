---
layout: default
title: Scenario
parent: Demo
grand_parent: Primer
nav_order: 1
permalink: docs/primer/demo/scenario/
---

# Scenario
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Scenario

Our company, [ThoughtWorks](https://www.thoughtworks.com/), is thinking in building an API to provide information about its offices around the globe.  This API will not have any frontend UI but will simply provide a set of [REST endpoints](https://de.wikipedia.org/wiki/Representational_State_Transfer).

1. Expose the health endpoint

   This endpoint will be used by [Kubernetes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/) to verify that our application is up and running.  It takes the form of `GET` request to the `/health`, as shown in the following example.

   ```bash
   $ curl -v "http://localhost:8080/health"
   ```

   Our application needs to return [an HTTP `200` response](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/200), as shown next.

   ```bash
   ...
   < HTTP/1.1 200
   ...
   ```

   The content returned by the response does not matter much.

1. Return the list of offices

   The second endpoint that we need to build is a `GET` request to the `/offices`, as shown in the following example.

   ```bash
   $ curl "http://localhost:8080/offices"
   ```

   This should return a [JSON](https://www.json.org/json-en.html) array of objects as shown next.

   ``` json
   [
     {
       "office": "string",
       "address": "string",
       "phone": "string",
       "email": "string"
     }
   ]
   ```

   Following is an example of the [Cologne office contact information](https://www.thoughtworks.com/locations/cologne).

   ```json
   [
     {
       "office": "ThoughtWorks Cologne",
       "address": "Lichtstr. 43i, 50825 Cologne, Germany",
       "phone": "+49 221 64 30 70 63",
       "email": "contact-de@thoughtworks.com"
     }
   ]
   ```

   The application needs to be deployed as a docker image.

## Tasks

We need to carry out the following tasks

- [ ] Health endpoint
- [ ] OpenAPI
- [ ] Return one office contact details
- [ ] Dockerize application
- [ ] Return all offices contact details

The application is dockerized as soon as we have something working (returning one office).  This approach allows us to deploy our application and make sure that it works in production before we invest any further by adding more functionality.  You will be surprised by the challenges you may encounter.
