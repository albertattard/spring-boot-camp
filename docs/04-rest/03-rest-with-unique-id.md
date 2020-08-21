---
layout: default
title: REST implementation with unique ID
parent: REST
nav_order: 3
permalink: docs/rest/with-id/
---

# REST implementation with unique ID
{: .no_toc }

Spring Boot provides us with options to customise the URL of the request. This way we can assign a unique URL to each resource. Such an implementation could be considered as a level 1 of the Richardson Maturity Model.

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

## More complicated Use Case - several resources

The simple REST controller from the previous chapter provides us access to the resource, for example, to the list of Offices. Sometimes we don't need the whole list, but more specific query instead.

Suppose, we need the one Office only. How can we inform the controller that we need the only one Office and which one. From technical perspective, there are __Request Parameter__ or __Path Variable__ for these purposes. We can use one of the or both in combination.

For example, we need one Office with the name *"Cologne"*. We can provide tho URL to achieve the result:
  * Using a URL with __Request Parameter__: ``` http://localhost/offices?name=Cologne ```
  * Using a URL containing __Path Variable__: ``` http://localhost/offices/Cologne ```

Another example, we need to list only offices from some country, for example *"Germany"*. Again, we can provide two approaches:
* Using a URL with __Request Parameter__: ``` http://localhost/offices?country=Germany ```
* Using a URL containing __Path Variable__: ``` http://localhost/offices/countries/Germany ```

In the first case the same base URL ```http://localhost/offices``` is always used and the customization can be achieved with a __Request Parameter__. From architectural point of view this means that we have to implement one endpoint only, but we need to have some logics to analise the __Request Parameter__.

In the second case the customization can be achieved with __Path Variable__, and the URLs for each kind of requests are somewhat different. From architectural point of view this means that we have to implement dedicated endpoints for each URL, but we don't need any logics to perform further analysis of the URLs.

The REST uses the second approach. The __Request Parameters__ are used mostly to provide additional parameters for the requests.


## Rest Controller with Path Variable

Spring provide some specific annotations to support the customisation of the URL
* [@PathVariable](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/web/bind/annotation/PathVariable.html): this annotation indicates that a method parameter should be bound to a URI, it supposes for annotated handler methods from ``` @RequestMapping ```;
* [@RequestParam](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/web/bind/annotation/RequestParam.html): this annotation indicates that a method parameter should be bound to a web request parameter;

Here is a sample of REST controller with __Path Variable__ (Could be considered as Level 1 according to Richardson Maturity Model)

```Java
@RestController
public class OfficeController {

  private final ContactUsService service;

  @RequestMapping( "/offices/{name}" )
  public Office offices(final @PathVariable("name") String name) {
    return service
      .findOneByName(name)
      .orElseThrow(() -> new OfficeNotFoundException("Office not found with name " + name));
  }
}
```

## How to test

In comparison with the simple Rest Controller from the previous chapter, the controller has the same responsibilities. So, the same approach for testing could be used here. The only small difference is related with handling the __Path Variable__

```Java
@WebMvcTest( OfficesController.class )
public class OfficesControllerTest {

  @Test
  @DisplayName( "should return one office" )
  public void shouldReturnOneOffice() {
    final String name = "ThoughtWorks Cologne";
    final Office office =
      new Office( name,
        "Lichtstr. 43i, 50825 Cologne, Germany",
        "+49 221 64 30 70 63",
        "contact-de@thoughtworks.com" );
    when(service.findOneByName(eq(name))).thenReturn(Optional.of(office));

    MvcResult mvcResult = mockMvc.perform(get("/offices/{name}", name))
      .andDo(print())
      .andExpect(status().isOk())
      .andReturn();
    String actualResponseBody = mvcResult.getResponse().getContentAsString();

    assertThat(objectMapper.readValue( actualResponseBody, Office.class )).isEqualTo(office);
    verify(service, times(1)).findOneByName(name);
    verifyNoMoreInteractions(service);
  }
}
```
