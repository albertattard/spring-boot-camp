---
layout: default
title: Custom Queries
parent: Spring Data
nav_order: 6
has_children: true
permalink: docs/data/custom-queries/
---

# Custom Query
{: .no_toc }

We are able to return all offices using the provided [`findAll()`](https://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/repository/CrudRepository.html#findAll--) method.  In this section we will introduce custom queries and take advantage of [Spring Repositories](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#jpa.repositories).

For the time being we will not expose any of the features added here to the front-end as this is covered at a later stage.  Our changes should stop at the service layer.
