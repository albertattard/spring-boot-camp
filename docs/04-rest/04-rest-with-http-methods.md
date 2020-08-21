---
layout: default
title: REST with HTTP Methods
parent: REST
nav_order: 4
permalink: docs/rest/with-http-methods/
---

# Simple REST implementation
{: .no_toc }

Spring Boot allows us to use **HTTP Methods** for a resource manipulating. Such an implementation of a REST Controller could be considered as a level 2 of the Richardson Maturity Model.

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

## Manipulating the resources

Sometimes we need to perform some manipulations with resources, for example to modify or to create a new one. If the manipulations are some kind of CRUD, we can use **HTTP Methods** to define the operation needed. The **HTTP Method** can be could _safe_ if it does not change the state of the resource. Accordingly, the **HTTP Method** can be considered as _idempotent_ if making multiple the identical requests must produce the same result every time until some other process has changed the state of the resource.

In additional, to inform the client about the result of the manipulation, the **HTTP Response Code** could be used.
The most widely used codes to indicate a success are:
* ```200 (OK)``` - indicates that the request is processed successfully, and the response contains an information about the resource as a body,
* ```201 (Created)``` - indicates that a new resource is created, and the response contains a resource as a body,
* ```204 (No Content)``` - indicated that the request is processed successfully, but the response don*t contain any additional information as a body.

The common code to indicate a failure are:
* ```400 (Bad Request)``` - if the request has incorrect form,
* ```404 (Not Found)``` - if the resource isn't found on the server.

The most usable **HTTP method** for the action performed by API are listed below
* **HTTP GET**: GET requests will be used to retrieve resource representation/information only. The success response returns ```200 (OK)``` together with the body containing the resource. The request is _safe_ and _idempotent_ and the response is cache-able.
* **HTTP POST**: Post request will be used to create a new resource, mostly in some already existing collection of similar resources. If the resource has been successfully created, the response should contain ```201 (Created)``` together with the body containing the new resource. The request is neither _safe_ nor _idempotent_ and the response is not cache-able.
* **HTTP PUT**: PUT request will be used to update existing resource or (in some cases). The success response returns ```201 (Created)``` together with the body containing the resource if a new resource was created. If the existing resource was modified, the response will contain ```200 (OK)``` or ```204 (No Content)```, depending on the existence of a body with the information about the resource. The request is _idempotent_ but not _safe_ and the response is not cache-able.
* **HTTP DELETE**: DELETE request will be used to remove the resource from the server. The success response returns ```200 (OK)``` if it contains a body with information or ```204 (No Content)``` if the body isn't present. Sometimes the remove action can't be performed immediately and have to be queued. In this case the response will return ```202 (Accepted)```. The request is _idempotent_ but not _safe_ and the response is not cache-able.

## Rest Controller with HTTP methods

Spring provide some specific annotations to support the HTTP Methods:
* [@GetMapping](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/web/bind/annotation/GetMapping.html): this is a composed annotation that is equal to ```@RequestMapping(method = RequestMethod.GET)```;
* [@PostMapping](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/web/bind/annotation/PostMapping.html): this is a composed annotation that is equal to ```@RequestMapping(method = RequestMethod.POST)```;
* [@PutMapping](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/web/bind/annotation/PutMapping.html): this is a composed annotation that is equal to ```@RequestMapping(method = RequestMethod.PUT)```;
* [@DeleteMapping](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/web/bind/annotation/DeleteMapping.html): this is a composed annotation that is equal to ```@RequestMapping(method = RequestMethod.DELETE)```;


### Getting a resource with HTTP GET

The example implementation is here:

```Java
@RestController
public class OfficeController {

  private final ContactUsService service;

  @GetMapping( "/offices/{name}" )
  public Office offices(final @PathVariable("name") String name) {
    return service
      .findOneByName(name)
      .orElseThrow(() -> new OfficeNotFoundException("Office not found with name " + name));
  }
}
```
&nbsp;
The test:
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
      .andExpect(status().isOk())
      .andReturn();
    String actualResponseBody = mvcResult.getResponse().getContentAsString();

    assertThat(objectMapper.readValue( actualResponseBody, Office.class )).isEqualTo(office);
    verify(service, times(1)).findOneByName(name);
    verifyNoMoreInteractions(service);
  }
}
```

With ```service.findOneByName(eq(name))``` the test makes sure the right value of variable _name_ was used.



### Crating a resource with HTTP POST

The example implementation is here:

```Java
@RestController
public class OfficeController {

