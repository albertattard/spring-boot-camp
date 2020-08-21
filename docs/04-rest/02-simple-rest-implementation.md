---
layout: default
title: Simple REST implementation
parent: REST
nav_order: 2
permalink: docs/rest/simple/
---

# Simple REST implementation
{: .no_toc }

Spring Boot provides a support to building REST API for server applications at all maturity levels. Here the simple implementation is presented. It could be considered as a level 0.

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

## Simple REST Controller

The main component provided by Spring Boot is [RestController](https://docs.spring.io/spring/docs/current/spring-framework-reference/web.html#mvc-controller). It will be used to express request mappings, request input, exception handling, and more.

The __RestController__ and other related components can be found in the Spring Boot Starter Web dependency:
```
compile('org.springframework.boot:spring-boot-starter-we    b')
```

Here is a sample of simple REST controller (Level 0 according to Richardson Maturity Model)

```Java
@RestController
public class OfficeController {

  private final ContactUsService service;

  @RequestMapping( "/offices" )
  public List<Office> offices() {
    return service.list();
  }

}
```

Some annotations (directly or indirectly) are used here:
* [@RestController](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/web/bind/annotation/RestController.html): this is a composed annotation that contains annotations ```@Controller``` and ```@ResponseBody```;
* [@Controller](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/stereotype/Controller.html): allows Spring for auto-detection the class as a component and indicating its role as a web component;
* [@ResponseBody](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/web/bind/annotation/ResponseBody.html): shows that methods write directly to the response body without resolution and rendering with an HTML template;
* [@RequestMapping](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/web/bind/annotation/RequestMapping.html): helps to map requests to controllers methods, it has attributes for URL, HTTP method, request parameters, headers, and media types.


## How to test

Before testing the responsibilities of the REST controller have to be determined:
1. Listen to HTTP Requests: The controller should respond to certain URLs, HTTP methods and content types;
1. Call the Business Logic: The controller must call the expected business logic;
1. Serialize the Output: The controller takes the output of the business logic and serializes it into an HTTP response;

The only responsibility the Unit Test is able to check is the calling of the business logics. For other ones the integration test is needed. The main mocking tool that can be used for integration tests is [MockMvc](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/testing.html#spring-mvc-test-framework).


### Test HTTP Request Matching

We have to check the URL which the Controller is listening for, and the accepted HTTP Method (GET by default):

```Java
@WebMvcTest( OfficesController.class )
public class OfficesControllerTest {

  @Test
  @DisplayName( "should match URL and HTTP method for the request" )
  public void shouldMatchUrlAndMethod() {
    mockMvc.perform(get("/offices"))
      .andExpect(status().isOk());
  }
}
```
We expect the response with HTTP Status Code ```200 OK```.


### Test calling the business logics

To test the call of a business logic, the mock on business service can be used. We can check which method of the service class is called and how many times.

```Java
@WebMvcTest( OfficesController.class )
public class OfficesControllerTest {

  @DisplayName( "should verify the call for business logics" )
  public void shouldCallBusinessLogics() {
    mockMvc.perform(get("/offices"))
      .andExpect(status().isOk());

    verify(service, times(1)).list();
    verifyNoMoreInteractions(service);
  }
}
```
Also, we can check that no more interactions with the business service happened.


### Test the serialization of the Output

The Controller have to serialize the output from object to JSON. We have to test if the serialization was correct. For this purpose we can self serialize the output and compare with the result from the Controller.

```Java
@WebMvcTest( OfficesController.class )
public class OfficesControllerTest {

  @Test
  @DisplayName( "should verify the serialization of output" )
  public void shouldSerializeOutput() {
    final Office cologne =
      new Office( "ThoughtWorks Cologne",
        "Lichtstr. 43i, 50825 Cologne, Germany",
        "+49 221 64 30 70 63",
        "contact-de@thoughtworks.com" );
    when( service.list() ).thenReturn( List.of( cologne ) );

    MvcResult mvcResult = mockMvc.perform(get("/offices"))
      .andExpect(status().isOk())
      .andReturn();
    String actualResponseBody = mvcResult.getResponse().getContentAsString();

    assertThat(objectMapper.readValue( actualResponseBody, Office.class )).isEqualTo(office);
    verify(service, times(1)).list();
    verifyNoMoreInteractions(service);
  }
}
```

Here the _Object Mapper_ is a Spring component which is used for some object serialization or deserialization.


### Full Integration test

All the three tests are quite similar. Sometimes may be simpler to use the one test only. This test can check all the responsibilities.

```Java
@WebMvcTest( OfficesController.class )
public class OfficesControllerTest {

  @Test
  @DisplayName( "should return the list of offices returned by the service" )
  public void shouldReturnTheOffices() {
    final Office cologne =
      new Office( "ThoughtWorks Cologne",
        "Lichtstr. 43i, 50825 Cologne, Germany",
        "+49 221 64 30 70 63",
        "contact-de@thoughtworks.com" );
    when( service.list() ).thenReturn( List.of( cologne ) );

    mockMvc.perform( get( "/offices" ) )
      .andExpect( status().isOk() )
      .andExpect( jsonPath( "$" ).isArray() )
      .andExpect( jsonPath( "$", hasSize( 1 ) ) )
      .andExpect( jsonPath( "$.[0].name", is( cologne.getName() ) ) )
      .andExpect( jsonPath( "$.[0].address", is( cologne.getAddress() ) ) )
      .andExpect( jsonPath( "$.[0].phone", is( cologne.getPhone() ) ) )
      .andExpect( jsonPath( "$.[0].email", is( cologne.getEmail() ) ) );

    verify( service, times( 1 ) ).list();
    verifyNoMoreInteractions(service);
  }
}
```

Please pay the attention, in this test we don't use _Object Mapper_. Instead, we check the values of JSON directly with MockMvc. This is also one option to use.
