---
layout: default
title: Basics of REST
parent: REST
nav_order: 1
permalink: docs/rest/basics/
---

# Basics of REST
{: .no_toc }

REST was defined by computer scientist [Roy Fielding](https://en.wikipedia.org/wiki/Roy_Fielding). He presented the REST principles in his [PhD dissertation](https://www.ics.uci.edu/~fielding/pubs/dissertation/fielding_dissertation.pdf) in 2000.

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

## An overview

It consists of two components:
* REST server which provides access to the resources;
* REST client which accesses and modify the REST resources.

Clients and servers exchange representations of resources by using a standardized interface and protocol. REST allows that resources have different representations, e.g. XML, JSON etc. The rest client can ask for specific representation via the HTTP protocol.

![Rest-Architecture.jpg]({{ 'assets/images/Rest-Architecture.jpg' | absolute_url }})


## REST architectural constraints and properties

REST is an approach to designing the APIs, and it introduces some design rules or constraints. Because REST is not an established standard, the implementation rules of the constraints aren't strictly defined.

The main list of constraints used to design REST API:
1. Client — Server (Separation Of Concern): the client or server can change or evolve without impacting each other;
1. Uniform Interface: the client and server a common “technical” interface (contract), mostly the definition of the contract uses HTTP methods and media-types;
1. Statelessness: this means the server does not remember if the user of the API already sent any request for the same resource in the past, it does not remember which resources the user of the API requested before, and so on;
1. Caching: when the server uses the cache, it has to inform the client in some way;
1. Layered System: the REST API architecture should consist of multiple layers and each of this layer within the architecture can change independent of the other layers;
1. Code On Demand (optional): the server can extend the client’s functionality by sending some code (for example JavaScript).


## Unique Identifier

Each resource has a unique identifier, for example a name or a number. The identifier will be part of the URL used to access the resource. This way each resource becomes a specific and unique URL.

URL stands for Uniform Resource Locator. This is a reference to a web resource that specifies its location on a computer network and a mechanism for retrieving its. URL could be considered as a specific type of Uniform Resource Identifier (URI) with __scheme__ _HTTP_ or _HTTPS_:
```
URI = scheme:[//authority]path[?query][#fragment]
```


## HTTP methods

REST uses HTTP protocol methods to define the operations which will be performed with resources:
* **GET**: It is used for reading the resource without side effects. This operation is idempotent i.e. it can be applied multiple times without changing the result.
* **PUT**: It is generally used for updating resource. It must also be idempotent.
* **DELETE**: It removes the resources. The operation is idempotent i.e. it can get repeated without leading to different results (the resource doesn't exist after the operation).
* **POST**: It is used for creating a new resource. The only operation that is not idempotent.


## Hypermedia extension HATEOAS

The response from server besides the resource could contain additional information about other resources available. A specific representation **HATEOAS** (Hypermedia as the Engine of Application State) can be used for this purpose.

The idea behind it is that the server not only sends the response data but also sends the possible actions the client can take on the resource or data.



## Richardson Maturity Model

In the book [RESTful Web APIs](https://www.goodreads.com/book/show/17346969-restful-web-apis) Leonard Richardson propose [REST API maturity model](https://martinfowler.com/articles/richardsonMaturityModel.html).
This a way to grade REST API according to the constraints of REST. The better the REST API adheres to these constraints, the higher its score is.


The model defines four maturity levels of REST API:
* Level Zero: This level does not make use of URLs, HTTP Methods, and HATEOAS capabilities.
* Level One: At this level a unique URL for each resource can be used to access the resource.
* Level Two: At this level a unique URL and HTTP Methods can used to manipulate the resource.
* Level Three: All the options - unique URL, HTTP Methods and HATEOAS can used to manipulate current and other resources.