  private final ContactUsService service;

  @PostMapping("/offices")
  @ResponseStatus(value = HttpStatus.CREATED)
  public Office createOffice(final @RequestBody Office office) {
    return service
      .create(office);
  }
}
```
&nbsp;
The test:
```Java
@WebMvcTest( OfficesController.class )
public class OfficesControllerTest {

  @Test
  @DisplayName( "should create a new office" )
  public void shouldCreateNewOffice() {
    final String name = "ThoughtWorks Cologne";
    final Office office =
      new Office( name,
        "Lichtstr. 43i, 50825 Cologne, Germany",
        "+49 221 64 30 70 63",
        "contact-de@thoughtworks.com" );
    when(service.create(eq(office))).thenReturn(office);

    MvcResult mvcResult = mockMvc.perform(post("/offices")
      .contentType("application/json")
      .characterEncoding("utf-8")
      .content(objectMapper.writeValueAsString(office)))
      .andExpect(status().isCreated())
      .andReturn();
    String actualResponseBody = mvcResult.getResponse().getContentAsString();

    assertThat(objectMapper.readValue( actualResponseBody, Office.class )).isEqualTo(office);
    verify(service, times(1)).create(office);
    verifyNoMoreInteractions(service);
  }
}
```
With ```service.create(eq(office))``` the test makes sure the expected variable _office_ was used to call a business logic. This way we are able to test the serialisation of an _RequestBody_ (as an Input).


### Modifying a resource with HTTP PUT

The example implementation is here (without creating a new resource if not found):

```Java
@RestController
public class OfficeController {

  private final ContactUsService service;

  @PutMapping("/offices/{name}")
  public Office updateOffice(final @RequestBody Office office, final @PathVariable("name") String name) {
    return service
      .update(office)
      .orElseThrow(() -> new OfficeNotFoundException("Office not found with name " + office.getName()));
  }
}
```
&nbsp;
The test:
```Java
@WebMvcTest( OfficesController.class )
public class OfficesControllerTest {

  @Test
  @DisplayName( "should update the office" )
  public void shouldUpdateOffice() {
    final String name = "ThoughtWorks Cologne";
    final Office office =
      new Office( name,
        "Lichtstr. 43i, 50825 Cologne, Germany",
        "+49 221 64 30 70 63",
        "contact-de@thoughtworks.com" );
    when(service.update(eq(office))).thenReturn(Optional.of(office));

    MvcResult mvcResult = mockMvc.perform(put("/offices/{name}", name)
      .contentType("application/json")
      .characterEncoding("utf-8")
      .content(objectMapper.writeValueAsString(office)))
      .andExpect(status().isOk())
      .andReturn();
    String actualResponseBody = mvcResult.getResponse().getContentAsString();

    assertThat(objectMapper.readValue( actualResponseBody, Office.class )).isEqualTo(office);
    verify(service, times(1)).update(office);
    verifyNoMoreInteractions(service);
  }
}
```
With ```service.update(eq(office))``` the test makes sure the expected variable _office_ was used to call a business logic. This way we are able to test the serialisation of an _RequestBody_ (as an Input).


### Removing a resource with HTTP DELETE

The example implementation is here:

```Java
@RestController
public class OfficeController {

  private final ContactUsService service;

  @DeleteMapping("/{name}")
  @ResponseStatus(value = HttpStatus.NO_CONTENT)
  public void deleteOffice(final @PathVariable("name") String name) {
    service.delete(name);
  }
}
```
&nbsp;
The test:
```Java
@WebMvcTest( OfficesController.class )
public class OfficesControllerTest {

  @Test
  @DisplayName( "should delete the office" )
  public void shouldDeleteOffice() {
    final String name = "ThoughtWorks Cologne";
    final Office office =
      new Office( name,
        "Lichtstr. 43i, 50825 Cologne, Germany",
        "+49 221 64 30 70 63",
        "contact-de@thoughtworks.com" );
    when(service.delete(eq(name))).thenReturn(Optional.of(office));

    mockMvc.perform(delete("/offices/{name}", name))
      .andExpect(status().isNoContent())
      .andReturn();

    verify(service, times(1)).delete(name);
    verifyNoMoreInteractions(service);
  }}
```
With ```service.delete(eq(name))``` the test makes sure the right value of variable _name_ was used.
